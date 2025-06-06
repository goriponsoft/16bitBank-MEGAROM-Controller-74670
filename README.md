# 16bitセグメント8KBバンク・メガROMコントローラーの汎用ロジックICによる実装例
[English README is here](https://github.com/goriponsoft/16bitBank-MEGAROM-Controller-74670/blob/main/README-en.md)

![](schematic.png)

ASCII-8メガROMコントローラーをベースに、セグメントレジスタを16bitに拡張する提案の回路例(74x670仕様)です。
動作未確認ですが、同様の74x670(4×4レジスタファイル)を使用した、8bitセグメントのASCII-8互換メガROMコントローラーについては、[似非²RAM](https://github.com/goriponsoft/ESE2RAM-Cartridge-74670)として公開しており、そちらは動作確認済みです。

MITライセンスで公開されています。

X(旧Twitter): @goriponsoft

## 仕様
### 概要
ASCIIマッパーをベースに、セグメントレジスタを16bitに拡張することにより、セグメント数が256倍の最大65536セグメントとなり、8KBバンクの場合で最大4Gbit、16KBバンクで最大8Gbitのメモリを管理することができるようになります。

### NEO-8マッパーとの相違点
[NEO-8マッパー](https://aoineko.org/msxgl/index.php?title=NEO_mapper)と部分的に互換性があり、以下の点が異なります。
- ページ0のマッピングには対応しておらずページ0用のセグメントレジスタも存在しません。
- セグメントレジスタおよびバンクのミラーアドレスが異なります。
- セグメントレジスタの初期値が異なります。
- セグメントレジスタは16bitフル実装で上位4bitも利用可能です。最上位ビットはバックアップRAM選択ビットとして実装されています。

### 書き込みアクセス
従来のASCIIマッパーの、セグメントレジスタアドレス範囲の偶数アドレス(アドレスのbit0が0)に下位バイト、奇数アドレス(アドレスのbit0が1)に上位バイトが配置されています。この部分のアイデアについてはNEOマッパと完全に同一です。

この実装例(およびNEOマッパー)ではセグメントレジスタの下位バイトと上位バイトは分離されており、下位もしくは上位のみの書き換えが行えるため、奇数アドレスの上位バイトにアクセスさえしなければ、従来のASCIIマッパーと同一のコードが完全に動作します。

ASCIIマッパーと同様に、セグメントレジスタ範囲(6000h-7FFFh)への書き込みはバックアップRAMセグメント(セグメント最上位ビットが1)に対して透過しません。バンク6000h-7FFFhにバックアップRAMセグメントをマップしても書き込みは行えません。ROMセグメント(セグメント最上位ビットが0)への書き込み、またはバックアップRAM書き込み信号(/MEM_WR)を使用しない場合に、書き込みが透過するかどうかは、この実装例に含まれないデコード部の回路構成によりますので、フラッシュメモリなどを載せる場合にはご注意ください。

ページ0およびページ3にミラーが出現するかどうかは、書き込み信号(/WR)の回路構成によります。

### 読み出しアクセス
セグメントレジスタが16bitになっている以外は、従来のASCIIマッパーと同様にROMの内容がマッピングされています。セグメントレジスタは書き込み専用となっているため、セグメントレジスタ範囲であってもROMの内容が読み出されます。

ページ0およびページ3にミラーが出現するかどうかは、読み込み信号(/RD)の回路構成によります。例えば/RD信号の代わりに/CS12信号を使えば読み出しに対するミラーは出現しません。

### セグメントレジスタの初期値と初期化コード
セグメントレジスタを実現するために使用しているICである74HC670は、リセット機能を持たず、電源投入直後のレジスタの値は不定であるとされています。
しかし、MSXシステム起動時のセグメントが不定ではマッパーとして使うことができないため、対策として電源投入直後(およびリセット直後)は74HC670からのセグメント出力を停止し、プルダウンにより全てのセグメントレジスタの全てのビットを0として、全てのバンクにセグメント0がマップされるよう回路が組まれています。

MSXシステムによりROMヘッダの検索が行われ、ページ1のINITエントリが実行されるまでの間、4000h～5FFFhバンクを含めた全てのバンクにセグメント0がマップされるようにするのがこの機能の目的です(KONAMI4マッパーにおいて、4000h～5FFFhバンクがセグメント0に固定されているのに似ています)。

ただし、そのままではマッパーとして機能しないため、セグメントの固定は「セグメントレジスタへの最初の書き込み」をトリガとして停止(74HC670によるセグメント出力が再開)するようになっています。
その際、全てのセグメントレジスタが一斉に切り替わり、書き込みが行われたセグメントレジスタ以外のセグメントには、電源投入直後の不定値(リセットの場合はリセットするまでに書き込まれた値)が出力されるようになるため、プログラム側から見ると、4000h～BFFFhの全てのバンクのマッピングが変化したように見えることになります。

以上の事情により、このマッパーを使用する際には、INITエントリのできるだけ早い段階で、4000h～5FFFhバンクをセグメント0に、それ以外のバンクも適当なセグメントのマッピングを行うことが必要となります。

### マッパー
|バンク|セグメントレジスタアドレス|セグメント初期値|
|:--|:--|--:|
|4000h-5FFFh(ミラー:0000h-1FFFh[^2])|下位バイト6000h/上位バイト6001h(ミラー:6002h-67FFh)|不定[^1]|
|6000h-7FFFh(ミラー:2000h-3FFFh[^2])|下位バイト6800h/上位バイト6801h(ミラー:6802h-6FFFh)|不定[^1]|
|8000h-9FFFh(ミラー:C000h-DFFFh[^2])|下位バイト7000h/上位バイト7001h(ミラー:7002h-77FFh)|不定[^1]|
|A000h-BFFFh(ミラー:E000h-FFFFh[^2])|下位バイト7800h/上位バイト7801h(ミラー:7802h-7FFFh)|不定[^1]|

[^1]: 最初にセグメントレジスタに書き込みが行われるまでは0に固定されており、書き込みが行われたセグメントレジスタ以外は一斉に不定値に変わります。
[^2]: メモリのREAD信号の実装に依存します。