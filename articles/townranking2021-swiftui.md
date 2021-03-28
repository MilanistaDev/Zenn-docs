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

### 参考にしたサイト

今回のアプリの題材として，LIFULL HOME'S の住みたい街ランキングを参考にしました。
コロナ禍なこともあるのか大きく順位を上げた街も多く興味深かったです。
首都圏だけ参照しましたが，他の地域も加えればもっと面白くなりそうです。

https://www.homes.co.jp/cont/s_ranking/shutoken/

使うデータの JSON ファイルはこちらです。
`isRankUp` はランクアップしたかどうか，`rankFluctuation` は変化量です。
(今思うとランク変化量だけでランクアップ/ダウン/キープ判断したほうがよかった🤔)

:::details 使うデータの JSON ファイル
```json:TownRanking2021.json
{
    "townRankingsForRent": [
        {
            "rank": 1,
            "townName": "本厚木",
            "isRankUp": true,
            "rankFluctuation": 3,
            "availableLine": "小田急小田原線"
        },
        {
            "rank": 2,
            "townName": "大宮",
            "isRankUp": true,
            "rankFluctuation": 3,
            "availableLine": "JR京浜東北・根岸線ほか"
        },
        {
            "rank": 3,
            "townName": "葛西",
            "isRankUp": false,
            "rankFluctuation": 1,
            "availableLine": "東京メトロ東西線"
        },
        {
            "rank": 4,
            "townName": "八王子",
            "isRankUp": true,
            "rankFluctuation": 3,
            "availableLine": "JR中央線ほか"
        },
        {
            "rank": 5,
            "townName": "池袋",
            "isRankUp": false,
            "rankFluctuation": 4,
            "availableLine": "JR山手線ほか"
        },
        {
            "rank": 6,
            "townName": "千葉",
            "isRankUp": true,
            "rankFluctuation": 8,
            "availableLine": "JR総武線ほか"
        },
        {
            "rank": 7,
            "townName": "蕨",
            "isRankUp": true,
            "rankFluctuation": 4,
            "availableLine": "JR京浜東北・根岸線"
        },
        {
            "rank": 8,
            "townName": "三鷹",
            "isRankUp": true,
            "rankFluctuation": 9,
            "availableLine": "JR中央線ほか"
        },
        {
            "rank": 9,
            "townName": "柏",
            "isRankUp": true,
            "rankFluctuation": 7,
            "availableLine": "JR常磐線ほか"
        },
        {
            "rank": 10,
            "townName": "川崎",
            "isRankUp": false,
            "rankFluctuation": 7,
            "availableLine": "JR東海道本線ほか"
        },
        {
            "rank": 11,
            "townName": "高円寺",
            "isRankUp": true,
            "rankFluctuation": 2,
            "availableLine": "JR中央線ほか"
        },
        {
            "rank": 12,
            "townName": "西川口",
            "isRankUp": true,
            "rankFluctuation": 8,
            "availableLine": "JR京浜東北・根岸線"
        },
        {
            "rank": 13,
            "townName": "船橋",
            "isRankUp": true,
            "rankFluctuation": 8,
            "availableLine": "JR総武線ほか"
        },
        {
            "rank": 14,
            "townName": "町田",
            "isRankUp": true,
            "rankFluctuation": 8,
            "availableLine": "小田急小田原線ほか"
        },
        {
            "rank": 15,
            "townName": "荻窪",
            "isRankUp": false,
            "rankFluctuation": 7,
            "availableLine": "JR中央線ほか"
        },
        {
            "rank": 16,
            "townName": "三軒茶屋",
            "isRankUp": false,
            "rankFluctuation": 10,
            "availableLine": "東急田園都市線ほか"
        },
        {
            "rank": 17,
            "townName": "川口",
            "isRankUp": true,
            "rankFluctuation": 15,
            "availableLine": "JR中央線ほか"
        },
        {
            "rank": 18,
            "townName": "吉祥寺",
            "isRankUp": false,
            "rankFluctuation": 9,
            "availableLine": "JR中央線ほか"
        },
        {
            "rank": 19,
            "townName": "新小岩",
            "isRankUp": false,
            "rankFluctuation": 1,
            "availableLine": "JR総武線ほか"
        },
        {
            "rank": 20,
            "townName": "小岩",
            "isRankUp": false,
            "rankFluctuation": 0,
            "availableLine": "JR総武線"
        }
    ],
    "townRankingsForBuy": [
        {
            "rank": 1,
            "townName": "勝どき",
            "isRankUp": false,
            "rankFluctuation": 0,
            "availableLine": "都営大江戸線"
        },
        {
            "rank": 2,
            "townName": "白金高輪",
            "isRankUp": true,
            "rankFluctuation": 17,
            "availableLine": "東京メトロ南北線ほか"
        },
        {
            "rank": 3,
            "townName": "本厚木",
            "isRankUp": true,
            "rankFluctuation": 8,
            "availableLine": "小田急小田原線"
        },
        {
            "rank": 4,
            "townName": "三鷹",
            "isRankUp": false,
            "rankFluctuation": 1,
            "availableLine": "JR中央線ほか"
        },
        {
            "rank": 5,
            "townName": "北浦和",
            "isRankUp": false,
            "rankFluctuation": 1,
            "availableLine": "JR京浜東北・根岸線"
        },
        {
            "rank": 6,
            "townName": "八王子",
            "isRankUp": false,
            "rankFluctuation": 0,
            "availableLine": "JR中央線ほか"
        },
        {
            "rank": 7,
            "townName": "柏",
            "isRankUp": true,
            "rankFluctuation": 8,
            "availableLine": "JR常磐線ほか"
        },
        {
            "rank": 8,
            "townName": "目黒",
            "isRankUp": true,
            "rankFluctuation": 13,
            "availableLine": "JR山手線ほか"
        },
        {
            "rank": 9,
            "townName": "恵比寿",
            "isRankUp": false,
            "rankFluctuation": 7,
            "availableLine": "JR山手線ほか"
        },
        {
            "rank": 10,
            "townName": "東京",
            "isRankUp": false,
            "rankFluctuation": 5,
            "availableLine": "JR東海道本線ほか"
        },
        {
            "rank": 11,
            "townName": "橋本",
            "isRankUp": true,
            "rankFluctuation": 31,
            "availableLine": "JR横浜線ほか"
        },
        {
            "rank": 12,
            "townName": "浦和",
            "isRankUp": false,
            "rankFluctuation": 5,
            "availableLine": "JR京浜東北・根岸線ほか"
        },
        {
            "rank": 13,
            "townName": "千葉",
            "isRankUp": true,
            "rankFluctuation": 49,
            "availableLine": "JR総武線ほか"
        },
        {
            "rank": 14,
            "townName": "平塚",
            "isRankUp": true,
            "rankFluctuation": 25,
            "availableLine": "JR東海道本線ほか"
        },
        {
            "rank": 15,
            "townName": "大宮",
            "isRankUp": true,
            "rankFluctuation": 9,
            "availableLine": "JR京浜東北・根岸線ほか"
        },
        {
            "rank": 16,
            "townName": "朝霞",
            "isRankUp": true,
            "rankFluctuation": 13,
            "availableLine": "東武東上線"
        },
        {
            "rank": 17,
            "townName": "八街",
            "isRankUp": true,
            "rankFluctuation": 9,
            "availableLine": "JR総武本線"
        },
        {
            "rank": 18,
            "townName": "川口",
            "isRankUp": true,
            "rankFluctuation": 9,
            "availableLine": "JR京浜東北・根岸線"
        },
        {
            "rank": 19,
            "townName": "牛込柳町",
            "isRankUp": true,
            "rankFluctuation": 191,
            "availableLine": "都営大江戸線"
        },
        {
            "rank": 20,
            "townName": "中目黒",
            "isRankUp": false,
            "rankFluctuation": 7,
            "availableLine": "東京メトロ日比谷線ほか"
        }
    ]
}
```
:::

