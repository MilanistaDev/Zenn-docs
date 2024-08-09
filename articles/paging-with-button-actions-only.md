---
title: "[SwiftUI] ボタンアクションのみでページングさせる実装を考える"
emoji: "📱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iOS", "SwiftUI", "Paging"]
published: true
---

## はじめに

今回も SwiftUI の `TabView` 周りの小ネタです。

ある案件で，
「コンテンツ画面に一時的に特別なコンテンツも表示したい」
という話をいただき，ページングさせる形式で
基本コンテンツ↔️特別コンテンツの切り替えを行うという施策がありました。

いつも通り `PageTabViewStyle` 使えば良いよな〜
と思って仕様詰めをしていたのだが，
**「ボタンによるコンテンツ切り替えのみにしたい」** 
という追加のご希望があった。

`TabView` 使う想定のままなんとかいけそうか，
できないなら代替案を考えようということで調査に取り掛かった。

## 結論

`PageTabViewStyle` の `TabView` を使う。
ただし，ドラッグ/スワイプでページングができてしまうため，
ドラッグジェスチャーを各ページのコンテンツのビューに付与し，
子ビュー側のドラッグジェスチャーを優先させる。

`.gesture(DragGesture())`

## 仕様とサンプルアプリ

今回のサンプルアプリの GitHub は下記です。

https://github.com/MilanistaDev/PagingWithButtonActionsOnly

* iOS 16 以上
* SwiftUI
* MVVM

動作イメージは下記の GIF のような感じ。

