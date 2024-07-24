---
title: "【SwiftUI】iOS 16.0 で PageTabViewStyle の TabView でクラッシュすることがある"
emoji: "🐛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iOS", "SwiftUI", "TabView", "PageTabViewStyle"]
published: false
---

## はじめに

業務にて OS 依存のバグがあったのでリハビリがてら書く。
表題の通り，**「iOS 16.0 で `PageTabViewStyle` の `TabView` でクラッシュすることがある」**。

iOS 16.0 系だけ発生するらしく，
該当の実機もなかったので，結合試験時にも検知されず，
Firebase Crashlytics のレポートで気付いた形です。

## 発生例

`ForEach` でページングさせたいコンテンツの配列を指定するが，
その配列が空配列をの場合にクラッシュするようである。

表示させたいコンテンツを API を叩いて取得する際に最初は空配列にしておくことは結構ある。

## サンプル

起動後に色名と色を格納した配列を3秒遅延させて表示させるサンプル作ってみた。

GitHub は下記です。
https://github.com/MilanistaDev/iOS16PageTabViewStyleBug

モデルは下記でシンプル。

```swift:Content.swift
import SwiftUI

struct Content {
    let colorName: String
    let color: Color
}

let sampleContents: [Content] = [
    Content(
        colorName: "Red",
        color: .red
    ),
    Content(
        colorName: "Blue",
        color: .blue
    ),
    Content(
        colorName: "Green",
        color: .green
    ),
]
```

ViewModel 側で色配列の処理を行う。非同期を意識して 3s の遅延を入れた。
待つ間 `isLoading` が `true` になる。

```swift:ContentViewModel.swift
import Foundation

@MainActor
final class ContentViewModel: ObservableObject {
    @Published var contents: [Content] = []
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

View 側の実装は下記の通り。
画面表示時に色配列をセットするようにし，
3秒待っている間に ProgressView を表示させる。

```swift:ContentView.swift
import SwiftUI

struct ContentView: View {
    @StateObject private var viewModel = ContentViewModel()
    @State private var selection = 0

    var body: some View {
        TabView(selection: $selection) {
            ForEach(viewModel.contents.indices, id: \.self) { index in
                Text(viewModel.contents[index].colorName)
                    .font(.title)
                    .foregroundStyle(.white)
                    .frame(maxWidth: .infinity, maxHeight: .infinity)
                    .background(viewModel.contents[index].color)
            }
        }
        .tabViewStyle(.page(indexDisplayMode: .never))
        .ignoresSafeArea(.all)
        .overlay(alignment: .center) {
            if viewModel.isLoading {
                ProgressView()
            }
        }
        .onAppear {
            // 色配列を取得したい
            viewModel.onAppear()
        }
    }
}
```

実行させてみると・・・
iOS 16.0 はクラッシュ，iOS 16.4 とかはちゃんと表示される。

|iOS 16.0|iOS 16.4|
|:--:|:--:|
|![ios16_crash](https://github.com/user-attachments/assets/10800b91-6f41-4f6b-9e9e-8f7bb26ffa2c)|![ios16_ok](https://github.com/user-attachments/assets/ae9cbf21-ab87-4644-88c0-bf2a30624e23)|

## 解決策

空配列を避ければ良い。
空配列なら `EmptyView` を充てて，
色配列を取得できたら `TabView` 表示に切り替える。

変更後の例は下記の通りです。

```swift:ContentView.swift
import SwiftUI

struct ContentView: View {
    @StateObject private var viewModel = ContentViewModel()
    @State private var selection = 0

    var body: some View {
        VStack {
            if viewModel.contents.isEmpty {
                EmptyView()
            } else {
                TabView(selection: $selection) {
                    ForEach(viewModel.contents.indices, id: \.self) { index in
                        Text(viewModel.contents[index].colorName)
                            .font(.title)
                            .foregroundStyle(.white)
                            .frame(maxWidth: .infinity, maxHeight: .infinity)
                            .background(viewModel.contents[index].color)
                    }
                }
                .tabViewStyle(.page(indexDisplayMode: .never))
                .ignoresSafeArea(.all)
            }
        }
        .overlay(alignment: .center) {
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

これで落ちなくなる🙆

![ios16_fixed](https://github.com/user-attachments/assets/501dd2e4-6689-4a48-b529-9ec2e2058c33)

## おわりに

ユーザは色々なOSバージョンを使っている。
そして，クラッシュレポートは気づくのに有益，大事。

色々書きたい記事溜まってるので年末にかけて書いていけたらいいな。


## 参考

https://stackoverflow.com/questions/73950003/simple-tabview-is-crashing-in-ios-16
