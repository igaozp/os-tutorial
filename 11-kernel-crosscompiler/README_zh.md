*你可能想提前 Google 的概念：交叉编译器*

**目标：创建一个开发环境来构建你的内核**

如果你使用 Mac，你需要立即执行此过程。否则，它可以再等几节课。无论如何，一旦我们跳转到使用更高级语言（即 C）进行开发，你将需要一个交叉编译器。[阅读原因](http://wiki.osdev.org/Why_do_I_need_a_Cross_Compiler%3F)

我将改编 [OSDev wiki 上的说明](http://wiki.osdev.org/GCC_Cross-Compiler)。


所需包
-----------------

首先，安装所需的包。在 Linux 上，使用你的包分发版。在 Mac 上，如果你在第 00 课中没有这样做，请[安装 brew](http://brew.sh/)，并使用 `brew install` 获取这些包

- gmp
- mpfr
- libmpc
- gcc

是的，我们需要 `gcc` 来构建我们的交叉编译 `gcc`，特别是在 Mac 上，gcc 已被弃用，改用 `clang`

安装后，找到你的打包 gcc 的位置（记住，不是 clang）并导出它。例如：

```
export CC=/usr/local/bin/gcc-4.9
export LD=/usr/local/bin/gcc-4.9
```

我们需要构建 binutils 和一个交叉编译的 gcc，我们将把它们放入 `/usr/local/i386elfgcc`，所以现在让我们导出一些路径。随意更改为你喜欢的。

```
export PREFIX="/usr/local/i386elfgcc"
export TARGET=i386-elf
export PATH="$PREFIX/bin:$PATH"
```

binutils
--------

记住：在从互联网粘贴大段文本之前，始终要小心。我建议逐行复制。

```sh
mkdir /tmp/src
cd /tmp/src
curl -O http://ftp.gnu.org/gnu/binutils/binutils-2.24.tar.gz # 如果链接 404，请寻找更新的版本
tar xf binutils-2.24.tar.gz
mkdir binutils-build
cd binutils-build
../binutils-2.24/configure --target=$TARGET --enable-interwork --enable-multilib --disable-nls --disable-werror --prefix=$PREFIX 2>&1 | tee configure.log
make all install 2>&1 | tee make.log
```

gcc
---
```sh
cd /tmp/src
curl -O https://ftp.gnu.org/gnu/gcc/gcc-4.9.1/gcc-4.9.1.tar.bz2
tar xf gcc-4.9.1.tar.bz2
mkdir gcc-build
cd gcc-build
../gcc-4.9.1/configure --target=$TARGET --prefix="$PREFIX" --disable-nls --disable-libssp --enable-languages=c --without-headers
make all-gcc
make all-target-libgcc
make install-gcc
make install-target-libgcc
```

就是这样！你应该在 `/usr/local/i386elfgcc/bin` 有所有 GNU binutils 和编译器，前缀为 `i386-elf-` 以避免与系统的编译器和 binutils 冲突。

你可能想将 `$PATH` 添加到你的 `.bashrc`。从现在开始，在本教程中，我们将在使用交叉编译的 gcc 时明确使用前缀。
