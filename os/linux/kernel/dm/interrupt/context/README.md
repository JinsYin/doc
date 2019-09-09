# 中断上下文（Interrupt Context）

当执行一个中断处理程序时，内核处于中断上下文（interrupt context）。因为没有后备进程，所以中断上下文不可睡眠，否则有怎能对它重新调度呢？

中断上下文偶尔也称作 _原子上下文_，即该上下文的执行代码不允许阻塞。


中断上下文具有较为严格的时间限制，因为它打断了其他代码。中断上下文

中断处理程序不在进程上下文执行，而在一个与所有进程无关的、专门的中断上下文执行。之所以存在这样一个专门的执行环境，是为了保证中断处理程序能够在第一时间响应和处理中断请求，然后快速退出。