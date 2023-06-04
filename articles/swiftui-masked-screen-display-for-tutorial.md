---
title: "[SwiftUI]チュートリアルやウォークスルーなどでよくあるスポットライトビューを全画面で被せて表示させたい"
emoji: "📱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SwiftUI", "Swift", "iOS", "UIModalPresentationStyle"]
published: true
---

:::message
今回のポイント
* SwiftUI の基本的な書き方の復習
* SwiftUI で UIKit の `UIModalPresentationStyle.overFullScreen` を使えるようにする
* View の一部を切り取ってスポットライトのように表示させる
:::

## はじめに
ありがたいことに，最近は業務でも SwiftUI を触れています。

ある機能の要件定義にて・・・
チュートリアルやウォークスルーとかでよくある，
暗い背景色のビューを画面全体に被せてある部分だけ
スポットが当たったような見せ方したいのだができる？
と言われました。

UIKit では個人開発でも業務でも対応したことがあったのですが
SwiftUI ではまだやったことなくて即答できなかったのが
悔しくて実際にサンプルを作ってみたので備忘録として書きます。

該当画面を今回はスポットライトビューという表現で統一します。

SwiftUI の開発楽しいけど，サポートしている OS によって
簡単に実装できたりできなかったり，一番下のサポート OS に
合わせて作ることが多いので辛いところですね😣


## 今回のサンプル

今回のサンプルの仕様は下記のようにします。

:::message
今回の主な仕様
* TabBar と NavigationBar があるメイン画面の上にスポットライトビューを画面全体に被せる
* スポットライトビューはメイン画面表示後から1秒後に表示させる(アニメーションなし)
* スポットライトビューの画面をタップしたらスポットライト状態を解除する
* スポットライトは 64pt の円とし，画面右下のボタンの上に表示する
* 機能説明の吹き出しをスポットライトの上に表示する
:::

画面は下記のような感じです。