![tab_paging_result](https://github.com/user-attachments/assets/0abf5185-71ad-428c-940c-11fd19c9f523)

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

### ViewModel 実装

ViewModel 側でデータ取得の処理を行う。
非同期を意識して 3s の遅延を入れた。
待つ間 `isLoading` が `true` になる。

```swift:TabPagingViewModel.swift
import Foundation

final class TabPagingViewModel: ObservableObject {
    @Published var contents: [Contents] = []
    @Published var isLoading = false

    func onAppear() {
        Task {
            isLoading = true
            defer {
                isLoading = false
            }

            do {
                // wait 3s
                try await Task.sleep(nanoseconds: 3_000_000_000)
                contents = sampleContents

            } catch { }
        }
    }
}
```

### TabView によるページング仮実装

View 側の実装は下記のような感じ。
一旦通常通りスワイプでページングできるようにしておく。

```swift:TabPagingView.swift
import SwiftUI

struct TabPagingView: View {
    @StateObject private var viewModel = TabPagingViewModel()
    @State private var selection = 0

    var body: some View {
        TabView(selection: $selection) {
            ForEach(viewModel.contents.indices, id: \.self) { index in
                ContentView(data: viewModel.contents[index].columns)
                    .tag(index)
                    .ignoresSafeArea()
            }
        }
        .tabViewStyle(.page(indexDisplayMode: .never))
        .edgesIgnoringSafeArea(.top)
        .overlay {
            if viewModel.isLoading {
                ProgressView()
            }
        }
        .onAppear {
            viewModel.onAppear()
        }
    }
}
```

### ページ切り替えボタン実装

次にページ切り替え用のボタンの実装をします。
左右のボタンの設定のための enum を定義しておきます。

```swift:PageSwitchingButtonType.swift
enum PageSwitchingButtonType: CaseIterable {
    case left
    case right

    // ボタンタップ時に追加するIndex値
    var addIndex: Int {
        switch self {
        case .left:
            return -1

        case .right:
            return 1
        }
    }

    // 左右のボタンのシェブロンのアイコン名(SFSymbols)
    var edgeIcon: String {
        switch self {
        case .left:
            return "chevron.left"

        case .right:
            return "chevron.right"
        }
    }

    // 左端のマージン
    var leadingMargin: CGFloat {
        switch self {
        case .left:
            return 4.0

        case .right:
            return 8.0
        }
    }

    // 右端のマージン
    var trailingMargin: CGFloat {
        switch self {
        case .left:
            return 8.0

        case .right:
            return 4.0
        }
    }

    // 左端の角丸の設定
    var leadingCornerRadius: CGFloat {
        switch self {
        case .left:
            return .zero

            case .right:
                return 8.0
        }
    }

    // 右端の角丸の設定
    var trailingCornerRadius: CGFloat {
        switch self {
        case .left:
            return 8.0

        case .right:
            return .zero
        }
    }
}

```

ボタン本体の実装は下記の通りです。
左ボタンか右ボタンかで表示の違いがあるので
`PageSwitchingButtonType` を引数としてもらって実装し分けています。

```swift:PageSwitchingButton.swift
struct PageSwitchingButton: View {
    let content: Contents
    let type: PageSwitchingButtonType
    var onTap: (() -> Void)?

    var body: some View {
        Button {
            onTap?()
        } label: {
            HStack(spacing: 8.0) {
                if type == .left {
                    Image(systemName: type.edgeIcon)
                        .resizable()
                        .scaledToFit()
                        .frame(width: 16.0, height: 16.0)
                }

                Text(content.pageName)
                    .font(.headline)

                if type == .right {
                    Image(systemName: type.edgeIcon)
                        .resizable()
                        .scaledToFit()
                        .frame(width: 16.0, height: 16.0)
                }
            }
            .padding(.leading, type.leadingMargin)
            .padding(.trailing, type.trailingMargin)
            .frame(height: 32.0)
            .foregroundColor(Color.init(hex: content.textColor))
            .background(Color.init(hex: content.backColor))
            .clipShape(
                .rect(
                    topLeadingRadius: type.leadingCornerRadius,
                    bottomLeadingRadius: type.leadingCornerRadius,
                    bottomTrailingRadius: type.trailingCornerRadius,
                    topTrailingRadius: type.trailingCornerRadius
                )
            )
            .compositingGroup()
            .shadow(color: .black.opacity(0.6), radius: 2.0, x: 0.0, y: 0.0)
        }
    }
}
```

コンテンツの上にページ切り替えボタンを表示させます。
`ZStack` でもいいですがネスト深くなるので
今回は `overlay` モディファイアを使うことにします。


```diff swift:TabPagingView.swift
struct TabPagingView: View {
    @StateObject private var viewModel = TabPagingViewModel()
    @State private var selection = 0

    var body: some View {
        TabView(selection: $selection) {
            ForEach(viewModel.contents.indices, id: \.self) { index in
                ContentView(data: viewModel.contents[index].columns)
                    .tag(index)
                    .ignoresSafeArea()
            }
        }
        .tabViewStyle(.page(indexDisplayMode: .never))
        .edgesIgnoringSafeArea(.top)
+       .overlay(alignment: .top) {
+           PageSwitchingButtons(
+               contents: viewModel.contents,
+               selection: $selection
+           )
+           .padding(.top, 20.0)
+       }
        .overlay {
            if viewModel.isLoading {
                ProgressView()
            }
        }
        .onAppear {
            viewModel.onAppear()
        }
    }
}
```

これで実行するとページ切り替えボタンでのページングが実現できています。
加えて，まだスワイプによるページングもまた可能なままです。

![tab_paging](https://github.com/user-attachments/assets/3440b1dd-5648-4da5-9948-cf41df1aaeb5)

### TabView のページングをなくす

次にスワイプによるページングをできなくします。
親ビューより子ビューのジェスチャーが優先される仕組みを使って，
ページのコンテンツビューに `DragGesture` を付与して
その `DragGesture` では結局何もしないという形で実現します。
`TabView` のページングよりコンテンツの `DragGesture` が優先され，
`TabView` のページングがキャンセルされるといった感じ(認識)です。
(認識違ってたらご指摘いただけると嬉しいです🙇)

↓`DragGesture` を `ContentView` (各ページのビュー)に対して追加しています。

```diff swift:TabPagingView.swift
struct TabPagingView: View {
    @StateObject private var viewModel = TabPagingViewModel()
    @State private var selection = 0

    var body: some View {
        TabView(selection: $selection) {
            ForEach(viewModel.contents.indices, id: \.self) { index in
                ContentView(data: viewModel.contents[index].columns)
                    .tag(index)
+                   .gesture(DragGesture())
                    .ignoresSafeArea()
            }
        }
        .tabViewStyle(.page(indexDisplayMode: .never))
        .edgesIgnoringSafeArea(.top)
        .overlay(alignment: .top) {
            PageSwitchingButtons(
                contents: viewModel.contents,
                selection: $selection
            )
            .padding(.top, 20.0)
        }
        .overlay {
            if viewModel.isLoading {
                ProgressView()
            }
        }
        .onAppear {
            viewModel.onAppear()
        }
    }
}
```

これで実行するとページ切り替えボタンだけのページングが実現できました。

![tab_paging_fixed](https://github.com/user-attachments/assets/ef949cb2-8844-466e-b1b3-78e42ed8e730)


該当箇所で下記みたいなコード書いてみて，
コンテンツビュー側をドラッグしてみると
print 文の出力があることがわかります。

```swift
.gesture(
    DragGesture()
        .onEnded({ _ in
            print("Dragged ContentView side.")
        })
)
```

## 他に考えた案

今回はあっさり実現できたが，頭にあった案を軽く書いてみます。

### `ScrollViewReader` 使う案

`ScrollViewReader` をラップして，
`scrollDisabled(true)` でスクロールできなくして，
`scrollTo()` で次のコンテンツにスクロールさせる案。

サンプルアプリの2番目のタブに実装しました。

```swift:ScrollViewReaderPagingView.swift
struct ScrollViewReaderPagingView: View {
    @StateObject private var viewModel = TabPagingViewModel()
    @State private var selection = 0

    var body: some View {
        ScrollViewReader { scrollProxy in
            ScrollView(.horizontal) {
                HStack(spacing: .zero) {
                    ForEach(viewModel.contents.indices, id: \.self) { index in
                        ContentView(data: viewModel.contents[index].columns)
                            .tag(index)
                            .frame(width: UIScreen.main.bounds.width) // よくない🤔
                    }
                }
            }
            .scrollDisabled(true)
            .onChange(of: selection) { _ in
                withAnimation {
                    // selection の値変更を受けて各Index値にスクロールさせる
                    scrollProxy.scrollTo(selection, anchor: .center)
                }
            }
        }
        .ignoresSafeArea(edges: [.top])
        .overlay(alignment: .top) {
            PageSwitchingButtons(
                contents: viewModel.contents,
                selection: $selection
            )
            .padding(.top, 20.0)
        }
        .overlay {
            if viewModel.isLoading {
                ProgressView()
            }
        }
        .onAppear {
            viewModel.onAppear()
        }
    }
}
```

これでも良さそうです。
これだと動きは良いのですが，
各ページ表示時にスクリーンイベントを取りたいということになり，
ビュー生成時に一気に呼ばれることから不採用になりました。

### `NavigationLink` による Push/Pop 遷移

左右に動くアニメーションといえば・・・
`NavigationLink` を使った Push & Pop な遷移での実現も考えました。
しかし，親子の関係でもなく用途が違いすぎるためボツになりました。

### 座標をずらすパターン

`ScrollViewReader` だとスワイプもできちゃうと思っていた頃の原始案。
ページ数分のビューを `HStack` で準備しておき，
ページ切り替えボタンで `画面幅 * index` で座標をずらして
無理やり表示させる原始的な実装です。

## おわりに

今回は，ボタンアクションのみでページングさせる実装を考えてみました。
基本的に `PageTabViewStyle` でのスワイプによるページングは，
外すことは少ないと思いますがいい勉強になりました。

画面作るの楽しすぎますねー
ご覧いただきありがとうございました！
