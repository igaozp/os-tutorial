*你可能想提前 Google 的概念：IRQs、PIC、轮询*

**目标：完成中断实现和 CPU 定时器**

当 CPU 启动时，PIC 将 IRQ 0-7 映射到 INT 0x8-0xF，将 IRQ 8-15 映射到 INT 0x70-0x77。这与我们上一课编程的 ISR 冲突。由于我们编程了 ISR 0-31，标准做法是将 IRQ 重新映射到 ISR 32-47。

PIC 通过 I/O 端口进行通信（参见第 15 课）。主 PIC 的命令端口为 0x20，数据端口为 0x21，而从 PIC 的命令端口为 0xA0，数据端口为 0xA1。

重新映射 PIC 的代码很奇怪，包括一些掩码，所以如果你好奇的话，请查看[这篇文章](http://www.osdev.org/wiki/PIC)。否则，只需查看 `cpu/isr.c`，在我们为 ISR 设置 IDT 门之后的新代码。之后，我们为 IRQ 添加 IDT 门。

现在我们跳到汇编，在 `interrupt.asm`。第一个任务是为我们刚刚在 C 代码中使用的 IRQ 符号添加全局定义。查看 `global` 语句的末尾。

然后，添加 IRQ 处理程序。同样在 `interrupt.asm` 的底部。注意它们如何跳转到一个新的通用存根：`irq_common_stub`（下一步）

然后我们创建这个 `irq_common_stub`，它与 ISR 的非常相似。它位于 `interrupt.asm` 的顶部，它还定义了一个新的 `[extern irq_handler]`

现在回到 C 代码，在 `isr.c` 中编写 `irq_handler()`。它向 PIC 发送一些 EOI 并调用适当的处理程序，该处理程序存储在名为 `interrupt_handlers` 的数组中，并在文件顶部定义。新的结构体在 `isr.h` 中定义。我们还将使用一个简单的函数来注册中断处理程序。

这是一项很大的工作，但现在我们可以定义我们的第一个 IRQ 处理程序！

`kernel.c` 中没有更改，因此没有新的东西可以运行和查看。请继续下一课以查看那些闪亮的新 IRQ。
