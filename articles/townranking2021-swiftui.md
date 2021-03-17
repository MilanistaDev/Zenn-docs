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

## 今回抑えたい内容

* `HStack`，`VStack`，`ZStack` などの基本的な内容の復習
* `@State`，`@Binding` などの値の扱い
* `@ObservedObject`，`@Observable`，`@Published` を利用したデータの扱い
* iOS 14 で追加された TabView の `PageTabViewStyle` を使ってみる
* 簡単な Animation の復習
  - 状態を変化に対して (`withAnimation`)
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
                VStack {
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
    // 左右にページングすることで状態が変わるので @Binding を使う
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
                VStack {
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
`animation(_:)` モディファイアを使います。
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


### 街ランキングリスト部分(リスト表示)




## 実行結果

## Future Work

## おわりに

SwiftUI は業務でまださわれない分，個人開発で！という感じで付き合っています。
今作ってる新規アプリは，UIKit でまず画面単位で作って SwiftUI で作れそうなら
`UIHostingController` を利用して採用という形でやっています。
`ContainerView` などで部分的に作れる場合も考慮はしています。
iOS 14 (SwiftUI 2)以上でやっと使えるレベルだと思ってるので
ストレスなく開発できるようになるまで正直我慢ですね。

ご覧いただきありがとうございました。

## 参考

[^1]:https://developer.apple.com/documentation/swiftui/withanimation(_:_:)
