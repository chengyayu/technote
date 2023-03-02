# Goroutine 调度模型

## 模型结构

![go_gpm](../static/go_gpm.png)

- G（[Goroutine](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/runtime2.go;l=407)）
- P（[Processor](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/runtime2.go;l=609)）
- M（[Machine Thread](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/runtime2.go;l=526)）

## 调度器的生命周期

![调度器生命周期](../static/go_gpm_scheduler_life.png)

## Go 世界的开始

runtime.rt0_go() 中步骤如下

### 1. 

**初始化 g0 的栈区间**，检测 CPU 厂商及型号，按需调用 _cgo_init()，设置和检测 TLS，**将 m0 和 g0 互相关联**，并将 g0 设置到 TLS 中。

### 2. runtime.args() ...
### 3. runtime.osinit() ...

### 4. [runtime.schedinit()](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=669-775;bpv=0;bpt=1)

**P 已经初始化完毕**，还没有创建任何 G，所有 P 的 runq 都是空的。procresize() 函数返回后**当前 M 会和第一个 P 关联**，也就是 allp[0]。

### 5. [runtime.newproc()](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=4238-4254;bpv=1;bpt=1)

**创建主 goroutine**，指定入口函数 runtime.main()，**并把它放入 P 的本地队列中**。

此时 GMP 模型结构如下：
```
G(g0)-M(m0)-P(allp[0])  ... P(allp[n]) ... P(allp[GOMAXPROCS-1])
            |
            runq[ G(main goroutine) ]
```

### 6. [runtime.mstart()](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=1416-1465;bpv=0;bpt=1)

**当前线程 M0 进入调度循环**。一般情况下线程进入调度循环后不会再返回。进入调度循环的线程会去**执行上一步创建的 goroutine**。

主 goroutine 得到执行后，runtime.main() 会设置最大栈大小、启动监控线程 sysmon、初始化 runtime 包、开启 GC、最后初始化 mian 包并调用 main.main() 函数。

至此，初始化过程完毕。

## 调度循环

以 go 1.20 代码为例分析 [runtime.schedule()](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=3318-3388;drc=b35ee3b0467e042621aec9af7f18a2d8c63029ad)。

```go
// One round of scheduler: find a runnable goroutine and execute it.
// Never returns.
func schedule() {
    // 获取当前 G，执行 schedule() 函数时一般都是系统栈 g0。
    // mp 为当前线程 M。
    mp := getg().m

    // 当前 M 持有锁不进行调度
    if mp.locks != 0 {
        throw("schedule: holding locks")
    }

    // 当前 M 有没有和 G 绑定，如果有，这个 M 不能用来执行其他 G 了，只能挂起等待与之绑定的 G 得到调度。
    if mp.lockedg != 0 {
        stoplockedm()
        execute(mp.lockedg.ptr(), false) // Never returns.
    }

    // 当前 M 正在执行 cgo 函数调用，不能被调用。
    // 因为 cgo 函数调用占用 g0 栈空间
    if mp.incgo {
        throw("schedule: in cgo")
    }

top:
    // 获取当前 P
    pp := mp.p.ptr()
    // 标记字段，禁止对 P 的抢占
    pp.preempt = false


    // 安全检查，如果 MP 自旋状态，runnext 与 本地队列必须为空 
    // Check this before calling checkTimers, as that might call
    // goready to put a ready goroutine on the local run queue.
    if mp.spinning && (pp.runnext != 0 || pp.runqhead != pp.runqtail) {
        throw("schedule: spinning with local work")
    }

    // 找到可执行的 G
    gp, inheritTime, tryWakeP := findRunnable() // blocks until work is available

    // 当前 M 即将执行 G，如果此时时自旋状态，需要修改状态。可能会创建一个新的自旋 M
    if mp.spinning {
        resetspinning()
    }

    // 判断是否处于禁止调度用户协程的状态
    // 如果是系统协程，可以执行
    // 否则，需要暂存到 disable 队列中，调度逻辑跳转到 top 重新寻找可执行的 G
    if sched.disable.user && !schedEnabled(gp) {
        // Scheduling of this goroutine is disabled. Put it on
        // the list of pending runnable goroutines for when we
        // re-enable user scheduling and look again.
        lock(&sched.lock)
        if schedEnabled(gp) {
            // Something re-enabled scheduling while we
            // were acquiring the lock.
            unlock(&sched.lock)
        } else {
            sched.disable.runnable.pushBack(gp)
            sched.disable.n++
            unlock(&sched.lock)
            goto top
        }
    }

    // 尝试唤醒新的线程，以保证有足够的线程来调度 GCworker 和 tracereader
    if tryWakeP {
        wakep()
    }
    // 判断 G 是否有绑定的 M，如果有就唤醒 M 来执行 G
    if gp.lockedm != 0 {
        // Hands off own p to the locked m,
        // then blocks waiting for a new p.
        startlockedm(gp)
        goto top
    }

    // 如果 G 没有绑定 M 通过这个函数来执行。
    // 关联 gp 与 当前 M
    // 将 gp 状态设置为 _Grunning
    // 通过 gogo() 恢复上下文
    execute(gp, inheritTime)
}
```

