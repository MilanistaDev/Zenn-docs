---
title: "住みたい街ランキング2021年版が発表されたのでSwiftUIでトレースしてみた!!"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iOS", "SwiftUI", "Swift"]
published: false
---

## はじめに

今回が Zenn 初記事になります。
Qiita で記事を書いていましたが気分転換に
やってみた系はこちらに投稿していこうと思います。

アプリにしやすそうなネタを題材にして
こういう実装どうやるんだろう？とかあの機能使ってみたいなー
的な内容を実際に実装してみて書き残していこうと思います。
基本 SwiftUI でと考えていて UIKit と比較する形も取れればと思います。
まだ個人開発レベルでは実現できるかに重きをおこうと思ってるので
設計や書き方はあまり綺麗にとは考えておりません。
こうした方が良さそうなどは指摘いただくと嬉しいです。

## 今回実装するもの

LIFULL HOME’S が毎年算出している住みたい街ランキングを題材にします。
先日，2021 年版が出たのでそのデータを使ってみます。
ランキングデータは私がサイトをみて JSON 方式にしました。

借りて住みたい，買って住みたいランキングがそれぞれあり，
今回は首都圏のランキング各20位までを表示します。
またサイトにあるように 11位以降は出し分けできるようにします。
左右にスワイプ，またはタブ部分をタップすることで
借って住みたいと買って住みたいを切り替えられるようにします。

私はサクッと Google App Script で JSON を返すように外部 API を作りましたが
GitHub のサンプルはローカルの JSON を読み込むようにしています。

完成したものはこちらです。

GIF

## 開発環境

開発環境は下記です。

* iOS 14.0 and later
* Xcode 12.4
* macOS Big Sur 11.2

GitHub にサンプルアプリを用意したので必要に応じて参照してみてください。
https://github.com/MilanistaDev/TownRanking2021

## 今回抑えたい内容

* `HStack`，`VStack`，`ZStack` などの基本的な内容の復習
* `@State`，`@Binding` などの値の扱い
* `@ObservedObject`，`@Observable`，`@Published` を利用したデータの扱い
* iOS 14 で追加された TabView の Page
* 簡単な Animation

## 実装

私は SwiftUI でアプリ作るときは親クラスの View に実装して
完成後に別ファイルに View の実装を切り出すようにしています。
`@Binding` 使うのか，ただの値渡しで良いのかなどの考慮を後で考えたいからです。
実際の開発では設計が事前にできることが多いので，
業務ではいきなり別クラスで直接書くことは多そうです。

今回は UI の実現が主な内容なので，部分的に切り出した形で書いていきます。

* 親 View クラスの画面
* 上部の状態切り替えタブ部分
* 街ランキングリスト部分(ページング)
* 街ランキングリスト部分(リスト表示)
* 街ランキングリスト部分(11位以降の表示/非表示対応)
* ランキングリスト用の街データモデルの実装
* JSON のデータをリストに表示

### 上部の状態切り替えタブ部分の実装

今回の親 View は新規でプロジェクト作った際にデフォルトで用意されている，
`ContentView.swift` とします。各部分を実装していくたびに少しずつ変わっていきます。

上部の状態切り替えタブの実装ですが，
借りると買うの状態は，選択時にボタンのフォントカラーをつけるのと，
ボタンの下にバーを用意させて選択状態で追従させます。

![](https://storage.googleapis.com/zenn-user-upload/recxsh7pk20oxg6ux5gkojh0tpn9)

![](https://storage.googleapis.com/zenn-user-upload/hca13alxbao2jfffimithodaedo6)

`HStack` と `VStack` を組み合わせて作ります。
借りると買うの状態は `Bool` で管理しますが，
状態の変化によってボタンとバー，ランキングリストを追従させるために `@State` をつけます。

少し考えなければならないのが，バーの Frame です。
`GeometryReader` を使って画面幅の半分を算出します。
(`UIScreen.main.bounds.size` でも今回は大丈夫です。)
座標については `offset` を使って，借りる，買うの状態によってずらすことにしました。

```swift:ContentView.swift
struct ContentView: View {
    @State private var isRent = true

    var body: some View {
        GeometryReader { geometry in
            NavigationView {
                VStack {
                    UpperTabView(isRent: $isRent, geometrySize: geometry.size)
                    Spacer()
                }
                .navigationBarTitle("住みたい街ランキング2021(首都圏)",
                                    displayMode: .inline)
            }
        }
    }
}
```

`geometrySize` を `CGSize` としてを値渡ししています。
ボタンとバーの幅，バーの座標設定に利用しています。
3項演算子を使って，借りると買うの各状態の色の設定，バーの座標計算なども行っています。
このタブでのボタン操作によって変わる状態を親 View に伝える必要が
今後出てくるため，ここで `@Binding` をつけておきます。

```swift:UpperTabView.swift
struct UpperTabView: View {
    // ボタンのタップで状態を変える 親View状態を伝えるため @Binding をつけておく
    @Binding var isRent: Bool
    let geometrySize: CGSize

    var body: some View {
        VStack(alignment: .leading, spacing: .zero) {
            HStack(spacing: .zero) {
                Button(action: {
                    // 借りて住みたいボタンタップで借りる状態に変更
                    self.isRent = true
                }, label: {
                    Text("借りて住みたい")
                        .foregroundColor(self.isRent ?
                                            .rentOrange: .gray)
                         .font(.headline)
                })
                .frame(width: geometrySize.width / 2, height: 44.0)
                Button(action: {
                    // 買って住みたいボタンタップで買う状態に変更
                    self.isRent = false
                }, label: {
                    Text("買って住みたい")
                        .foregroundColor(self.isRent ?
                                            .gray: .buyBlue)
                        .font(.headline)
                })
                .frame(width: geometrySize.width / 2, height: 44.0)
            }
            Rectangle()
                .fill(self.isRent ? Color.rentOrange: Color.buyBlue)
                .frame(width: geometrySize.width / 2, height: 2.0)
                .offset(x: self.isRent ? .zero: geometrySize.width / 2, y: .zero)
        }
    }
}
```

ここまでの実装で実行するとそれぞれのボタンタップで，
借りる，買うの状態が変わってボタンの色やバーも追従してくれます。

バーをアニメーションさせたいなと思って調べて 2種類方法見つけたのですが，
ここではひとつ目のものを採用しました。見つけたときちょっと感動しました。
`withAnimation`[^1] を使うことでアニメーションさせながら
`Bool` 値を切り替えられるというものです。下記のような感じです。
0.3 秒かけて等速でアニメーションさせています。

```diff swift
Button(action: {
-    self.isRent = true
+    withAnimation(.linear(duration: 0.3)) {
+        self.isRent = true
+    }
}, label: {
    // 略
})

Button(action: {
-    self.isRent = false
+    withAnimation(.linear(duration: 0.3)) {
+        self.isRent = false
+    }
}, label: {
    // 略
})
```
これでバーが状態を切り替えると遅れて追従してくれるようになりました🎉
最終的には別のアニメーション実装を選択することになりましたがそれは後述します。

GIF

:::message
* 状態切り替えの扱い方
* バーの座標切り替えは `offset` でずらせば実現できる！
* `withAnimation` 便利！
:::

### 街ランキングリスト部分(リスト表示)



## 実行結果

## Future Work

## おわりに

## 参考

[^1]:https://developer.apple.com/documentation/swiftui/withanimation(_:_:)
