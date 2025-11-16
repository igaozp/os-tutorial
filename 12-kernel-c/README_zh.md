*你可能想提前 Google 的概念：C、目标代码、链接器、反汇编*

**目标：学习使用 C 编写与汇编相同的低级代码**


编译
-------

让我们看看 C 编译器如何编译我们的代码，并将其与汇编器生成的机器码进行比较。

我们将开始编写一个包含函数的简单程序，`function.c`。打开文件并检查它。

要编译系统独立的代码，我们需要标志 `-ffreestanding`，所以以这种方式编译 `function.c`：

`i386-elf-gcc -ffreestanding -c function.c -o function.o`

让我们检查编译器生成的机器码：

`i386-elf-objdump -d function.o`

现在这是我们认识的东西，不是吗？


链接
----

最后，为了生成二进制文件，我们将使用链接器。这一步的重要部分是学习高级语言如何调用函数标签。我们的函数将被放置在内存中的哪个偏移量？我们实际上不知道。对于这个例子，我们将把偏移量放在 `0x0`，并使用 `binary` 格式，它生成没有任何标签和/或元数据的机器码

`i386-elf-ld -o function.bin -Ttext 0x0 --oformat binary function.o`

*注意：链接时可能会出现警告，忽略它*

现在使用 `xxd` 检查两个"二进制"文件，`function.o` 和 `function.bin`。你会看到 `.bin` 文件是机器码，而 `.o` 文件有很多调试信息、标签等。


反编译
---------

出于好奇，我们将检查机器码。

`ndisasm -b 32 function.bin`


更多
----

我鼓励你编写更多的小程序，其中包括：

- 局部变量 `localvars.c`
- 函数调用 `functioncalls.c`
- 指针 `pointers.c`

然后编译并反汇编它们，并检查生成的机器码。按照 os-guide.pdf 进行解释。尝试回答这个问题：为什么 `pointers.c` 的反汇编不像你期望的那样？"Hello" 的 ASCII `0x48656c6c6f` 在哪里？