### [runtime.findrunnable()](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=2657-2998;drc=b35ee3b0467e042621aec9af7f18a2d8c63029ad)

```go
// Finds a runnable goroutine to execute.
// Tries to steal from other P's, get g from local or global queue, poll network.
// tryWakeP indicates that the returned goroutine is not normal (GC worker, trace
// reader) so the caller should try to wake a P.
func findRunnable() (gp *g, inheritTime, tryWakeP bool) {
    // 获取当前 M
    mp := getg().m

top:
    // 获取当前 P
    pp := mp.p.ptr()
    // 如果处于 gcwaiting 和 runSafePointFn，后续逻辑可能会阻塞
    if sched.gcwaiting.Load() {
        gcstopm()
        goto top
    }
    if pp.runSafePointFn != 0 {
        runSafePointFn()
    }


    // 运行当前 P 上所有已经达到触发时间的计时器，可能会使一些 goroutine 从 _Gwaiting 变成 _Grunnable 状态
    // now and pollUntil are saved for work stealing later,
    // which may steal timers. It's important that between now
    // and then, nothing blocks, so these numbers remain mostly
    // relevant.
    now, pollUntil, _ := checkTimers(pp, 0)


    // 一般的 goroutine 切换至就绪状态时会通过 wakep() 函数按需启动新的线程，但是下面的两类不会，所以将 tryWakeP 返回值设置为 true
    
    // 尝试获取待运行的 Trace Reader
    if trace.enabled || trace.shutdown {
        gp := traceReader()
        if gp != nil {
            casgstatus(gp, _Gwaiting, _Grunnable)
            traceGoUnpark(gp, 0)
            return gp, false, true
        }
    }
    // 尝试获取待运行的 GC Worker
    if gcBlackenEnabled != 0 {
        gp, tnow := gcController.findRunnableGCWorker(pp, now)
        if gp != nil {
            return gp, false, true
        }
        now = tnow
    }

    // 每调度60次本地 runq，就会调度一次全局 runq
    // 避免 P 上两个互相唤起的 G 导致全局 runq 中的 G 永远不会被调度到。
    if pp.schedtick%61 == 0 && sched.runqsize > 0 {
        lock(&sched.lock)
        gp := globrunqget(pp, 1)
        unlock(&sched.lock)
        if gp != nil {
            return gp, false, false
        }
    }

    // 判断是否需要唤醒 finalizer G
    // fing 是存储在 runtime 包的一个全局变量，用来参与垃圾回收
    if fingStatus.Load()&(fingWait|fingWake) == fingWait|fingWake {
        if gp := wakefing(); gp != nil {
            ready(gp, 0, true)
        }
    }
    if *cgo_yield != nil {
        asmcgocall(*cgo_yield, nil)
    }

    // 从本地 runq 获取 G
    if gp, inheritTime := runqget(pp); gp != nil {
        return gp, inheritTime, false
    }

    // 从全局 runq 获取 G
    if sched.runqsize != 0 {
        lock(&sched.lock)
        gp := globrunqget(pp, 0)
        unlock(&sched.lock)
        if gp != nil {
            return gp, false, false
        }
    }

    // 执行一次非阻塞的 netpoll
    // 如果返回的列表非空，就把第一个 G 从列表中 pop 出来，修改状态为 _Grunnable
    // 其余的插入全局 runq
    // 返回 G
    if netpollinited() && netpollWaiters.Load() > 0 && sched.lastpoll.Load() != 0 {
        if list := netpoll(0); !list.empty() { // non-blocking
            gp := list.pop()
            injectglist(&list)
            casgstatus(gp, _Gwaiting, _Grunnable)
            if trace.enabled {
                traceGoUnpark(gp, 0)
            }
            return gp, false, false
        }
    }

    // 限制自旋 M 的数量在工作 P 数量的一半
    // This is necessary to prevent excessive CPU consumption when
    // GOMAXPROCS>>1 but the program parallelism is low.
    if mp.spinning || 2*sched.nmspinning.Load() < gomaxprocs-sched.npidle.Load() {
        if !mp.spinning {
            mp.becomeSpinning()
        }

        // 窃取逻辑
        gp, inheritTime, tnow, w, newWork := stealWork(now)
        if gp != nil {
            return gp, inheritTime, false
        }
        if newWork {
            // There may be new timer or GC work; restart to discover.
            goto top
        }

        now = tnow
        if w != 0 && (pollUntil == 0 || w < pollUntil) {
            // Earlier timer to wait for.
            pollUntil = w
        }
    }

    // 能走到此处，意味着目前确实没有业务 G 要本运行
    // 如果处于 GC 标记阶段，当前 M 可以从 gcBgMarkWorkerPool 获取空闲节点中的 G 去协助 GC 标记，而不是放弃 P。
    if gcBlackenEnabled != 0 && gcMarkWorkAvailable(pp) && gcController.addIdleMarkWorker() {
        node := (*gcBgMarkWorkerNode)(gcBgMarkWorkerPool.pop())
        if node != nil {
            pp.gcMarkWorkerMode = gcMarkWorkerIdleMode
            gp := node.gp.ptr()
            casgstatus(gp, _Gwaiting, _Grunnable)
            if trace.enabled {
                traceGoUnpark(gp, 0)
            }
            return gp, false, false
        }
        gcController.removeIdleMarkWorker()
    }

    // 后面还有逻辑暂且不表
    ......
```