## 今回抑えたい内容

* `HStack`，`VStack`，`ZStack` などの基本的な内容の復習
* `@State`，`@Binding` などの値の扱いの復習
* `@ObservedObject`，`@Observable`，`@Published` を利用したデータの扱いの復習
* iOS 14 で追加された `TabView` の `PageTabViewStyle` を使ってみる
* iOS 14 で追加された `LazyVGrid` を使ってみる
* 簡単な Animation の復習
  - 状態の変化に対して (`withAnimation`)
  - ビュー内のアニメーション可能な変更に対して(`animation`)

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
借りると買うの状態は `enum` で管理しますが，
状態の変化によってボタンとバー，ランキングリストを追従させるために `@State` をつけます。

少し考えなければならないのが，バーの Frame です。
`GeometryReader` を使って画面幅の半分を算出します。
(`UIScreen.main.bounds.size` でも今回は大丈夫です。)
座標については `offset` を使って，借りる，買うの状態によってずらすことにしました。

```swift:ContentView.swift
enum TabType: Int {
    case rent
    case buy
}

struct ContentView: View {
    @State private var selection: TabType = .rent

    var body: some View {
        GeometryReader { geometry in
            NavigationView {
                VStack(spacing: .zero) {
                    UpperTabView(selection: $selection, geometrySize: geometry.size)
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
    @Binding var selection: TabType
    let geometrySize: CGSize

    var body: some View {
        VStack(alignment: .leading, spacing: .zero) {
            HStack(spacing: .zero) {
                Button(action: {
                    // 借りて住みたいボタンタップで借りる状態に変更
                    self.selection = .rent
                }, label: {
                    Text("借りて住みたい")
                        .foregroundColor(self.selection == .rent ?
                                            .rentOrange: .gray)
                         .font(.headline)
                })
                .frame(width: geometrySize.width / 2, height: 44.0)
                Button(action: {
                    // 買って住みたいボタンタップで買う状態に変更
                    self.selection = .buy
                }, label: {
                    Text("買って住みたい")
                        .foregroundColor(self.selection == .rent ?
                                            .gray: .buyBlue)
                        .font(.headline)
                })
                .frame(width: geometrySize.width / 2, height: 44.0)
            }
            Rectangle()
                .fill(self.selection == .rent ? Color.rentOrange: Color.buyBlue)
                .frame(width: geometrySize.width / 2, height: 2.0)
                .offset(x: self.selection == .rent ? .zero: geometrySize.width / 2, y: .zero)
        }
    }
}
```

ここまでの実装で実行するとそれぞれのボタンタップで，
借りる，買うの状態が変わってボタンの色やバーも追従してくれます。

バーをアニメーションさせたいなと思って調べて 2種類方法見つけたのですが，
ここではひとつ目のものを採用しました。見つけたときちょっと感動しました。
`withAnimation`[^1] を使うことでアニメーションさせながら
`Bool` 値を切り替えられたりするんですね。色々便利そう。

今回は，`enum` の状態を0.3 秒かけて変更させました。
こだわりはなかったので `linear`(等速)でアニメーションさせています。


```diff swift
Button(action: {
-    self.isRent = true
+    withAnimation(.linear(duration: 0.3)) {
+        self.selection = .rent
+    }
}, label: {
    // 略
})

Button(action: {
-    self.isRent = false
+    withAnimation(.linear(duration: 0.3)) {
+        self.selection = .buy
+    }
}, label: {
    // 略
})
```
これで状態を切り替えるとバーが追従してくれるようになりました🎉
この実装をしたときは感動がありましたが，最終的には・・・
別のアニメーション実装を選択することになりましたがそれは後述します。

GIF

:::message
* `@State`，`@Binding` を利用した状態切り替えの扱い方
* バーの座標切り替えは `offset` でずらせば実現できる！
* `withAnimation` で任意の時間をかけて状態を変化させることができて便利！
:::

### 街ランキングリスト部分(ページング)

