---
date: "2018-09-14"
title: "自作ブートローダ"
category: "Software Development"
---

「[__Operating Systems: From 0 to 1__](https://github.com/tuhdo/os01)」というオペレーティングシステムの具体的な開発についての本を読んで、ブートローダを開発してみました。

使ったシステム：
- Windows 10にVM Ware Workstation 12 PlayerでUbuntu 18.04の仮想マシーン
- Ubuntuの仮想マシーンでまたQEMUを使ってIntel 8086（詳しくは[__ここ__](https://qiita.com/timwata/items/e7b7a18cc80b31fd940a)へ）をエミュレート

## x86のブート・シーケンス
x86アーキテクチャのコンピューターは以下のように起動します([1](https://www.diag.uniroma1.it/~pellegrini/didattica/2017/aosv/1.Initial-Boot-Sequence.pdf))：
1. BIOS（その版によってPOSTもします）


- BIOSは起動可能のデバイス（フロッピーディスクやハードディスクドライブなど）を探し、見つかった場合、その第一セクターをRAMの0x7C00にコピーします
- どうやって見つかるというと、デバイスの第一セクターの、最後にある２バイトに0x55, 0xAAが代入されているかとを確かめます
- 0xAA55（リトルエンディアン）＝ブートシグニチャ


2. 前デバイスからコピーされたブートローダを実行
3. ブートローダは結局OSを実行

## ブートローダへ

以下にコメント付きの自作ブートローダを貼り付けました。BIOSの[__int 10h__](http://softwaretechnique.jp/OS_Development/Tips/Bios_Services/video_services.html)インタラプトで、いろいろなファンクション（文字列を書き込むなど）が使えます。


	;********************
	;A simple bootloader
	;*******************
	org 7c00h 		;予期している、ブートローダを格納するメモリーアドレス
	bits 16　		;16-bitモードの指定

	start: 
	  mov ax, cs	;Code Segmentアドレス --> AX
	  mov ds, ax	;AX --> Data Segment （つまり Data Segment ＝ Code Segment)
	  mov es, dx	;DXをExtra Segmentとして使う
	  call ClrScrn	;サブルーチンを呼び出し
	  call Print	;サブラチーンを呼び出し
	  jmp boot		;bootラベルにジャンプ

	ClrScrn:		;画面消去(Clear Screen)
	  mov al, 03h	;テキストモード（カラー）、80x25 を設定（実はモードを設定するだけで、画面消去もされる。画面消去だけのファンクションは特にない）
	  mov ah, 00h	;ビデオモード設定のファンクションを指定
	  int 10h		;ビデオサービスを割り込み
	  xor ax, ax	;AXの値をクリア
	  ret　			;戻る
	  
	Print:			;msgを表示するサブルーチン
	  push bp		;Base Pointerを保存
	  mov bp, msg	;msgのアドレスをbpにコピー
	  mov bh, 0h	;ヴィデオ・ページ・番号を指定
	  mov bl, 0fh	;
	  mov cx, 0ch	;文字列の長さ
	  mov al, 0h	;文字列書き込みモード 
	  mov ah, 13h	;文字列書き込みファンクションを指定
	  mov dx, 0h	;座標を指定（0,0は左上から）
	  int 10h		;;ビデオサービスを割り込み
	  pop bp		;まえ保存したbpを戻す
	  ret			;戻る

	boot:
	  cli ;インタラプト無しに
	  cld ;Direction Flag （方向のフラグ）をクリア
	  hlt ;CPUを停止
	  
	msg db "Hello World!"　;表示したい文字列

	times 510 - ($-$$) db 0
	dw 0xAA55

-


## そろそろ実行しよう
Ubuntuでターミナルをあけて、続いての命令を入力して、上のコードをコンパイルします。

	nasm -f bin bootloader.asm -o bootloader
	
ブートローダを格納するデバイスも必要なので、フロッピーディスクを作っていきましょう。

	dd if=/dev/zero of=disk.img bs=512 count=2880
	
作ったフロッピーディスクの第一セクターにコンパイルされたbootloaderを書き込みます。

	dd conv=notrunc if=bootloader of=disk.img bs=512 count=1 seek=0
	
最後にQEMUでブートローダを実行します。

	qemu-system-i386 -machine q35 -drive format=raw,file=disk.img,index=0,if=floppy 
	
全部エラー無くいけたら、下のような画面がみられるでしょう。

![](https://i.imgur.com/6xE3JSX.png)