### [runtime.stealWork()](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=3021-3094;drc=b35ee3b0467e042621aec9af7f18a2d8c63029ad)

```go
// stealWork attempts to steal a runnable goroutine or timer from any P.
//
// If newWork is true, new work may have been readied.
//
// If now is not 0 it is the current time. stealWork returns the passed time or
// the current time if now was passed as 0.
func stealWork(now int64) (gp *g, inheritTime bool, rnow, pollUntil int64, newWork bool) {
    
    // 获取当前 P
    pp := getg().m.p.ptr()

    ranTimer := false

    // 尝试4次窃取
    const stealTries = 4
    for i := 0; i < stealTries; i++ {
        // 一个标记，是否是第四次
        stealTimersOrRunNextG := i == stealTries-1

        for enum := stealOrder.start(fastrand()); !enum.done(); enum.next() {
            if sched.gcwaiting.Load() {
                // GC work may be available.
                return nil, false, now, pollUntil, true
            }
            // 从 allp 中随机拿一个 P2
            p2 := allp[enum.position()]
            // 如果 P2 与当前 P 相同，跳出，进入下一轮循环
            if pp == p2 {
                continue
            }

            // 如果当前是第四次循环，同时 p2 有 timer 到期
            if stealTimersOrRunNextG && timerpMask.read(enum.position()) {
                tnow, w, ran := checkTimers(p2, now)
                now = tnow
                if w != 0 && (pollUntil == 0 || w < pollUntil) {
                    pollUntil = w
                }
                // 运行过 timer
                if ran {
                    // 运行过计时器可能已经准备好了任意数量的 G，并将它们添加到 P 的本地 runq 中。
                    // 推翻了窃取的前置条件，本地 runq 未必有空间安放偷来的 P
                    // 所以先去从本地 runq 中获取 G
                    if gp, inheritTime := runqget(pp); gp != nil {
                        return gp, inheritTime, now, pollUntil, ranTimer
                    }
                    ranTimer = true
                }
            }

            // 如果 P2 空闲状态，别偷，偷不出来
            if !idlepMask.read(enum.position()) {
                if gp := runqsteal(pp, p2, stealTimersOrRunNextG); gp != nil {
                    return gp, false, now, pollUntil, ranTimer
                }
            }
        }
    }

    // No goroutines found to steal. Regardless, running a timer may have
    // made some goroutine ready that we missed. Indicate the next timer to
    // wait for.
    return nil, false, now, pollUntil, ranTimer
}
```

## 参考资料

- go 1.20 源码
- [简介G-P-M调度模型](https://mp.weixin.qq.com/s/1CY3E5daJ5U42orVwzCpaw)
- [深入golang runtime的调度](https://zboya.github.io/post/go_scheduler/#%E6%B7%B1%E5%85%A5golang-runtime%E7%9A%84%E8%B0%83%E5%BA%A6)
- [调度场景过程全解析](https://www.yuque.com/aceld/golang/srxd6d#5c3da99e)
