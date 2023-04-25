---
title: "複数ターゲットのプロジェクトに R.swift 7 系を Swift Package Manager で導入"
emoji: "📱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Swift", "Rswift", "iOS", "watchOS"]
published: false
---

# はじめに

個人開発でも業務でも iOS や watchOS のアプリ開発で
リソースファイルを扱いやすくするためによく R.swift を使っています。
詳しくは GitHub や良記事が多々あるので見てください。

R.swift 7 系から Swift Package Manager での導入が推奨になっています。
下記記事に書いたように色々ありつつも案件や個人開発においても
最新バージョンへの移行を終えていたのですが，

https://qiita.com/MilanistaDev/items/8fdc0fc9327dce3d697a

今回新しく，iOS アプリに Watch アプリも追加しようということで
Watch App 用のターゲットを追加したのですが
あれ？どうやってもう一つのターゲットでも使えるようにするんだっけ？
となったので備忘録として書きます。