借りて住みたい，買って住みたいランキング部分は左右にスワイプ(ページング)させてみます。
そこで使ってみたかったのが iOS 14 から使えるようになった `TabView` の新要素です。

`TabView` は SwiftUI 初期からありました。
用途は文字通り UIKit での `UITabBarController` 的な使い方です。
iOS 14 から `PageTabViewStyle` が追加されました。
`UIPageViewController` 的なページングが可能な UI が実現できます。

使い方は，`tabViewStyle` の設定を `PageTabViewStyle` にするだけです。
`TabView` の中で表示させたい `View` を書いていきます。
そうするとページングするとそれぞれの `View` が表示されます。

`indexDisplayMode` は `automatic`，`always`，`never` の3種類があります。
これは，UIKit での `UIPageControl`，つまり現在の index を示す UI です。
今回は不要なので `never` を使います。
下記のようにとても楽に書けます😂

```swift
TabView {
    Text("Page A")
    Text("Page B")
}
.tabViewStyle(PageTabViewStyle(indexDisplayMode: .never))
```

今回はこれに加えて `selection` の概念を使います。
ページングによって表示されている `View` の index を知ることができます。
そのため，`View` にはそれぞれ `tag` を付与します。
`tag` は `Hashable` に準拠していれば大丈夫で今回は `enum` の状態(`Int`型)を使います。

`ContentPageView.swift` を新規で作ってこれをランキング部分にします。
借って住みたい，買って住みたいの各状態にそれぞれの `View` があって
その `View` を左右にスワイプしてページングすることで各状態に対応した `View` を表示します。
また，その際上部のタブの状態も切り替えます。
ここで `selection` の連携が活きてきます。

```swift:ContentPageView.swift
struct ContentPageView: View {
    // ページングすることで状態が変わるので @Binding を使う
    @Binding var selection: TabType

    var body: some View {
        TabView(selection: $selection) {
            Text("Rent")
                .tag(TabType.rent)
            Text("Buy")
                .tag(TabType.buy)
        }
        .tabViewStyle(PageTabViewStyle(indexDisplayMode: .never))
    }
}
```
:::details この時点の ContentView.swift
```swift:ContentView.swift
enum TabType: Int {
    case rent
    case buy
}

struct ContentView: View {
    @State private var selection: TabType = .rent

    var body: some View {
        GeometryReader { geometry in
            NavigationView {
                VStack(spacing: .zero) {
                    UpperTabView(selection: $selection, geometrySize: geometry.size)
                    // ここ追加 ↓
                    ContentPageView(selection: $selection)
                }
                .navigationBarTitle("住みたい街ランキング2021(首都圏)",
                                    displayMode: .inline)
            }
        }
    }
}
```
:::

左右にスワイプしてページングすることで状態が変わるので `@Binding` を使って
親 View(`ContentView`) に状態の変化を伝えられるようにしています。
ここでのページングによる状態変化は，親 View 経由でタブ(`UpperTabView`)にも伝えられ，よってタブの状態切り替えも行われます。

ここまでの実装の実行結果はこちらです。

GIF

仕様的には問題ないですが，ちょっと惜しい部分があります。
上部のタブのボタンをタップして状態を切り替える際は，
ページングもバーも追従してくれますが，
左右にスワイプしてページングした場合は上部のタブのバーがアニメーションしません。

理由は，ボタンタップ時にのみアニメーションするように書いているためです。
よって状態の変化があるたびにアニメーションさせるために実装を見直します。
`animation(_:)`[^2] モディファイアを使います。
ラップするビュー内のすべてのアニメーション化可能な変更に適用されます。

`Button` の action で状態を切り替えていたのをやめて，
バーの View，すなはち `Rectangle` にラップさせます。
ここでは，状態変化に伴うバーの座標変更にアニメーションが適用されます。

```diff swift:ContentPageView.swift
struct UpperTabView: View {

    @Binding var selection: TabType
    let geometrySize: CGSize

    var body: some View {
        VStack(alignment: .leading, spacing: .zero) {
            HStack(spacing: .zero) {
                // 省略
            }
            Rectangle()
                .fill(self.selection == .rent ? Color.rentOrange: Color.buyBlue)
                .frame(width: geometrySize.width / 2, height: 2.0)
                .offset(x: self.selection == .rent ? .zero: geometrySize.width / 2, y: .zero)
+               .animation(.linear(duration: 0.3))
        }
    }
}
```

また，`ContentPageView` の `TabView` も同様にラップしておきます。
上部タブのタップで状態を切り替えた際に，ページングのアニメーションさせるためです。

```diff swift:ContentPageView.swift
struct ContentPageView: View {

    @Binding var selection: TabType

    var body: some View {
        TabView(selection: $selection) {
            Text("Rent")
                .tag(TabType.rent)
            Text("Buy")
                .tag(TabType.buy)
        }
        .tabViewStyle(PageTabViewStyle(indexDisplayMode: .never))
+       .animation(.linear(duration: 0.3))
    }
}
```

これで，借りて住みたい，買って住みたいボタンタップ時，
左右にスワイプしてページングした際にも期待した動きになりました。

GIF


:::message
* `TabView` の新しい `PageTabViewStyle` の使い方を抑える
* `animation` はラップした View のアニメーション化可能な変更に対してのアニメーション
:::

### 街ランキングリスト部分(リスト表示)

#### 街ランキングリスト部分(LazyVGrid部分)

次に借りて住みたい，買って住みたい，
それぞれのランキング表示用のリストを作ります。

`List` で `TableView` みたいな表示も可能だし，
`ForEach` を使って View を繰り返し表示することでも実現可能です。
せっかくなので iOS 14 から新しく使えるようになった `LazyVGrid` を使って
カードUI のリストみたいにしてみます。

`LazyVGrid`[^3] は UIKit での `UICollectionView` にあたります。
SwiftUI 初期に欲しかったですよね。`LazyHGrid` もあります。
Grid を組んで縦にスクロールさせる場合が `LazyVGrid`，
Grid を組んで横にスクロールさせる場合が `LazyHGrid` のイメージです。

