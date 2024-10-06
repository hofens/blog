> https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/gc-introduction-V5


当前主流编程语言实现的 GC 算法主要分为两大类：引用计数和对象追踪（即 Tracing GC）。ArkTS 运行时中就是基于分代模型和混合算法来实现不同场景下内存回收的高性能表现。
在 ArkTS 中，数据类型分为两类，简单类型和引用类型。简单类型内容直接保存在栈（Stack）中，由操作系统自动分配和释放。引用类型保存在堆（heap）中，需要引擎进行手动释放。GC 就是针对堆空间的内存自动回收的管理机制。

- SemiSpace：年轻代（Young Generation），存放新创建出来的对象，存活率低，主要使用 copying 算法进行内存回收。
- OldSpace：老年代（Old Generation），存放年轻代多次回收仍存活的对象会被复制到该空间，根据场景混合多种算法进行内存回收。
- HugeObjectSpace：大对象空间，使用单独的region存放一个大对象的空间。
- ReadOnlySpace：只读空间，存放运行期间的只读数据。
- NonMovableSpace：不可移动空间，存放不可移动的对象。
- SnapshotSpace：快照空间，转储堆快照时使用的空间。
- MachineCodeSpace：机器码空间，存放程序机器码。