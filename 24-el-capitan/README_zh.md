**目标：将我们的构建系统更新到 El Capitan**

如果你从一开始就遵循本指南，并仅升级到 El Capitan 才发现 Makefile 不再编译，请按照这些说明升级你的交叉编译器。

否则，继续下一课

升级交叉编译器
----------------------------

我们将或多或少地遵循与第 11 课相同的说明。

首先，运行 `brew upgrade`，你将获得升级到 5.0 版本的 gcc（在编写本指南时）

然后运行 `xcode-select --install` 以更新 OSX 命令行工具

安装后，找到你的打包 gcc 的位置（记住，不是 clang）并导出它。例如：

```
export CC=/usr/local/bin/gcc-5
export LD=/usr/local/bin/gcc-5
```

我们需要重新编译 binutils 和我们的交叉编译 gcc。导出目标和前缀：

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
curl -O http://mirror.bbln.org/gcc/releases/gcc-4.9.1/gcc-4.9.1.tar.bz2
tar xf gcc-4.9.1.tar.bz2
mkdir gcc-build
cd gcc-build
../gcc-4.9.1/configure --target=$TARGET --prefix="$PREFIX" --disable-nls --disable-libssp --enable-languages=c --without-headers
make all-gcc
make all-target-libgcc
make install-gcc
make install-target-libgcc
```


现在尝试在本课的文件夹中键入 `make` 并检查一切是否顺利编译
