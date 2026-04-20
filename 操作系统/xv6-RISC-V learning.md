 # 🔍 启动流程探索：从 QEMU 加载到第一个用户进程

让我们按时间线追踪 xv6 的启动过程：

---

## 🚀 阶段 1：QEMU 加载内核（kernel/entry.S）

代码位置：kernel/entry.S:1-22

asm

```
_entry:
    la sp, stack0           # 加载栈地址
    li a0, 1024*4           # 每个CPU栈大小4096字节
    csrr a1, mhartid        # 读取硬件线程ID
    addi a1, a1, 1
    mul a0, a0, a1
    add sp, sp, a0          # sp = stack0 + (hartid+1)*4096
    call start              # 跳转到start()
```

发生了什么：

- QEMU 将内核镜像加载到物理地址 0x80000000
- 每个 CPU 核心（hart）从这里开始执行
- 设置每个 CPU 独立的栈空间（避免踩栈）
- 跳转到 C 函数 start ()（在 start.c 中）

为什么重要：这是 CPU 看到的第一行代码，建立了 C 语言运行所需的基本环境（栈）。

---

## 🏗️ 阶段 2：机器模式初始化（kernel/start.c）

代码位置：kernel/start.c:14-49

c

运行

```
void start() {
  // 设置M模式→S模式切换
  x &= ~MSTATUS_MPP_MASK;
  x |= MSTATUS_MPP_S;
  w_mstatus(x);

  w_mepc((uint64)main);      // 设置异常返回地址为main()
  w_satp(0);                 // 暂时禁用分页

  // 委托所有中断/异常给S模式
  w_medeleg(0xffff);
  w_mideleg(0xffff);

  timerinit();               // 设置时钟中断
  asm volatile("mret");      // 切换到S模式，跳转到main()
}
```

发生了什么：

1. 特权级切换：从机器模式（M）切换到监管者模式（S）
2. 中断委托：将所有中断和异常处理委托给 S 模式
3. 定时器设置：启用时钟中断，为抢占式调度做准备
4. 跳转：通过 mret 指令 "返回" 到 main () 函数

关键点：RISC-V 启动时处于 M 模式，xv6 大部分时间运行在 S 模式。

---

## 🏛️ 阶段 3：内核主初始化（kernel/main.c）

代码位置：kernel/main.c:10-45

c

运行

```
void main() {
  if(cpuid() == 0) {         // 只有CPU0执行初始化
    consoleinit();           // 控制台
    kinit();                 // 物理页分配器
    kvminit();               // 内核页表
    kvminithart();           // 启用分页
    procinit();              // 进程表
    trapinit();              // 陷阱向量
    trapinithart();          // 安装内核陷阱处理
    plicinit();              // 中断控制器
    plicinithart();          // 启用设备中断
    binit();                 // 缓冲区缓存
    iinit();                 // inode表
    fileinit();              // 文件表
    virtio_disk_init();      // 虚拟磁盘
    userinit();              // ⭐创建第一个用户进程⭐
    started = 1;
  } else {
    // 其他CPU等待CPU0完成初始化
    while(started == 0);
    kvminithart();           // 启用分页
    trapinithart();          // 陷阱处理
    plicinithart();          // 中断
  }

  scheduler();               // 进入调度器循环
}
```

初始化顺序（关键依赖链）：

1. 内存子系统：kinit () → kvminit () → 物理页分配→内核页表
2. 进程管理：procinit () → 初始化 proc [NPROC] 数组
3. 中断系统：trapinit () → plicinit () → 设置异常和中断处理
4. 文件系统：binit () → iinit () → fileinit () → 磁盘缓存和元数据
5. 设备驱动：virtio_disk_init () → 虚拟磁盘
6. 用户进程：userinit () → 创建第一个进程

为什么只有 CPU0 初始化：避免并发初始化导致的竞态条件。

---

## 👶 阶段 4：创建第一个用户进程（kernel/proc.c）

代码位置：kernel/proc.c:219-232 + allocproc ()

c

运行

```
// userinit() - 第一个用户进程的特殊初始化
void userinit(void) {
  struct proc *p;
  p = allocproc();           // 分配进程结构体
  initproc = p;              // 全局指针，指向init进程

  p->cwd = namei("/");       // 根目录作为当前目录

  // ⚠️ 注意：这里缺少了设置trapframe的代码！
  // 实际代码中应该设置p->trapframe来指定初始用户态PC

  p->state = RUNNABLE;       // 标记为可运行
  release(&p->lock);
}
```

allocproc () 做了什么（proc.c:110-150）：

c

运行

```
static struct proc* allocproc(void) {
  // 1. 在proc[NPROC]数组中找UNUSED的槽位
  // 2. 分配trapframe页面（用于保存用户寄存器）
  p->trapframe = (struct trapframe *)kalloc();

  // 3. 创建空用户页表（含trampoline和trapframe映射）
  p->pagetable = proc_pagetable(p);

  // 4. 设置上下文：返回到forkret，最终进入用户空间
  p->context.ra = (uint64)forkret;
  p->context.sp = p->kstack + PGSIZE;
}
```

关键数据结构（kernel/proc.h）：

- struct proc：进程控制块（PID、状态、页表、打开文件等）
- struct trapframe：陷阱帧，保存用户寄存器
- struct context：内核上下文，用于 swtch () 切换

---

## 🔄 阶段 5：调度器启动（kernel/proc.c → scheduler ()）

代码位置：main.c:44 调用 scheduler ()

c

运行

```
scheduler(void) {
  struct proc *p;
  for(;;) {
    // 遍历进程表，找RUNNABLE进程
    for(p = proc; p < &proc[NPROC]; p++) {
      acquire(&p->lock);
      if(p->state == RUNNABLE) {
        // 切换到这个进程
        p->state = RUNNING;
        c->proc = p;
        swtch(&c->context, &p->context);
        c->proc = 0;
      }
      release(&p->lock);
    }
  }
}
```

此时发生了什么：

1. scheduler () 找到 state == RUNNABLE 的 init 进程
2. 通过 swtch () 切换到该进程的上下文
3. 由于上下文指向 forkret，最终会 "返回" 用户空间
4. init 进程开始执行用户态代码

---

## 📊 完整启动流程图示

plaintext

```
QEMU加载内核
    ↓
0x80000000: entry.S (汇编)
    ↓ 设置栈，call start
start.c (机器模式)
    ↓ 配置特权级，mret
main.c (监管者模式)
    ├── CPU0: 初始化所有子系统
    │       └── userinit() → 创建init进程
    └── 其他CPU: 等待，然后启用分页/中断
    ↓
scheduler() 调度循环
    ↓
swtch() 切换到init进程
    ↓
forkret → usertrapret → userret (trampoline.S)
    ↓
用户空间：init进程开始执行
```

---

## 🔎 发现的问题与思考点

在阅读代码时，我注意到一个有趣的现象：

问题：userinit () 函数（第 220-232 行）看起来不完整！它没有设置：

1. 初始程序计数器（PC）应该指向哪里？
2. 用户栈指针应该设置为什么？
3. 如何从内核 "返回" 用户空间？

可能的解释：

- 实际代码可能更长，我们只看到了部分
- 也许 allocproc () 或 proc_pagetable () 设置了默认值
- 或者第一个进程有特殊的启动路径

下一步探索：让我们用 grep 查找 init 进程的实际初始设置