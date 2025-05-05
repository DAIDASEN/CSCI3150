==**Semaphore**== 函数用法 `<semaphore.h>`
`sem_t *sem_open(const char *name, oflag, mode_t mode, unsigned int value);` oflag打开标志`O_CREAT`创建  `O_EXCL`独占 `mode`权限模式(0666) `value`初始值 
`int sem_wait(sem_t *sem);` 信号量>0, -1就返回, =0阻塞直到可用 `int sem_post(sem_t *sem);`信号量+1, 如有等待则唤醒 `sem_close(sem);`关闭当前进程对信号量的引用`int sem_unlink(const char *name);` 标记信号被删除, 所有进程都关闭他就实际删除
<font color=blue>**Unnamed Semaphore**</font> used only by threads in the **same process** or threads in different processes but have **mapped the same memory** into their address spaces
`int sem_init(sem_t *sem, int pshared, unsigned int value);` `pshared` = 0表示当前process使用, 非0则多进程共享 `int sem_destroy(sem_t *sem);`
<font color=blue>**Shared Memory**</font> 共享内存的文件应该编译时用 `-lrt` 链接
`int shm_open(const char *name, int oflag, mode_t mode);` 创建或打开共享内存对像<font color=green>`oflag`</font>为(`O_RDONLY`只读`O_RDWR` 读写 `O_CREAT`如不存在则创建, | 连接) , 有问题返回-1 <font color=green>**`mode`**</font>权限模式（当O_CREAT标志使用时），如0644
`void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);` 将文件或设备映射到内存中 <font color=green>`addr`</font>映射起始地址, NULL/0系统自己选择 <font color=green>`length`</font>映射自己长度 <font color=green>`prot`</font>内存保护标志(`PROT_READ` 可读 `PROT_WRITE` 可写 `PROT_EXEC` 可执行`PROT_NONE` 不可访问) <font color=green>`flags`</font>(MAP_SHARED 共享映射，修改对其他进程可见; MAP_PRIVATE 私有映射，修改不会写回; MAP_ANONYMOUS 不基于文件的映射$\rarr$与fd=-1一起用) <font color=green>`fd`</font>文件表描述符(`shm_open`返回那个) `offset`文件偏移量通常为0
`int munmap(void *addr, size_t length);` 取消映射 `int shm_unlink(const char *name);` 删除共享内存对像
**==lock-free==**
<font color=red>**Advantages:** </font> 1. Usable where locks must be avoided, like in interrupt handlers 2. Prevents blocking issues such as deadlocks and priority inversion 3. Enhances performance on multi-core processors
<font color=blue>**Atomic Instruction**</font>:  `<stdatomic.h>`  在定义变量前+`_Atomic`使其为原子化的
`_Bool atmoic_compare_exchange_weak(volatile A *object, C *expected, C desired);` 比较object指向的值是不是等于expected, 如果是用desired替换object的值, 不是就用object替换expected, A为atomic type, C为non-atomic counterpart. <font color = green>volatile</font> 类似const是type的一个property , 表示访问之前可能会发生变化, 即使没有比修修改. <font color=green>函数返回值是比较的结果</font>
`atomic_load(x)`给x做一个本地的保存
<font color=blue>lock-free stack</font>
`typedef struct _Node{int data; struct _Node *next;} Node;`
`typedef struct _lfstack_t{ Node *head;} lfstack_t;`
![image-20250505182000844](.\Images\image-20250505182000844.png)
但这会存在<font color=green>**ABA problem** </font>occurs in lock-free programming when a thread reads value A, gets preempted, and other threads change the value from A to B and back to A. When the first thread resumes, its compare-and-swap operation succeeds incorrectly because it can't detect these intermediate changes. **Solution:** adds a **counter alongside** each pointer(之前stack的例子中lfstack_t加一个tag, 每次+1). This ensures that when comparing values, both the pointer and its associated counter must match, effectively detecting any intermediate changes.
<font color=blue>Memory Fence</font> 操作器可能会内存操作reordering, 产生一些奇怪的问题, 但线程中无害但多线程有问题$\Rarr$ 加入frence限制部分的顺序