![article_230529_03](https://github.com/MilanistaDev/Zenn-docs/assets/8732417/f3ba5740-400d-444e-8ebc-8babba09a256)


GitHub にコードあげていますので気になる方はご覧ください。

https://github.com/MilanistaDev/MaskedScreenDisplayForTutorial

## 開発環境

今回の開発環境は下記の通りです。

* Xcode 14.3
* iOS 14.0 or later
* SwiftUI 3.0  or later

## 実装

**コードをトグルで非表示にしているので必要に応じて確認してください。**

### メイン画面実装

サクッとメイン画面作ります。

今回は `TabBar` と `NavigationBar` があって，
コンテンツ部分にはよくあるグリッド形式のリストを表示し，
右下にリストにセルを追加できるフローティングアクションボタン(FAB)を実装します。

#### TabView 実装

まず `TabView` を実装するビューを `MainView` として
こちらをルートビューとして扱います。

```swift:MaskedScreenDisplayForTutorialApp.swift
@main
struct MaskedScreenDisplayForTutorialApp: App {
    var body: some Scene {
        WindowGroup {
            MainView() // ルートビューに
        }
    }
}
```

今回はふたつのタブにして一つ目がメイン機能(HOME)，
もうひとつがメニュー機能(MENU:今回は実装なし)とします。
下記のように `enum` でタブ情報を定義しておきます。

:::details [Tap] TabView のタブを扱いやすくするための enum
```swift:TabType.swift
enum TabType: Int, CaseIterable {
    case home
    case menu

    var title: String {
        switch self {
        case .home:
            return "HOME"

        case .menu:
            return "MENU"
        }
    }

    /// 今回は SF Symbols
    var iconName: String {
        switch self {
        case .home:
            return "house"

        case .menu:
            return "menucard"
        }
    }
}
```
:::

`MainView` に `TabView` の実装を書きます。
`tag` は付けておくとプログラムでタブ切り替えをしたいときに便利です。(今回は不要)

```swift:MainView.swift
struct MainView: View {
    @State private var selectedTab: TabType = .home

    var body: some View {
        TabView(selection: $selectedTab) {
            HomeView()
                .iconTabItem(tabType: .home)
                .tag(TabType.home)

            MenuView()
                .iconTabItem(tabType: .menu)
                .tag(TabType.menu)
        }
    }
}

private extension View {
    func iconTabItem(tabType: TabType) -> some View {
        tabItem {
            Label {
                Text(tabType.title)
            } icon: {
                Image(systemName: tabType.iconName)
            }
        }
    }
}
```
ここまでで実行すると下記のようになっています。

![altテキスト](https://github.com/MilanistaDev/Zenn-docs/assets/8732417/be71ea22-c307-41d5-bfa8-addcb2efe68f =350x)

---

#### NavigationView 実装

続いてメイン画面に `NavigationBar` を表示させるため，
`NavigationView` でラップします。
(iOS 16以上で良い場合は `NavigationStack` 使いましょう)

:::details [Tap] NavigationView でラップ
```swift:HomeView.swift
struct HomeView: View {
    var body: some View {
        NavigationView {
            ScrollView {
                VStack {
                    Image(systemName: "globe")
                        .imageScale(.large)
                        .foregroundColor(.accentColor)
                    Text("Hello, world!")
                }
                .padding()

                Spacer()
            }
            .navigationTitle("Home")
        }
    }
}
```
:::

ここまでで実行すると下記のようになっています。

![altテキスト](https://github.com/MilanistaDev/Zenn-docs/assets/8732417/c9ac6d31-9b8c-48e9-a664-84817c618340 =350x)


---

#### リスト部分の実装

グリッド形式にするために今回は `LazyVGrid` を利用します。
今回は画面に対して 2つのアイテムを並べて，Z 形式で並ぶようにします。

https://developer.apple.com/documentation/swiftui/lazyvgrid

コンテンツは以前使った丸ノ内線のサンプルアプリから引っ張ってきます。
下記のような構造体を準備して，サンプルデータを作っておきます。
画像はアプリ側で持っておきます。
(よかったらこちらの記事をご覧ください🙇)

https://qiita.com/MilanistaDev/items/09809b38dc8b23efa9ac

:::details [Tap] 今回のセルのデータ

```swift:ContentItem.swift

struct ContentItem {
    var image: String
    var title: String
    var description: String
}

// 初期表示に用いるサンプルデータ
let sampleContents: [ContentItem] = [
    ContentItem(
        image: "img_news_01",
        title: "2000系爆誕",
        description: "02系に変わり新車両の2000系が登場します。さらに安心安全性が増した車両には電源スペースも。"
    ),
    ContentItem(
        image: "img_news_02",
        title: "サインカーブ！",
        description: "2000系も伝統のサインカーブの存在が光ります。ホームドアがあっても見えるのがいいですね。"
    ),
    ContentItem(
        image: "img_news_03",
        title: "サードレール",
        description: "丸ノ内線は銀座線と同じくサードレール方式(第三軌条方式)。トンネルが小さくてすみました。"
    ),
    ContentItem(
        image: "img_news_00",
        title: "茗荷谷，なんで？",
        description: "各駅は発車メロディに変わってきました。茗荷谷はまだ営団ブザーなのです。住宅街が近いから？"
    ),
]

// ボタンタップで追加されるダミーコンテンツ
let dummyContent = ContentItem(image: "img_news_00", title: "Title", description: "body text")
```
:::

下記のように `LazyVGrid` を実装します。
画面表示時にサンプルデータをセットしています。

```swift:HomeView.swift
struct HomeView: View {

    private let columns: [GridItem] = Array(repeating: .init(.flexible()), count: 2)

    @State private var contents: [ContentItem] = []

    var body: some View {
        NavigationView {
            ScrollView {
                gridView()
            }
            .navigationTitle("Home")
        }
        .onAppear {
            setContents()
        }
    }
}

extension HomeView {
    /// グリッド形式のリスト
    private func gridView() -> some View {
        LazyVGrid(columns: columns, spacing: 12.0) {
            ForEach(0..<contents.count, id: \.self) { index in
                HomeContentRow(content: contents[index])
            }
        }
        .padding(.horizontal, 20.0)
        .padding(.vertical, 20.0)
    }
}

extension HomeView {
    /// サンプルデータをセット
    private func setContents() {
        contents = sampleContents
    }
}
```

ここまでで実行すると下記のようになっています。
いい感じですね👍

![altテキスト](https://github.com/MilanistaDev/Zenn-docs/assets/8732417/cd0e588c-7fd6-45f6-99bd-5f07230fba3c =350x)

---

#### FAB の実装

最後に右下に FAB を表示できるようにします。
FAB の実装は下記の通りで特別なことはしていません。
クロージャで親ビューにタップ時の処理を扱えるようにしています。

:::details [Tap] FAB の実装
```swift:FloatingActionButton.swift
struct FloatingActionButton: View {
    var didTap: (() -> Void)?

    var body: some View {
        Button {
            didTap?()
        } label: {
            Image(systemName: "plus.rectangle.fill.on.rectangle.fill")
                .resizable()
                .scaledToFit()
                .frame(width: 24.0, height: 24.0)
                .foregroundColor(.white)
                .padding(.all, 12.0)
                .background(Color.red)
                .cornerRadius(24.0)
                .shadow(color: .black.opacity(0.3),
                        radius: 5.0,
                        x: 1.0, y: 1.0)
        }
        .padding(.trailing, 20.0)
        .padding(.bottom, 20.0)
    }
}
```
:::

コンテンツはスクロールさせるけど FAB は右下に固定で表示したいので，
ビューを分けて FAB を上に表示させる必要があります。
SwiftUI では ZStack 使うのが一般的かなと思います。
右下に表示させたいので `alignment: .bottomTrailing` を指定します。
`HomeView` に実装していきます。

```diff swift:HomeView.swift
struct HomeView: View {

    private let columns: [GridItem] = Array(repeating: .init(.flexible()), count: 2)

    @State private var contents: [ContentItem] = []

    var body: some View {
        NavigationView {
+           ZStack(alignment: .bottomTrailing) {
+               ScrollView {
+                   gridView()
+               }
+
+               FloatingActionButton {
+                   addDummyContent()
+               }
+           }
            .navigationTitle("Home")
        }
        .onAppear {
            setContents()
        }
    }
}

extension HomeView {
    /// グリッド形式のリスト
    private func gridView() -> some View {
        ...
    }
}

extension HomeView {
    /// サンプルデータをセット
    private func setContents() {
        contents = sampleContents
    }

+   /// タップでダミーコンテンツ追加
+   private func addDummyContent() {
+       contents.append(dummyContent)
+   }
}

```

メイン画面完成です。
ここまでで実行すると下記のようになっています。
FAB をタップするとダミーコンテンツがひとつずつ追加されます。

|初期表示|何個か追加|
|:--:|:--:|
|![](https://github.com/MilanistaDev/Zenn-docs/assets/8732417/b95a3657-2e10-40e6-8d2f-d3a83b6267eb)|![](https://github.com/MilanistaDev/Zenn-docs/assets/8732417/e081bc62-0936-4bc6-876e-f0b71f18c255)|

---

### スポットライトビューの表示実装

スポットライトビューを被せるのがコンテンツ領域だけでよい場合は，
`ZStack` で実装できますが，画面全体となると話が変わってきます。

UIKit では，`UIModalPresentationStyle` に `.overFullScreen` を指定して
present 関数コールすれば遷移元の画面の上に透過可能なビューを表示できます。

```swift:
// UIKit の場合
let vc = SpotlightViewController()
vc.modalPresentationStyle = .overFullScreen
present(vc, animated: false)
```

現時点で SwiftUI では実現できません。
該当する遷移のできるモディファイアが用意されていないからです。
よく使うモディファイアは下記ですが，
対応している `UIModalPresentationStyle` は満たせていないです。

|SwiftUI|UIKit|
|:--|:--|
|sheet|UIModalPresentationStyle.pageSheet|
|fullScreenCover|UIModalPresentationStyle.fullScreen|

なので UIKit の力を借りて遷移用の独自のモディファイアを用意して，
下記のような手順で SwiftUI で使えるようにしていきます。

1. アプリの最前面にある Window を取得
2. その rootViewController に対してスポットライトビューを被せる

#### SwiftUI で overFullScreen での遷移を使えるようにする

アプリの最前面にある画面の取得にあたって，
`UIApplication` の拡張を行います。

`rootViewController` を参照できるように変数を用意して，
`while` ループを使用して最全面の `ViewController` を取得できる関数を実装します。
ついでに `UIScreen.main.bounds` での画面サイズ取得が
deprecated になったので取れるように関数用意しました。
また，Safe Area のサイズも知りたいので取得できるようにしておきます。

```swift:UIApplication+Window.swift
extension UIApplication {
    /// KeyWindow 取得
    private var keyWindow: UIWindow? {
        return UIApplication.shared.connectedScenes
            .compactMap { $0 as? UIWindowScene }
            .first?
            .windows
            .filter { $0.isKeyWindow }
            .first
    }

    /// KeyWindow の rootViewControllerを取得
    private var rootViewController: UIViewController? {
        return keyWindow?.rootViewController
    }

    // 最前面のViewControllerを取得
    func frontMostViewController() -> UIViewController? {
        guard let rootViewController = rootViewController else {
            return nil
        }

        var frontMostViewController = rootViewController

        // ループして最前面の画面を探して返却
        while let presentedViewController = frontMostViewController.presentedViewController {
            frontMostViewController = presentedViewController
        }

        return frontMostViewController
    }

    /// 画面の横幅を取得
    /// - Returns: 画面の横幅
    func screenWidth() -> CGFloat {
        guard let rootViewController = rootViewController else {
            return .zero
        }
        return rootViewController.view.frame.width
    }

    /// 画面の縦幅を取得
    /// - Returns: 画面の縦幅
    func screenHeight() -> CGFloat {
        guard let rootViewController = rootViewController else {
            return .zero
        }
        return rootViewController.view.frame.height
    }

    /// 上部のSafeAreaの高さを取得
    /// - Returns: 上部のSafeAreaの高さ
    func safeAreaTopHeight() -> CGFloat {
        guard let keyWindow = keyWindow else {
            return .zero
        }
        return keyWindow.safeAreaInsets.top
    }

    /// 下部のSafeAreaの高さを取得
    /// - Returns: 下部のSafeAreaの高さ
    func safeAreaBottomHeight() -> CGFloat {
        guard let keyWindow = keyWindow else {
            return .zero
        }
        return keyWindow.safeAreaInsets.bottom
    }
}
```

次に `View` を拡張して遷移用のモディファイアを実装します。
`UIHostingController` 使って UIKit チックに遷移処理を書きます。
表示条件 `isPresented` は SwiftUI の遷移処理であるやつですね。
`isPresented` が `true` になったら遷移処理が動きます。

`UIHostingController` 使う影響で
iOS 14 で被せた画面を閉じられないのと
アニメーションなしでスポットライトビューを解除したいので閉じる関数も追加しました。


```swift:View+Transition.swift
import SwiftUI

extension View {
    /// overFullScreen での遷移を行う
    func presentWithOverFullScreen<Content>(isPresented: Binding<Bool>, @ViewBuilder content: @escaping () -> Content) -> some View where Content: View {
        if isPresented.wrappedValue {
            let viewController = UIHostingController(rootView: content())
            // 遷移元のビューを見せるために背景色を透明にしておく(被せたい画面側で透過する背景色を設定)
            viewController.view.backgroundColor = .clear
            viewController.modalPresentationStyle = .overFullScreen
            // 最前面の画面に対してoverFullScreen での遷移を行う
            UIApplication.shared.frontMostViewController()?.present(viewController, animated: false)
            DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
                isPresented.wrappedValue = false
            }
        }
        return self
    }

    /// 画面を閉じる
    /// - Parameter isAnimated: アニメーションの有無
    func dismissScreen(isAnimated: Bool) {
        UIApplication.shared.frontMostViewController()!.dismiss(animated: isAnimated, completion: nil)
    }

    /**
     @Environment(\.presentationMode) var presentationMode
     // iOS 14 で画面閉じられない
     presentationMode.wrappedValue.dismiss()

     FYI
     https://stackoverflow.com/questions/57190511/dismiss-a-swiftui-view-that-is-contained-in-a-uihostingcontroller
     */
}
```

メインコンテンツの上に投下しているビューを被せる準備ができたので，
実際に背景色 `#000000`，`alpha値 0.5` のビューを被せてみます。
ビュータップで先ほど作っておいた画面を閉じる関数をコールさせています。

```diff swift:HomeView.swift

struct HomeView: View {

    private let columns: [GridItem] = Array(repeating: .init(.flexible()), count: 2)

    @State private var contents: [ContentItem] = []
+   @State private var isPresented = false

    var body: some View {
        NavigationView {
            ZStack(alignment: .bottomTrailing) {
                ScrollView {
                    gridView()
                }

                FloatingActionButton {
                    addDummyContent()
                }
            }
            .navigationTitle("Home")
        }
        .onAppear {
            setContents()
+           displayTransparentBlackScreen()
        }
+       .presentWithOverFullScreen(isPresented: $isPresented) {
+           Color.black
+               .opacity(0.5)
+               .ignoresSafeArea()
+               .onTapGesture {
+                   // 画面タップで元の画面表示
+                   dismissScreen(isAnimated: false)
+               }
+       }
    }
}

extension HomeView {
    /// グリッド形式のリスト
    private func gridView() -> some View {
        ...
    }
}

extension HomeView {
    ...
+    /// 透過した黒い画面を表示
+    private func displayTransparentBlackScreen() {
+        // 1秒後に透過した黒い画面を表示
+        DispatchQueue.main.asyncAfter(deadline: .now() + 1.0) {
+            isPresented = true
+        }
+    }
}
```

ここまでで実行すると下記のようになっています。
あと少し。

![article_230529_01](https://github.com/MilanistaDev/Zenn-docs/assets/8732417/73c7c52d-9258-4f25-84fd-7e83047adf73)

---

#### FAB の部分のスポットライト実装

最後に FAB の部分にスポットライトを当てる実装をしていきます。
具体的には透過させた黒いビューを 64pt の円で切り取るということになります。

SwiftUI では `.mask` モディファイアがあるので利用します。
このサンプルでは iOS 14 に寄せているので deprecated になってるけど古い方使います。

iOS 14 まで
https://developer.apple.com/documentation/swiftui/view/mask(_:)

iOS 15 以降
https://developer.apple.com/documentation/swiftui/view/mask(alignment:_:)

今回切り取る領域の y座標が Safe Area・TabBar の高さがあって
やや複雑なのでしっかり計算してやる必要があります。

切り抜く 64pt の円のビューの実装をします。
表示する座標をどの端末でもボタン部分になるように，
先ほど実装した端末のサイズや Safe Area のサイズを取得して利用します。

```swift:MaskedCircleView.swift
struct MaskedCircleView: View {
    private let padding = 20.0
    private let buttonRadius = 24.0
    private let tabBarHeight = 49.0

    var body: some View {
        Circle()
            .frame(width: 64, height: 64)
            .position(
                x: xPosition(),
                y: yPosition())
            .background(Color.white)
            .compositingGroup()
            .luminanceToAlpha()
    }
}

extension MaskedCircleView {
    private func xPosition() -> CGFloat {
        // 画面幅 - 画面右端とボタンまでのパディング - ボタンの半径
        UIApplication.shared.screenWidth() - padding - buttonRadius
    }

    private func yPosition() -> CGFloat {
        // 画面の高さ - Safe Area下部の高さ - タブバーの高さ - タブバーとボタンまでのパディング - ボタンの半径
        UIApplication.shared.screenHeight() -
        UIApplication.shared.safeAreaBottomHeight() -
        tabBarHeight -
        padding -
        buttonRadius
    }
}
```

`Circle`に適用したモディファイアの下3行部分が大事で，ただ仕様見ても理解しづらいです。
`luminanceToAlpha` が明るい色が透過されるモディファイア，
`compositingGroup` で alpha 0.5 の黒ビューと白い円をくっつけて，
結果的に明るい色の円部分だけ透過されるって感じです。

https://developer.apple.com/documentation/swiftui/image/luminancetoalpha()

https://developer.apple.com/documentation/swiftui/group/compositinggroup()


うまくいきました！

![article_230529_02](https://github.com/MilanistaDev/Zenn-docs/assets/8732417/a8ce3ae5-f2de-484b-8342-e4b7889e4801)


---

#### UIKit だと

UIKit だと Storyboard 使って
切り抜く部分を AutoLayout 指定できるので少し楽にできる印象です。

:::details [Tap] UIKit だったら

Storyboard で切り抜きたい部分に `UIView` をペタッとして，
AutoLayout で制約を与えてやる。

`CAShapeLayer` 使ってやります。

```swift:

final class FugeViewController: UIViewController {
    // 切り抜きたい部分
    @IBOutlet weak var cutOutView: UIView!

    override func viewDidLoad() {
        super.viewDidLoad()
    }

    override func viewDidAppear() {
        super.viewDidAppear(animated)
        // Viewのレイアウト確定してからきり抜く領域をとる
        let rect = self.cutOutView.convert(self.cutOutView.bounds, to: self.view)
        self.view.cutOut(rect, cornerRadius: 26.0)
    }

extension UIView {
    func cutOut(_ rect: CGRect, cornerRadius: CGFloat) {
        let maskLayer = CAShapeLayer()
        maskLayer.frame = self.bounds
        let path = UIBezierPath(rect: self.bounds)
        path.append(UIBezierPath(roundedRect: rect, cornerRadius: cornerRadius))
        maskLayer.fillRule = .evenOdd
        maskLayer.path = path.cgPath
        self.layer.mask = maskLayer
    }
}
```
:::

## 機能説明追加

ただこの画面を表示してもユーザは『？』という反応になると思うので，
チュートリアルとかウォークスルーのコーチマーク的な感じで
この部分はどういう機能，またはどういう表示ですよーって
ユーザに示してあげる必要があると思います。

今回はボタン部分の上に吹き出しを表示させて，
ボタンタップ時の処理について記載しておこうと思います。

被せる画面ごとに実装を切り出して `enum` で被せる画面のタイプとかで
表示を出し分けるとよさそうです。(今回はしません)

被せる画面の実装を `SpotlightFilterView` として切り出して
一緒にボタンの上に吹き出しがくるように実装します。
吹き出しの三角部分の実装は面倒なので画像使ってます。

`ZStack` を使ってスポットライトと黒背景の上に吹き出しを乗っけるだけです。
吹き出しを表示する位置を調整するのも忘れずに。

```swift:SpotlightFilterView.swift
struct SpotlightFilterView: View {
    private let spotlightHeight = 64.0
    private let tabBarHeight = 49.0

    var body: some View {
        ZStack(alignment: .bottomTrailing) {
            // スポットライト部分
            Color.black
                .opacity(0.5)
                .mask(MaskedCircleView())
                .ignoresSafeArea()

            // 吹き出し部分
            VStack(alignment: .trailing, spacing: .zero) {
                Text("ボタンをタップすると，リストのアイテムがひとつ追加されます。")
                    .font(.system(size: 14.0))
                    .padding(.all, 16.0)
                    .background(Color.white)
                    .cornerRadius(4.0)
                    .padding(.horizontal, 16.0)

                Image("speechBubble")
                    .frame(width: 18.0, height: 16.0)
                    .padding(.trailing, 36.0)
            }
            .offset(x: .zero, y: speechBubbleYPosition())
        }
        .onTapGesture {
            dismissScreen(isAnimated: false)
        }
    }
}

extension SpotlightFilterView {
    /// 吹き出しを表示する位置を調整
    private func speechBubbleYPosition() -> CGFloat {
        return -(tabBarHeight + spotlightHeight + 20.0)
    }
}
```

最後に `HomeView` の遷移部分を実装した `SpotlightFilterView` に替えます。


```diff swift:HomeView.swift
struct HomeView: View {
    ...
    var body: some View {
        NavigationView {
            ...
        }
        .onAppear {
            setContents()
            displayTransparentBlackScreen()
        }
        .presentWithOverFullScreen(isPresented: $isPresented) {
-           Color.black
-               .opacity(0.5)
-               .mask(MaskedCircleView())
-               .ignoresSafeArea()
-               .onTapGesture {
-                   // 画面タップで元の画面表示
-                   dismissScreen(isAnimated: false)
-               }
+           SpotlightFilterView()
        }
    }
}
```

動作見てみましょう。
おkおk。うまく表示されました🎉

![article_230529_03](https://github.com/MilanistaDev/Zenn-docs/assets/8732417/f3ba5740-400d-444e-8ebc-8babba09a256)

## おわりに

チュートリアルやウォークスルーなどでよくあるスポットライトビューを全画面で被せて表示させる実装について書きました。

SwiftUI の遷移で `.overFullScreen` を使えるようにすること，
`.mask` モディファイアでスポットライトを実現する部分は勉強になりました。

SwiftUI の API はだいぶ充実してきましたが，できないことはまだ多く，
UIKit の力を借りるところは多々あります。(下位互換ないのも辛い)

要件定義の時点で実現できるかできないか，
自信持てるようにしていかないといけないなーと感じました。

もっと色々なビューの実装やっていかないとなー

乱文でしたが，ご覧いただきありがとうございました。
もっとこうした方がいいよーとかあったらご教示いただけると嬉しいです。

## 参考

SwiftUI 単体で overFullScreen が使えないので
UIKit と連携して遷移できるようにする部分で参考になりました。
https://medium.com/@cuongnguyenhuu/how-to-present-a-screen-with-modalpresentationstyle-in-swiftui-like-uikit-fe9b53e09d72

ビューのマスクとても参考になりました。
https://qiita.com/hcrane/items/7211453164ff6726a73c
