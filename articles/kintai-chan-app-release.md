---
title: "SwiftUIで実装したApple Watch専用アプリをリリースしました"
emoji: "⌚️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Swift", "SwiftUI", "AppleWatch", "watchOS"]
published: false
---

## はじめに

タイトル通り Apple Watch 専用アプリをリリースしたので
実装や App Store Connect 周りで思ったことを書いていきます。


## 作ったアプリ

勤怠ちゃんという勤怠に関する定型ワードをSlackに投稿するアプリです。

![altテキスト](https://user-images.githubusercontent.com/8732417/127726905-7c6fa973-8f36-4694-96f1-498110dfc35a.png =250x)
[![altテキスト](https://tools.applemediaservices.com/api/badges/download-on-the-app-store/black/ja-jp?size=250x83&amp;releaseDate=1627603200&h=67f860c8a4424c97a47065fb78d09e10)](https://apps.apple.com/jp/app/%E5%8B%A4%E6%80%A0%E3%81%A1%E3%82%83%E3%82%93/id1577214020?itsct=apps_box_badge&amp;itscg=30200)

PCやスマホで Slack アプリを開いて打ち込むの面倒だな〜
Apple Watch で数回画面をタップするだけで代わりにできたら楽だな〜
ということで 日報ちゃんというアプリ名で
2018年の Qiita Advent Calendar で実装してみたのが最初で，
2019年の夏の合宿で SwiftUI で作ってみようと取りかかり，
2019年の Qiita Advent Calendar で実装して記事描きました。

![KintaichanAppMovie](https://user-images.githubusercontent.com/8732417/126958044-b1de1545-d1b4-4516-a3c4-e83ac38f6ce2.gif)

watchOS 6 から iOS アプリは不要となり，
アプリを Apple Watch 専用に作れるようになったのが大きく，
よりシンプルなアプリが作れるようになりました。
機能的にも勤怠ちゃんはシンプルなので単体アプリはよく合っています。

今回は，watchOS 7 以降をサポート，
つまり SwiftUI 2 で作り直した感じです。
SwiftUI 初版はまだまだ融通が効かないことも
多かったので気持ちよく実装はできました。


良かったら下記の記事たちもご覧ください。

https://qiita.com/MilanistaDev/items/b97cab77d6add96c96dc

https://qiita.com/MilanistaDev/items/dbc4d2c2cc4522f12972

https://qiita.com/MilanistaDev/items/a2325ce916f625aa1d44


## Apple Watch 専用アプリを実装する上で困ったこと

### 画面の狭さ

40mm と 44mm の細かい差が辛いところもあって，
基本的に `ScrollView` を使う必要があって，
View のコードはネストが深くなりがちでした。
ファイル分けたらそこまで気にならなくはなりました。

ウォークスルーなどで長い説明文を書いても画面が狭く，
スクロール必須になるのでよりシンプルな文言選びをしました。
同様に画面のスクショを使いながら説明する形はできなかった。
iPhone 側で補足することもできないのは辛いなと思いました。

### App Store Connect が分かりにくい

App Store Connect にログインして，
My App から新規アプリを作るのですが，
Watch App は選択肢にない。。。











