---
title: "乗車記録をつけるiOSアプリをリリースしました"
emoji: "🚇"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iOS", "Swift", "SwiftUI", "アプリ"]
published: false
---

## 作ったアプリ

今回リリースしたのは，東京メトロ東西線の乗車記録をつけるアプリです。

　<img src="https://user-images.githubusercontent.com/8732417/150129251-f82bed44-ab99-4a53-b544-83df97159c40.png" width="150"><BR>
　<a href="https://apps.apple.com/jp/app/memories-%E4%B9%97%E8%BB%8A%E8%A8%98%E9%8C%B2%E3%82%A2%E3%83%97%E3%83%AA/id1616337665" style="display: inline-block; overflow: hidden; border-top-left-radius: 13px; border-top-right-radius: 13px; border-bottom-right-radius: 13px; border-bottom-left-radius: 13px; width: 150px; height: 83px;"><img src="https://tools.applemediaservices.com/api/badges/download-on-the-app-store/black/ja-jp?size=250x83&amp;releaseDate=1627603200&h=67f860c8a4424c97a47065fb78d09e10" alt="Download on the App Store" style="border-top-left-radius: 13px; border-top-right-radius: 13px; border-bottom-right-radius: 13px; border-bottom-left-radius: 13px; width: 150px; height: 60px;"></a>


なぜ作った？そのこころは

東京メトロが好きで特に東西線が好きです。
コロナ禍となり利用機会が激減してしまったのですが，
通勤していたころは毎日使っていました。

満員電車の中で東京メトロネットワークの紙をずっと見てて，
駅の順番，接近/車内放送や発車メロディなどから好きになり，
メトロ車両以外には乗らない縛りをつけたりするうちに
下記のスクショのように乗車記録をつけ始めました。

画像

乗車時に逐一テキストベースで記録してきましたが，
私はアプリエンジニアだしアプリで記録付けれるようにして，
ついでに必要な情報も確認できるようにすれば良いのでは？
ということで作り始めました。

開発始めたのは 2020年の7月でした。
リリースまでだいぶかかりました。
主にリモート勤務になって使う機会が減ってのモチベ低下です。
長い人生何が起きるかわからないものですね。

今年はたくさん利用するぞという強い意志を持って
なんとかリリースまでもっていきました。

開発に関して

こだわり実装0: 東西線オンリー？
こだわり実装1: 新規画面は SwiftUI で

こだわり実装2: 運行状況表示
運行状況は API から取得しています。

Url

取得できると分かった時点で絶対やろうと思ったのが電光掲示板みたいな右から左に流れる案内表示でした。

GIF

こちらは UIKit で作っていて AutoLatout の制約とアニメーションを組み合わせて実現しました。

東西線に特化する予定でしたが一緒に 9路線分取れちゃうので一覧表示をモーダルで出せるようにしました。この画面はとてもシンプルなので SwiftUI で作ってます。(ScrollView と LazyVStack)

画像
こだわり実装3: 時刻表の運行会社区別表示

こだわり実装4: 駅マップ


フィードバック
