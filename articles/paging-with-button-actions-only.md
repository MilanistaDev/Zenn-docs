---
title: "[SwiftUI] ボタンアクションのみでページングさせる実装を考える"
emoji: "📱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iOS", "SwiftUI", "Paging"]
published: false
---

## はじめに

ある案件で，縦に画像が並ぶコンテンツ画面を拡張して
一時的に特別なコンテンツも表示したいという話をいただき，
ページングさせる形式で
基本コンテンツ↔️特別コンテンツの切り替えを行うことになった。

いつも通り `PageTabViewStyle` 使えば良いよな，と思って
仕様詰めをしていたのだが，
**「ボタンによるコンテンツ切り替えのみにしたい」** 
という追加のご希望があった。

`TabView` 使う想定のままなんとかいけそうか，
できないなら代替案を考えようということで調査に取り掛かった。

## 仕様とサンプルアプリ

今回のサンプルアプリの GitHub は下記です。

https://github.com/MilanistaDev/PagingWithButtonActionsOnly

* iOS 16 以上
* SwiftUI
* MVVM

動作イメージは下記の GIF のような感じ。



### 仕様

* ページ切り替えボタンを画面の上部の左右端に表示する
* ページ切り替えボタンはページ切り替えができない場合非表示(最初と最後のpage)
* ページングは左右に動くアニメーションありで実現する
* ドラッグやスワイプによるページングはできないようにする(ページ切り替えボタンのみ)
* コンテンツは適当な色を着色したビューを縦に並べ，スクロールできるものとする

### コンテンツデータ

API などで非同期での取得を想定するが，
今回は下記の構造体を使ったサンプルデータを使う。

```swift:Contents.swift
struct Contents {
    /// ページ名(ボタンに表示されるテキスト)
    let pageName: String
    /// ボタンに表示されるテキストカラー
    let textColor: String
    /// ボタンの背景色
    let backColor: String
    /// 縦に並ぶコンテンツをページ数分格納した配列
    let columns: [Color]
}

let sampleContents: [Contents] = [
    Contents(
        pageName: "PAGE1",
        textColor: "#000000",
        backColor: "#f39700",
        columns: [.cyan, .mint, .teal]
    ),
    Contents(
        pageName: "PAGE2",
        textColor: "#FFFFFF",
        backColor: "#E60012",
        columns: [.red, .purple, .yellow]
    ),
    Contents(
        pageName: "PAGE3",
        textColor: "#FFFFFF",
        backColor: "#9b7cb6",
        columns: [.pink, .orange, .blue]
    )
]
```

## 実装

## おわりに

## 参考

