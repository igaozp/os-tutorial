*你可能想提前 Google 的概念：内核、ELF 格式、makefile*

**目标：创建一个简单的内核和一个能够启动它的引导扇区**

内核
----------

我们的 C 内核将只在屏幕的左上角打印一个 'X'。继续打开 `kernel.c`。

你会注意到一个什么都不做的虚拟函数。该函数将迫使我们创建一个内核入口例程，它不指向我们内核中的字节 0x0，而是指向我们知道启动它的实际标签。在我们的例子中，函数 `main()`。

`i386-elf-gcc -ffreestanding -c kernel.c -o kernel.o`

该例程编码在 `kernel_entry.asm` 中。阅读它，你将学习如何在汇编中使用 `[extern]` 声明。要编译此文件，不是生成二进制文件，而是生成一个 `elf` 格式文件，它将与 `kernel.o` 链接

`nasm kernel_entry.asm -f elf -o kernel_entry.o`


链接器
----------

链接器是一个非常强大的工具，我们才刚刚开始从中受益。

要将两个目标文件链接到单个二进制内核并解析标签引用，运行：

`i386-elf-ld -o kernel.bin -Ttext 0x1000 kernel_entry.o kernel.o --oformat binary`

注意我们的内核将不会放在内存中的 `0x0`，而是 `0x1000`。引导扇区也需要知道这个地址。


引导扇区
--------------

它与第 10 课中的非常相似。打开 `bootsect.asm` 并检查代码。实际上，如果你删除所有用于在屏幕上打印消息的行，它只有几十行。

使用 `nasm bootsect.asm -f bin -o bootsect.bin` 编译它


将它们放在一起
-----------------------

现在怎么办？我们有两个单独的文件用于引导扇区和内核？

我们不能只是将它们"链接"到一个文件中吗？是的，我们可以，而且很容易，只需连接它们：

`cat bootsect.bin kernel.bin > os-image.bin`


运行！
----

你现在可以使用 qemu 运行 `os-image.bin`。

记住，如果你发现磁盘加载错误，你可能需要调整磁盘编号或 qemu 参数（软盘 = `0x0`，硬盘 = `0x80`）。我通常使用 `qemu-system-i386 -fda os-image.bin`

你将看到四条消息：

- "Started in 16-bit Real Mode"
- "Loading kernel into memory"
- （左上角）"Landed in 32-bit Protected Mode"
- （左上角，覆盖之前的消息）"X"

恭喜！


Makefile
--------

作为最后一步，我们将使用 Makefile 整理编译过程。打开 `Makefile` 脚本并检查其内容。如果你不知道 Makefile 是什么，现在是 Google 并学习它的好时机，因为这将在未来为我们节省大量时间。
