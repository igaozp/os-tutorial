*你可能想提前 Google 的概念：linux、mac、终端、编译器、模拟器、nasm、qemu*

**目标：安装运行本教程所需的软件**

我在 Mac 上工作，不过 Linux 更好，因为它已经为你提供了所有标准工具。

在 Mac 上，[安装 Homebrew](http://brew.sh)，然后运行 `brew install qemu nasm`

如果你已经安装了 Xcode 开发者工具的 `nasm`，不要使用它，因为在大多数情况下它不起作用。始终使用 `/usr/local/bin/nasm`

在某些系统上，qemu 被拆分为多个二进制文件。你可能需要调用 `qemu-system-x86_64 binfile`
