*你可能想提前 Google 的概念：宏内核、微内核、调试器、gdb*

**目标：暂停并稍微整理我们的代码。然后学习如何使用 gdb 调试内核**

也许你没有意识到，但你已经有自己的内核在运行了！

然而，它做的很少，只是打印一个 'X'。现在是时候停下来，将代码组织到文件夹中，为未来的代码创建一个可扩展的 Makefile，并思考一个策略。

看看新的文件夹结构。大多数文件都从之前的课程中符号链接，所以如果我们在某个时候必须更改它们，删除符号链接并创建一个新文件会是一个更好的主意。

此外，由于从现在开始我们将主要使用 C 来编码，我们将利用 qemu 打开到 gdb 的连接的能力。首先，让我们安装一个交叉编译的 `gdb`，因为 OSX 使用 `lldb`，它与 ELF 文件格式不兼容（Homebrew 仓库中可用的 `gdb` 也不兼容）

```sh
cd /tmp/src
curl -O http://ftp.rediris.es/mirror/GNU/gdb/gdb-7.8.tar.gz
tar xf gdb-7.8.tar.gz
mkdir gdb-build
cd gdb-build
export PREFIX="/usr/local/i386elfgcc"
export TARGET=i386-elf
../gdb-7.8/configure --target="$TARGET" --prefix="$PREFIX" --program-prefix=i386-elf-
make
make install
```

查看 Makefile 目标 `make debug`。此目标使用构建 `kernel.elf`，这是一个对象文件（不是二进制文件），包含我们在内核上生成的所有符号，这要归功于 gcc 上的 `-g` 标志。请使用 `xxd` 检查它，你会看到一些字符串。实际上，检查对象文件中字符串的正确方法是 `strings kernel.elf`

我们可以利用这个很酷的 qemu 功能。输入 `make debug`，在 gdb shell 中：

- 在 `kernel.c:main()` 设置断点：`b main`
- 运行操作系统：`continue`
- 在代码中运行两步：`next` 然后 `next`。你会看到我们即将在屏幕上设置 'X'，但它还不在那里（查看 qemu 屏幕）
- 让我们看看视频内存中有什么：`print *video_memory`。那里有来自"Landed in 32-bit Protected Mode"的 'L'
- 嗯，让我们确保 `video_memory` 指向正确的地址：`print video_memory`
- `next` 在那里放置我们的 'X'
- 让我们确保：`print *video_memory` 并查看 qemu 屏幕。它肯定在那里。

现在是阅读一些关于 `gdb` 的教程并学习超级有用的东西的好时机，比如 `info registers`，这将在未来为我们节省大量时间！


你可能会注意到，由于这是一个教程，我们还没有讨论我们将编写哪种类型的内核。它可能是一个宏内核，因为它们更容易设计和实现，毕竟这是我们的第一个操作系统。也许在未来我们会添加一个带有微内核设计的课程"15-b"。谁知道呢。
