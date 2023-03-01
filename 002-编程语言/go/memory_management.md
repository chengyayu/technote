# 内存管理

## 分割

- **Page** x64 下 8KB 大小的内存块为1个 Page。
- **Span 是内存管理的基本单位**，代码中为 `mspan`，一组连续的 Page 组成1个 Span。根据一组中 Page 数量不同分为不同的 size 类别。
- **mcache 保存的是各类 size 的 Span，并按 Span class 分类**，小对象直接从 mcache 分配内存，它起到了缓存的作用，并且可以无锁访问。
    - 每个 P 拥有一个 mcache。
    - 每个级别的 Span 有两个链表：
        - scan 包含指针的对象。
        - noscan 不包含指针的对象。
- **mcentral 是所有线程共享的缓存，需要加锁访问**，它按 Span class 对 Span 分类，串联成链表，当 mcache 的某个级别 Span 的内存被分配光时，它会向 mcentral 申请1个当前级别的 Span。
- **mheap** 它是堆内存的抽象，把从 OS 申请出的内存页组织成 Span，并保存起来。当 mcentral 的 Span 不够用时会向 mheap 申请，mheap 的 Span 不够用时会向 OS 申请，向 OS 的内存申请是按页来的，然后把申请来的内存页生成 Span 组织起来，同样也是需要加锁访问的。mheap把Span组织成了树结构。并且还是2棵树，然后把Span分配到heapArena进行管理，它包含地址映射和span是否包含指针等位图，这样做的主要原因是为了更高效的利用内存：分配、回收和再利用。

## 分层

## 分配

## 参考资料

- [图解Go内存分配器](https://tonybai.com/2020/02/20/a-visual-guide-to-golang-memory-allocator-from-ground-up/)
- [Go内存分配那些事](https://lessisbetter.site/2019/07/06/go-memory-allocation/)
- [TCMalloc解密](https://wallenwang.com/2018/11/tcmalloc/)

