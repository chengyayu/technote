# G-P-M 模型

## 模型结构

- G（[goroutine](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/runtime2.go;l=407)）
- P（[processor](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/runtime2.go;l=609)）
- M（[Machine Thread](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/runtime2.go;l=526)）

## 调度策略

P 在执行调度时，会按照以下优先级与顺序进行：

1. 先尝试从 runnext 中获取下一个可执行的 G，如果 runnext 为空，则继续从当前 P 的本地运行队列 runq 中获取需要执行的 G。
2. 如果本地运行队列为空，则尝试从全局队列中获取需要执行的 G。
3. 如果全局队列为空，则尝试从其他 P 的本地队列中窃取 G。
4. 如果窃取不到，当前 P 与 M 解绑，P 被放入空闲队列中，M 进入休眠状态。

[schedule, src/runtime/proc.go](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=3318-3388;drc=ffb07d0c66db2f3f33faedf2927f9aa476d47720)

### 1. 从本地队列获取

如果 runnext 不为空则返回对应的 G，如果为空则继续从 P 的本地运行队列 runq 中获取需要执行的 G。因为可能存在其他 P 窃取任务造成的同时访问情况，所以这里的获取操作都需要**加锁**。

[runqget, src/runtime/proc.go](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=6070-6095;bpv=0;bpt=0)

### 2. 从全局队列获取

当 P 每执行61次调度，或者本地队列中不存在可用的 G 时，从全局队列获取一批 G 到本地队列中。

全局队列的数据结构是一个双端队列。由于多个 P 共享 全局队列，先根据 P 的数量平分全局队列中的 G，同时要转移数量不能超过本地队列容量一半（256/2=128），再通过循环调用 runqput 将全局队列中的 G 放入 P 的本地队列中。如果 P 的本地队列已满，调度器会将本地队列中的一半 G 放入全局队列。

[globrunqget, src/runtime/proc.go](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=5753-5782;bpv=1;bpt=1)

### 3. 获取准备就绪的网络协程

窃取前的一个小优化，在本地队列和全局队列都找不到可用 G 的情况下，调度器会寻找当前是否有已经准备好的网络协程。runtime.netpoll 获取当前可运行的协程列表。返回第一个 G，并通过 injectglist 将其余 G 放入全局队列。

[2740-2757, src/runtime/proc.go](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=2740-2757;drc=ffb07d0c66db2f3f33faedf2927f9aa476d47720;bpv=0;bpt=1)

### 4. 从其他 P 本地队列窃取

自旋PM（前三部分都未获得可用 G），需要从其他 P 的本地队列中窃取可用的 G。

1）先选 P

[allp []*p](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/runtime2.go;l=1150;bpv=0;bpt=1) 这个全局变量存储着所有的 P。为了保证公平性，Go 语言采用[一种方式](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=3043-3047;bpv=0;bpt=1)既保证了随机性又保证了每个 P 都能被遍历一次。

2）再偷 G

找到窃取目标 P1 后，会将 P1 本地队列中的一半 G 放入自己的本地队列中。具体通过 [runqgrab](https://cs.opensource.google/go/go/+/refs/tags/go1.20:src/runtime/proc.go;l=6136-6190;bpv=0;bpt=1) 实现。

## 调度时机