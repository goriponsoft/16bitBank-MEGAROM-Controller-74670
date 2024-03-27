# 16bitバンク・メガROMコントローラーの汎用ロジックICによる実装例
[English README is here](https://github.com/goriponsoft/16bitBank-MEGAROM-Controller-74670/blob/main/README-en.md)

ASCIIメガROMコントローラーをベースに、バンク番号レジスタを16bitに拡張する提案の回路例(74x670仕様)です。
動作未確認ですが、同様の74x670(4×4レジスタファイル)を使用した8bitバンクメガROMについては、[似非²RAM](https://github.com/goriponsoft/ESE2RAM-Cartridge-74670)として公開しており、そちらは動作確認済みです。

[NEO-8マッパー](https://aoineko.org/msxgl/index.php?title=NEO_mapper)と部分的に互換性があり、以下の点が異なります。
- ページ0のマッピングには対応していません。
- バンクレジスタおよびバンクのミラーアドレスが異なります。

MITライセンスで公開されています。

X(旧Twitter): @goriponsoft