> The grid is “lazy,” in that the grid view does not create items until they are needed.

公式ドキュメントにある通り，表示に必要になるまで Grid View(セル)を生成しないです。
要素数が多い場合でもパフォーマンスよくなる感じです。
新しく登場した `LazyVStack`，`LazyHStack` も同じ考え方です。

`LazyVGrid` では横のセル同士のマージンの調整は，`GridItem` で，
次の列のセルとのマージンは `LazyVGrid` の方で設定します。
`LazyHGrid` は逆になります。
`GridItem` ではセルの幅や行ごとのセルの数など設定できますが，今回は最小の実装にします。
次回あたりテーマにしてみようと思います。
以下，Grid View は，セルとして表現します。

`TownRankingListView.swift` を新たに生成し，
こちらにランキングリストを実装していきます。
この View がページングされるように書き換えます。
ランキングリスト自体は同じクラスで作るので状態を引数として渡します。
ついでに SafeArea も無視して一番下まで表示できるようにしておきます。

```diff swift:ContentPageView.swift
struct ContentPageView: View {

    @Binding var selection: TabType

    var body: some View {
        TabView(selection: $selection) {
-           Text("Rent")
+           TownRankingListView(selection: selection)
                .tag(TabType.rent)
-           Text("Buy")
+           TownRankingListView(selection: selection)
                .tag(TabType.buy)
        }
        .tabViewStyle(PageTabViewStyle(indexDisplayMode: .never))
        .animation(.linear(duration: 0.3))
+       .edgesIgnoringSafeArea(.bottom)
    }
}
```

今回の仕様としては，1行で10列，それぞれのセルとのマージンは 10pt とします。
左右のマージンは 16pt ずつですがマージンの設定は，
訳ありで `ForEach` で View 生成の際に設定するようにします。

一旦セルの代わりに `Rectangle` で代用します。
借りるだったらオレンジ，買うだったらブルーで塗りつぶします。

セルと左右上下端との余白，`UICollectionView` でいうところの
`UICollectionViewFlowLayout` の `sectionInset` は，
`.padding(.all, 16.0)` で表現できます。深く考えなくていいので楽かもしれないですね。

```swift:TownRankingListView.swift
struct TownRankingListView: View {

    // このViewで変更は起きないので値渡しだけで良い
    let selection: TabType

    var body: some View {
        ScrollView {
            LazyVGrid(columns: [GridItem()], spacing: 0.0) {
                ForEach(0 ..< 10) { index in
                    Rectangle()
                        .fill(selection == .rent ? Color.rentOrange: Color.buyBlue)
                        .frame(height: 50)
                        .cornerRadius(8.0)
                        .padding(.bottom, 10.0)
　　　　　　　　　　　　　　// ↑今回は擬似的に minimumLineSpacing 相当とする
                       // 本来は LazyVGrid の spacing で設定する
                }
            }
            .padding(.all, 16.0) // sectionInset相当
        }
        .background(Color.gridBackground)
    }
}
```

実行すると下記のようになります。

GIF

#### 街ランキングリスト部分(1~10位のセル実装)

今までは `Rectangle` で代用していましたが，
ランキングと街情報を表示するセルを実装していきます。
シンプルイズベストということで，サイトと同じようなデザインにします。
**私が仕様** なので画像は SFSymbols から探してガンガン使ってます。

:::message
**仕様**
* 1位〜3位までは王冠付きのランク表示
* 街名，ランク情報，最寄りの路線を表示する
* 画像は SFSymbols から探す
:::

`TownRowView` というクラスで，`HStack` と `VStack` を使って実装します。
ただ，1~3位の王冠のサイズを大きくしたり，順位変動部分の幅が可変だったりすることもあって，`ZStack` を使って領域分の幅を固定で取っています。
また，現時点ではそれぞれの値は固定値としています。
三項演算子ばかりで少し可読性悪いかもですね🤔
↓ 複雑な割に大したことはないのでトグルにします。

:::details TownRowView の実装

```swift:TownRowView.swift
import SwiftUI

struct TownRowView: View {
    let selection: TabType
    let rank: Int
    let isRankUp: Bool
    let rankFluctuation: Int

    var body: some View {
        HStack(spacing: 16.0) {
            ZStack {
                Rectangle()
                    .fill(Color.clear)
                    .frame(width: 50.0, height: 50.0)
                Image(systemName: rank < 4 ? "crown.fill": "circle.fill")
                    .resizable()
                    .scaledToFit()
                    .frame(width: rank < 4 ? 50.0: 30.0, height: rank < 4 ? 50.0: 30.0)
                    .offset(x: .zero, y: rank < 4 ? -5.0: .zero)
                    .foregroundColor(selection == .rent ? .rentOrange: .buyBlue)
                Text(rank.description)
                    .font(rank < 4 ? .title2: .subheadline)
                    .bold()
                    .foregroundColor(.white)
            }
            ZStack {
                Rectangle()
                    .fill(Color.clear)
                    .frame(width: 50.0, height: 50.0)
                VStack(spacing: 4.0) {
                    if !isRankUp && rankFluctuation == 0 {
                        Image(systemName: "minus.circle.fill")
                            .resizable()
                            .frame(width: 20.0, height: 20.0)
                            .foregroundColor(.gray)
                        Text("キープ")
                            .font(.footnote)
                            .foregroundColor(.gray)
                    } else {
                        Image(systemName: isRankUp ? "arrow.up.circle.fill": "arrow.down.circle.fill")
                            .resizable()
                            .frame(width: 20.0, height: 20.0)
                            .foregroundColor(isRankUp ? .red: .blue)
                        Text(rankFluctuation.description + (isRankUp ? "アップ": "ダウン"))
                            .font(.footnote)
                            .foregroundColor(isRankUp ? .red: .blue)
                    }
                }
            }
            Text("浦安")
                .font(.body)
                .bold()
            Spacer()
            Text("東京メトロ東西線")
                .frame(width: 90.0)
                .lineLimit(nil)
                .foregroundColor(.gray)
                .font(.footnote)
        }
        .padding(.horizontal, 16.0)
        .padding(.vertical, 10.0)
        .background(Color.rowBackground)
        .cornerRadius(8.0)
        .shadow(color: Color.black.opacity(0.16), radius: 3.0, x: .zero, y: 3.0)
    }
}

struct TownRowView_Previews: PreviewProvider {
    static var previews: some View {
        TownRowView(selection: .rent, rank: 1, isRankUp: true, rankFluctuation: 10)
            .previewLayout(PreviewLayout.sizeThatFits)
            .padding()
        TownRowView(selection: .buy, rank: 4, isRankUp: false, rankFluctuation: 5)
            .previewLayout(PreviewLayout.sizeThatFits)
            .padding()
        TownRowView(selection: .rent, rank: 10, isRankUp: false, rankFluctuation: 0)
            .previewLayout(PreviewLayout.sizeThatFits)
            .padding()
    }
}
```
:::

