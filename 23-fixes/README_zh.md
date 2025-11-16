*你可能想提前 Google 的概念：freestanding、uint32_t、size_t*

**目标：修复我们代码中的各种问题**

OSDev wiki 有一个[描述 JamesM 教程中一些问题的部分](http://wiki.osdev.org/James_Molloy%27s_Tutorial_Known_Bugs)。由于我们在第 18-22 课（从中断到 malloc）中遵循了他的教程，我们需要确保在继续之前修复任何问题。

1. 错误的 CFLAGS
---------------

我们在编译 `.o` 文件时添加 `-ffreestanding`，其中包括 `kernel_entry.o`，因此包括 `kernel.bin` 和 `os-image.bin`。

之前，我们通过使用 `-nostdlib` 禁用了 libgcc（不是 libc），并且我们没有在链接时重新启用它。由于这很棘手，我们将删除 `-nostdlib`

`-nostdinc` 也被传递给 gcc，但我们将在步骤 3 中需要它，所以让我们删除它。


2. kernel.c `main()` 函数
-----------------------------

修改 `kernel/kernel.c` 并将 `main()` 更改为 `kernel_main()`，因为 gcc 将"main"识别为特殊关键字，我们不想弄乱它。

相应地更改 `boot/kernel_entry.asm` 以指向新名称。

要修复 `i386-elf-ld: warning: cannot find entry symbol _start; defaulting to 0000000000001000` 警告消息，添加一个 `global _start;` 并在 `boot/kernel_entry.asm` 中定义 `_start:` 标签。


3. 重新发明的数据类型
-----------------------

看起来定义非标准数据类型如 `u32` 等是一个坏主意，因为 C99 引入了标准的固定宽度数据类型，如 `uint32_t`

我们需要包含 `<stdint.h>`，它甚至在 `-ffreestanding` 中工作（但需要 stdlibs），并使用这些数据类型而不是我们自己的，然后在 `type.h` 上删除它们

我们还删除了 `__asm__` 和 `__volatile__` 周围的下划线，因为它们不是必需的。


4. 未正确对齐的 `kmalloc`
-------------------------------

首先，由于 `kmalloc` 使用大小参数，我们将使用正确的数据类型 `size_t` 而不是 `u32int_t`。`size_t` 应该用于所有"计数"内容且不能为负数的参数。包含 `<stddef.h>`。

我们将在未来修复我们的 `kmalloc`，使其成为一个适当的内存管理器并对齐数据类型。目前，它将始终返回一个新的页面对齐的内存块。


5. 缺少的函数
--------------------

我们将在后续课程中实现缺少的 `mem*` 函数


6. 中断处理程序
---------------------
`cli` 是冗余的，因为我们已经在 IDT 条目上建立了使用 `idt_gate_t` 标志在处理程序中是否启用中断。

`sti` 也是冗余的，因为 `iret` 从栈中加载 eflags 值，其中包含一个位，告诉中断是开还是关。换句话说，中断处理程序会自动恢复中断，无论此中断之前中断是否启用

在 `cpu/isr.h` 上，`struct registers_t` 有几个问题。首先，所谓的 `esp` 被重命名为 `useless`。该值是无用的，因为它与当前栈上下文有关，而不是被中断的内容。然后，我们将 `useresp` 重命名为 `esp`

我们在 `cpu/interrupt.asm` 上的 `call isr_handler` 之前添加 `cld`，如 osdev wiki 所建议的。

`cpu/interrupt.asm` 有一个最终的重要问题。通用存根在栈上创建 `struct registers` 的实例，然后调用 C 处理程序。但这破坏了 ABI，因为栈属于被调用的函数，它们可以随意更改它们。需要将结构体作为指针传递。

为此，编辑 `cpu/isr.h` 和 `cpu/isr.c` 并将 `registers_t r` 更改为 `registers_t *t`，然后，不是通过 `.` 访问结构体的字段，而是通过 `->` 访问指针的字段。最后，在 `cpu/interrupt.asm` 中，在调用 `isr_handler` 和 `irq_handler` 之前添加一个 `push esp` -- 记得也要 `pop eax` 以在之后清除指针。

两个当前回调，定时器和键盘，也需要更改为使用指向 `registers_t` 的指针。
