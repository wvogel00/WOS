# OSを作る
[OSDevサイト](https://wiki.osdev.org/Bare_Bones)
をなぞりながら、OS開発を進める

## クロスコンパイラ設定
[OSDevのクロスコンパイラ構築ページ](https://wiki.osdev.org/GCC_Cross-Compiler)
のとおりに進める

* boot用アセンブラビルド時
```asm
i686-elf-as boot.s -o boot.o
```
* kernelビルド時
```
i686-elf-gcc -c kernel.c -o kernel.o -std=gnu99 -ffreestanding -O2 -Wall -Wextra
```
または
```
i686-elf-g++ -c kernel.c++ -o kernel.o -ffreestanding -O2 -Wall -Wextra -fno-exceptions -fno-rtti
```
* カーネルとブートローダのリンク時
```
i686-elf-gcc -T linker.ld -o myos.bin -ffreestanding -O2 -nostdlib boot.o kernel.o -lgcc
```

## ブートイメージの作成とmultibootの検証
* boot imageの作成
```
menuentry "myos" {
	multiboot /boot/myos.bin
}
```
* multibootの検証
boot.sファイルの先頭でmultibootヘッダを構成している。
この値が正しく設定されていれば、multiboot可能なイメージが作成できる。
検証にはgrubツールを使う。ここでは、verify_
s.shというshコマンドに下記を記述し、検証用ツールとした。
```
if grub-file --is-x86-multiboot myos.bin; then
  echo multiboot confirmed
else
  echo the file is not multiboot
fi
```

## QEMU ()
以下コマンドを実行し、boot可能ファイルを一まとめにした後、実行する
```
mkdir -p isodir/boot/grub
cp myos.bin isodir/boot/myos.bin
cp grub.cfg isodir/boot/grub/grub.cfg
grub-mkrescue -o myos.iso isodir

qemu-system-i386 -cdrom myos.iso
```