こういう View を作るときはプレビューが便利ですよね。
状態や値を変えて並べてみました。
この View がランキングリストの各セルとして表示されます。

![](https://storage.googleapis.com/zenn-user-upload/ntm6yjrj8uabg0wwohfijr3jxjn1 =600x)

`TownRankingListView` のセルビュー生成部分を書き換えます。

```diff swift:TownRankingListView.swift
struct TownRankingListView: View {

    let selection: TabType

    var body: some View {
        ScrollView {
            LazyVGrid(columns: [GridItem()], spacing: 0.0) {
                ForEach(0 ..< 10) { index in
-                   Rectangle()
-                       .fill(selection == .rent ? Color.rentOrange: Color.buyBlue)
-                       .frame(height: 50)
-                       .cornerRadius(8.0)
+                   TownRowView(selection: selection, rank: 1, isRankUp: true, rankFluctuation: 10)
                        .padding(.bottom, 10.0)
                }
            }
            .padding(.all, 16.0)
        }
        .background(Color.gridBackground)
    }
}
```

実行するとこんな感じになります。10個それぞれ表示されます。

|借りて住みたい|買って住みたい|
|:--:|:--:|
|![](https://storage.googleapis.com/zenn-user-upload/3gmdl6sl9myvi2b2d4opdm3xkru2)|![](https://storage.googleapis.com/zenn-user-upload/vw9ppeqvnthuypyzu32cpkavbsdl)|

:::message
* `LazyVGrid` の使い方を抑える
:::

### 街ランキングリスト部分(11位以降の表示/非表示対応)

参考にした住みたい街ランキングサイトを見ると，
11位以降はボタンタップで表示・非表示が切り替えられるので，
同じようにアコーディオンチックな実装にしてみます。

11〜20位の仕様は下記のようにしてみました。

:::message
**仕様**
* 11〜20位はボタンのタップで表示・非表示を切り替える
* 11〜20位はマージンなしのリスト表示
* 表示はランク，ランク増減，街名，最寄り路線
* ランク部分は全て丸で背景色は色は少し薄めのオレンジ・ブルーに
* 利用可能路線情報は街名の下に表示
:::

#### 街ランキングリスト部分(11~20位のセル実装)

まずは，11〜20位用のセルの View を実装してみます。
1〜10位用に先ほど実装した `TownRowView` を使おうと思ったのですが，
表示させる項目は同じではあるのですが，三項演算子も多く，
レイアウトも異なるためさらに複雑にな理想だったので新規で作ります。
`SubTownRowView` とします。角丸設定も不要です。

:::details 11〜20位のセル実装 SubTownRowView
```swift:
struct SubTownRowView: View {
    let selection: TabType
    let rank: Int
    let isRankUp: Bool
    let rankFluctuation: Int

    var body: some View {
        HStack(spacing: 16.0) {
            ZStack {
                Rectangle()
                    .fill(Color.clear)
                    .frame(width: 50.0, height: 50.0)
                Image(systemName: "circle.fill")
                    .resizable()
                    .scaledToFit()
                    .frame(width: 30.0, height: 30.0)
                    .foregroundColor(selection == .rent ? .subRentOrange: .subBuyBlue)
                Text(rank.description)
                    .font(.subheadline)
                    .bold()
                    .foregroundColor(.white)
            }
            ZStack {
                Rectangle()
                    .fill(Color.clear)
                    .frame(width: 50.0, height: 50.0)
                VStack(spacing: 4.0) {
                    if !isRankUp && rankFluctuation == 0 {
                        Image(systemName: "minus.circle.fill")
                            .resizable()
                            .frame(width: 20.0, height: 20.0)
                            .foregroundColor(.gray)
                        Text("キープ")
                            .font(.footnote)
                            .foregroundColor(.gray)
                    } else {
                        Image(systemName: isRankUp ? "arrow.up.circle.fill": "arrow.down.circle.fill")
                            .resizable()
                            .frame(width: 20.0, height: 20.0)
                            .foregroundColor(isRankUp ? .red: .blue)
                        Text(rankFluctuation.description + (isRankUp ? "アップ": "ダウン"))
                            .font(.footnote)
                            .foregroundColor(isRankUp ? .red: .blue)
                    }
                }
            }
            VStack(alignment: .leading, spacing: 4.0) {
                Text("妙典")
                    .font(.subheadline)
                    .bold()
                Text("東京メトロ東西線")
                    .lineLimit(nil)
                    .foregroundColor(.gray)
                    .font(.footnote)
            }
            Spacer()
        }
        .padding(.horizontal, 16.0)
        .padding(.vertical, 4.0)
        .background(Color.rowBackground)
    }
}

struct SubTownRowView_Previews: PreviewProvider {
    static var previews: some View {
        SubTownRowView(selection: .rent, rank: 11, isRankUp: true, rankFluctuation: 10)
            .previewLayout(PreviewLayout.sizeThatFits)
            .padding()
        SubTownRowView(selection: .buy, rank: 20, isRankUp: false, rankFluctuation: 5)
            .previewLayout(PreviewLayout.sizeThatFits)
            .padding()
        SubTownRowView(selection: .rent, rank: 20, isRankUp: false, rankFluctuation: 0)
            .previewLayout(PreviewLayout.sizeThatFits)
            .padding()
    }
}
```
:::

先ほどと同じように状態や値を変えて並べてみました。
この View がランキングリスト 11〜20位の各セルとして表示されます。

![](https://storage.googleapis.com/zenn-user-upload/bjxb56giemoihsevnn8dbm9vek6f =600x)

#### 街ランキングリスト部分(11~20位のセル表示)

11〜20位のセルもランキングリストに表示させてみます。
リスト表示させたいので，`Section` でそれぞれ異なるブロックにします。
`Divider()` を使ってセルとセルとの間にラインを引いてリスト表示っぽくしています。
11〜20位のレイアウトために，`LazyVGrid` の `spacing` を 0 にしていた感じです。
(`ForEach` の範囲は 0〜9 までになっていますが あとで 11〜20位の要素用に変更されます。)

```diff swift:TownRankingListView.swift
struct TownRankingListView: View {

    let selection: TabType

    var body: some View {
        ScrollView {
            // 11〜20位のレイアウトためにこのspacingを0にしていた
            LazyVGrid(columns: [GridItem()], spacing: 0.0) {
+               Section {
                    ForEach(0 ..< 10) { index in
                        TownRowView(selection: selection, rank: 1, isRankUp: true, rankFluctuation: 10)
                            .padding(.bottom, 10.0)
                    }
+               }
+               Section {
+                   ForEach(0 ..< 10) { index in
+                       SubTownRowView(selection: selection, rank: 11, isRankUp: true, rankFluctuation: 30)
+                       Divider()
+                   }
+               }
            }
            .padding(.all, 16.0)
        }
        .background(Color.gridBackground)
    }
}
```

実行するとこんな感じになります。さらに下に 10個それぞれ表示されます。
あと少し。

|借りて住みたい|買って住みたい|
|:--:|:--:|
|![](https://storage.googleapis.com/zenn-user-upload/jvgf33fjktdyj6x56f6pd2vxpuql)|![](https://storage.googleapis.com/zenn-user-upload/fcwe6r8aytn500ntqtafe4sw7r1n)|

#### 街ランキングリスト部分(表示・非表示の実現)

最後に11〜20位のランキングの表示・非表示を実現させていきます。
11〜20位のランキングの表示，非表示という状態を表現するために
`isExpanded` という変数を定義し，`isExpanded` が `true` の場合のみ
11〜20位のランキングリストを表示させるように実装します。

```diff swift:TownRankingListView.swift
struct TownRankingListView: View {

    let selection: TabType
+   @State private var isExpanded = false

    var body: some View {
        ScrollView {
            LazyVGrid(columns: [GridItem()], spacing: 0.0) {
                Section {
                    ForEach(0 ..< 10) { index in
                        TownRowView(selection: selection, rank: 1, isRankUp: true, rankFluctuation: 10)
                            .padding(.bottom, 10.0)
                    }
                }
+               if isExpanded {
                    Section {
                        ForEach(0 ..< 10) { index in
                            SubTownRowView(selection: selection, rank: 11, isRankUp: true, rankFluctuation: 30)
                            Divider()
                        }
                    }
                }
+           }
            .padding(.all, 16.0)
        }
        .background(Color.gridBackground)
    }
}
```

`isExpanded` の状態切り替えは，`Button` のアクションに任せます。
`ExpandButtonView` を新規で実装します。
`isExpanded` の値によってボタンのタイトルを変更し，アロー画像の向きを変更させます。

:::details ExpandButtonView の実装

```swift:ExpandButtonView.swift
struct ExpandButtonView: View {

    let selection: TabType
    // このViewでの状態変化をランキングリストのViewに伝える
    @Binding var isExpanded: Bool

    var body: some View {
        Button {
            // ボタンのタップアクションで状態を切り替え
            self.isExpanded.toggle()
        } label: {
            HStack {
                Text(isExpanded ? "閉じる": "11位以降を見る")
                    .font(.subheadline)
                    .bold()
                    .foregroundColor(.white)
                Image(systemName: isExpanded ? "chevron.up": "chevron.down")
                    .foregroundColor(.white)
            }
            .frame(height: 52.0)
            .frame(minWidth: .zero, maxWidth: .infinity)
            .background(selection == .rent ? Color.rentOrange: Color.buyBlue)
            .cornerRadius(8.0)
        }
    }
}

struct ExpandButtonView_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            ExpandButtonView(selection: .rent, isExpanded: .constant(false))
                .previewLayout(PreviewLayout.sizeThatFits)
                .padding()
                .background(Color(.systemBackground))
            ExpandButtonView(selection: .rent, isExpanded: .constant(true))
                .previewLayout(PreviewLayout.sizeThatFits)
                .padding()
                .background(Color(.systemBackground))
        }
    }
}
```
:::

`isExpanded` の値によって下記のようなビューの違いになります。
また，`selection` の値によって背景色が変わります。

![](https://storage.googleapis.com/zenn-user-upload/7yisxv8b0vv0lxweh3zi5ehsrv44 =600x)

最後にランキングリストの一番下にこのボタンが表示されるように実装追加します。
一番下の SafeArea の高さ分ボタンの下に追加してやると見栄えが良くなります。
SafeArea の高さの取得方法は何種類かあると思いますが，
`ContentView` で `geometry.safeAreaInsets.bottom` で取れるので値渡しで持ってきました。

```diff swift:TownRankingListView.swift
struct TownRankingListView: View {

    let selection: TabType
+   let safeAreaBottomHeight: CGFloat
    @State private var isExpanded = false

    var body: some View {
        ScrollView {
            LazyVGrid(columns: [GridItem()], spacing: 0.0) {
                Section {
                    ForEach(0 ..< 10) { index in
                        TownRowView(selection: selection, rank: 1, isRankUp: true, rankFluctuation: 10)
                            .padding(.bottom, 10.0)
                    }
                }
                if isExpanded {
                    Section {
                        ForEach(0 ..< 10) { index in
                            SubTownRowView(selection: selection, rank: 11, isRankUp: true, rankFluctuation: 30)
                            Divider()
                        }
                    }
                }
            }
            .padding(.all, 16.0)
+           ExpandButtonView(selection: selection, isExpanded: $isExpanded)
+               .padding(.horizontal, 16.0)
+               .padding(.bottom, 16.0 + safeAreaBottomHeight)
        }
        .background(Color.gridBackground)
    }
}
```

実行してみると動きは実現できています。
あとはランキングデータを取得して表示させるだけです。

GIF

### ランキングリスト用の街データモデルの実装

住みたい街ランキング用の JSON を元にデータを扱うモデルを実装します。
複雑ではないです。今回は `Encodable` の方はいらないです。

```swift:
struct TownRankingData: Decodable {
    var townRankingsForRent: [TownInfo]
    var townRankingsForBuy: [TownInfo]
}

struct TownInfo: Decodable {
    var rank: Int
    var townName: String
    var isRankUp: Bool
    var rankFluctuation: Int
    var availableLine: String
}
```

### JSON のデータをリストに表示

先述したとおり今回の記事ではローカルの JSON ファイルからデータを取得します。
`TownRankingFetcher` クラスを作って，取得結果をコールバックさせます。

```swift:TownRankingFetcher.swift
class TownRankingFetcher {
    func fetchTownRanking(completion: @escaping (Result<TownRankingData, Error>) -> Void) {
        guard let path = Bundle.main.path(forResource: "TownRanking2021", ofType: "json") else { return }
        do {
            let data = try Data(contentsOf: URL(fileURLWithPath: path))
            let fetchedData = try JSONDecoder().decode(TownRankingData.self, from: data)
            DispatchQueue.main.async {
                completion(.success(fetchedData))
            }
        } catch  {
            DispatchQueue.main.async {
                completion(.failure(error))
            }
        }
    }
}
```

:::details APIのリンク用意して URLSession など使って通信する場合
```swift:TownRankingFetcher.swift
class TownRankingFetcher {
    func fetchTownRanking(completion: @escaping (Result<TownRankingData, Error>) -> Void) {
        URLSession.shared.dataTask(with: URL(string: APIのリンク)!) { (data, response, error) in
            guard let data = data else { return }
            let decoder: JSONDecoder = JSONDecoder()
            do {
                let fetchedData = try decoder.decode(TownRankingData.self, from: data)
                DispatchQueue.main.async {
                    completion(.success(fetchedData))
                }
            } catch {
                DispatchQueue.main.async {
                    completion(.failure(error))
                }
            }
        }.resume()
    }
}
```
:::

外部API を叩いてリスト表示するサンプル実装は
Qiita に以前書いたのでよかったらご覧ください🙇‍♂️
https://qiita.com/MilanistaDev/items/64dca8c9d5099a19529e

次に ViewModel を実装します。
`ObservableObject` に準拠させて，変更があった際に通知可能にします。
データのモデルのプロパティに `@Published` 属性を付加して監視対象とします。
`townRankingData` の初期化として借りて住みたい，買って住みたい，
それぞれのランキングデータの配列を空にしておいて JSON からデータ取得後に
変更を通知できるようになる感じです(簡単にいうと)。

```swift:TownRankingViewModel.swift
class TownRankingViewModel: ObservableObject {
    @Published var townRankingData =  TownRankingData(townRankingsForRent: [], townRankingsForBuy: [])

    let fetcher = TownRankingFetcher()

    init() {
        fetcher.fetchTownRanking { result in
            switch result {
            case .success(let townRankingData):
                // 取得したランキングデータを代入して変更をこのクラスを監視しているViewに通知
                self.townRankingData = townRankingData
            case .failure(let error):
                debugPrint(error.localizedDescription)
            }
        }
    }
}
```

データの変化を受け取って View を更新する処理を実装します。
今回の親 View の `ContentView` で ViewModel を定義して，
`@ObservedObject` 属性を付与します。
これでランキングデータ取得時に変更を受け取ることができます。
`TownRankingViewModel` のランキングデータモデル `townRankingData` を
子の View に渡していきます。

```diff swift:ContentView.swift
struct ContentView: View {
    @State private var selection: TabType = .rent
+   @ObservedObject private var townRankingVM = TownRankingViewModel()

    var body: some View {
        GeometryReader { geometry in
            NavigationView {
                VStack(spacing: .zero) {
                    UpperTabView(selection: $selection,
                                 geometrySize: geometry.size)
                    ContentPageView(selection: $selection,
+                                   townRankingData: townRankingVM.townRankingData,                                 
                                    safeAreaBottomHeight: geometry.safeAreaInsets.bottom)
                }
                .edgesIgnoringSafeArea(.bottom)
                .navigationBarTitle("住みたい街ランキング2021(首都圏)",
                                    displayMode: .inline)
            }
        }
    }
}
```

`ContentPageView` に `TownRankingData` 型で定数を宣言して値渡しできるようにします。
今回は，子Viewでデータの変更は行わないのでただの値渡しです。
ランキングリストの `TownRankingListView` にも値渡しします。
今回は，借りて住みたい，買って住みたいランキングデータのモデルをそれぞれ渡します。

```diff swift:
struct ContentPageView: View {
    // ページングすることで状態が変わるので @Binding を使う
    @Binding var selection: TabType
    // 子Viewでデータの変更は行わないので値渡しで良い
+   let townRankingData: TownRankingData
    let safeAreaBottomHeight: CGFloat

    var body: some View {
        TabView(selection: $selection) {
            TownRankingListView(selection: selection,
+                               townInfo: townRankingData.townRankingsForRent,
                                safeAreaBottomHeight: safeAreaBottomHeight)
                .tag(TabType.rent)
            TownRankingListView(selection: selection,
+                               townInfo: townRankingData.townRankingsForBuy,
                                safeAreaBottomHeight: safeAreaBottomHeight)
                .tag(TabType.buy)
        }
        .tabViewStyle(PageTabViewStyle(indexDisplayMode: .never))
        .animation(.linear(duration: 0.3))
    }
}
```

ランキングリストの `TownRankingListView` で宣言するのは，
`[TownInfo]` 型の定数です。今回は 20要素分あるので `min関数` を使って，
1〜10位，11〜20位の範囲の要素のデータを表示できるように `ForEach` の範囲を調整しています。
静的なデータをセル View に渡していたところを各街データ(`TownInfo`)に変更しています。
`index` 使わないのに書いてたのはこのためでした。
最下部のボタンは，データ取得後に表示させるために条件文を追加しています。

```diff swift:
struct TownRankingListView: View {

    let selection: TabType
+   let townInfo: [TownInfo]
    let safeAreaBottomHeight: CGFloat
    @State private var isExpanded = false

    var body: some View {
        ScrollView {
            LazyVGrid(columns: [GridItem()], spacing: 0.0) {
                Section {
-                   ForEach(0 ..< 10) { index in
-                       TownRowView(selection: selection, rank: 1, isRankUp: true, rankFluctuation: 10)
-                           .padding(.bottom, 10.0)
-                   }
+                   ForEach(0 ..< min(townInfo.count, 10)) { index in
+                       TownRowView(selection: selection,
+                                   townInfo: townInfo[index])
+                           .padding(.bottom, 10.0)
+                   }
                }
                if isExpanded {
                    Section {
-                       ForEach(0 ..< 10) { index in
-                           SubTownRowView(selection: selection, rank: 11, isRankUp: true, rankFluctuation: 30)
-                           Divider()
-                       }
+                       ForEach(min(townInfo.count, 10) ..< townInfo.count) { index in
+                           SubTownRowView(selection: selection,
+                                          townInfo: townInfo[index])
+                           Divider()
+                       }
                    }
                }
            }
            .padding(.all, 16.0)
            // データ取得前は表示させない
+           if !townInfo.isEmpty {
                ExpandButtonView(selection: selection, isExpanded: $isExpanded)
                    .padding(.horizontal, 16.0)
                    .padding(.bottom, 16.0 + safeAreaBottomHeight)
+           }
        }
        .background(Color.gridBackground)
    }
}
```

最後にセルのViewのデータの値渡しです。
`let townInfo: TownInfo` を定義して値を受けられるようにします。
`TownInfo` に街・ランク情報が入ってくるのでそのデータを参照するように書き換えます。
ここでは省略します。

実行してみます。

**これで終わりだと思った・・・**

借りて住みたい側の 1〜10位の表示が出ません。むむむ🤔
ボタンは表示されているし，11〜20位のデータもあるので全部取れているはずです。

GIF

検索してみたところ，下記記事にも同様の内容の記載がありました。
https://hack.nikkei.com/blog/advent20201201/

> TabViewのContentViewを静的な固定値ではなく、APIから動的に取得する場合、再レンダリングが正常に働かない挙動を確認しています。 TabViewにidメソッドを適用して、コンテンツに変化が起きたら強制的に再レンダリングを行わせることで対応できます。

iOS 15 で解消されればいいなぁ。
ということで同様の処理をしてみました。

まずはモデルの構造体を `Hashable` に準拠させます。

```swift:TownInfo.swift
// Hashable に準拠させる
struct TownInfo: Decodable, Hashable {
    var rank: Int
    var townName: String
    var isRankUp: Bool
    var rankFluctuation: Int
    var availableLine: String
}
```

次に `TabView` に `id` メソッドを適用させます。

```diff swift:ContentPageView.swift
struct ContentPageView: View {

    @Binding var selection: TabType
    let townRankingData: TownRankingData
    let safeAreaBottomHeight: CGFloat

    var body: some View {
        TabView(selection: $selection) {
            TownRankingListView(selection: selection,
                                townInfo: townRankingData.townRankingsForRent,
                                safeAreaBottomHeight: safeAreaBottomHeight)
                .tag(TabType.rent)
            TownRankingListView(selection: selection,
                                townInfo: townRankingData.townRankingsForBuy,
                                safeAreaBottomHeight: safeAreaBottomHeight)
                .tag(TabType.buy)
        }
        .tabViewStyle(PageTabViewStyle(indexDisplayMode: .never))
        .animation(.linear(duration: 0.3))
+       // APIで取得しても画面更新されないため
+       .id(townRankingData.townRankingsForRent.hashValue)
    }
}
```

上記修正を追加したらちゃんと 1〜10位も表示されました。

GIF

## おわりに

今回は，UIのイメージと使ってみたい新要素だけを持って実装進めましたが，
意外と作れるものだなぁが7割，これどう実装するんだ？3割くらいで楽しかったです。
幅広く復習できたのでまぁまぁ満足してます。

SwiftUI は業務でまださわれない分，個人開発で！という感じで付き合っています。
今作ってる新規アプリは，UIKit でまず画面単位で作って SwiftUI で作れそうなら
`UIHostingController` を利用して採用という形でやっています。
`ContainerView` などで部分的に作れる場合も考慮はしています。

UIKit じゃなくても実現できることが増えてきたので，
普段から触っておくのが良いなと思いました。
ただ，iOS 14 (SwiftUI 2)以上でやっと使えるレベルだと思ってるので
ストレスなく開発できるようになるまで正直我慢ですね。

ご覧いただきありがとうございました。

## 参考

[^1]:https://developer.apple.com/documentation/swiftui/withanimation(_:_:)
[^2]:https://developer.apple.com/documentation/swiftui/view/animation(_:)
[^3]:https://developer.apple.com/documentation/swiftui/lazyvgrid
