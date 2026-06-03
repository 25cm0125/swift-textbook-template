# 第1章：WebAPIの基本

> 執筆者：山田羽根
> 最終更新：2026-04-17

## この章で学ぶこと

（この章で扱うトピックの概要を2〜3行で書く。自分の言葉で。）

例：この章では、インターネット上のサービス（API）からデータを取得して、アプリ内に表示する方法を学ぶ。具体的にはiTunes Search APIを使って音楽を検索し、その結果をリスト表示するアプリを題材にする。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
// ============================================
// 第1章（基本）：iTunes Search APIで音楽を検索するアプリ
// ============================================
// このアプリは、iTunes Search APIを使って
// 音楽（曲）を検索し、結果をリスト表示します。
// APIキーは不要で、すぐに動かすことができます。
// ============================================

import SwiftUI

// MARK: - データモデル

struct SearchResponse: Codable {
    let results: [Song]
}

struct Song: Codable, Identifiable {
    let trackId: Int
    let trackName: String
    let artistName: String
    let artworkUrl100: String
    let previewUrl: String?

    var id: Int { trackId }
}

// MARK: - メインビュー

struct ContentView: View {
    @State private var songs: [Song] = []
    @State private var searchText: String = ""
    @State private var isLoading: Bool = false

    var body: some View {
        NavigationStack {
            VStack {
                // 検索バー
                HStack {
                    TextField("アーティスト名を入力", text: $searchText)
                        .textFieldStyle(.roundedBorder)

                    Button("検索") {
                        Task {
                            await searchMusic()
                        }
                    }
                    .buttonStyle(.borderedProminent)
                    .disabled(searchText.isEmpty)
                }
                .padding(.horizontal)

                // 検索結果リスト
                if isLoading {
                    ProgressView("検索中...")
                        .padding()
                    Spacer()
                } else if songs.isEmpty {
                    ContentUnavailableView(
                        "曲を検索してみよう",
                        systemImage: "music.note",
                        description: Text("アーティスト名を入力して検索ボタンを押してください")
                    )
                } else {
                    List(songs) { song in
                        SongRow(song: song)
                    }
                }
            }
            .navigationTitle("Music Search")
        }
    }

    // MARK: - API通信

    func searchMusic() async {
        guard let encodedText = searchText.addingPercentEncoding(
            withAllowedCharacters: .urlQueryAllowed
        ) else { return }

        let urlString = "https://itunes.apple.com/search?term=\(encodedText)&media=music&country=jp&limit=25"

        guard let url = URL(string: urlString) else { return }

        isLoading = true

        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            let response = try JSONDecoder().decode(SearchResponse.self, from: data)
            songs = response.results
        } catch {
            print("エラー: \(error.localizedDescription)")
            songs = []
        }

        isLoading = false
    }
}

// MARK: - 曲の行ビュー

struct SongRow: View {
    let song: Song

    var body: some View {
        HStack(spacing: 12) {
            AsyncImage(url: URL(string: song.artworkUrl100)) { image in
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
            } placeholder: {
                Color.gray.opacity(0.3)
            }
            .frame(width: 60, height: 60)
            .clipShape(RoundedRectangle(cornerRadius: 8))

            VStack(alignment: .leading, spacing: 4) {
                Text(song.trackName)
                    .font(.headline)
                    .lineLimit(1)

                Text(song.artistName)
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
            }
        }
        .padding(.vertical, 4)
    }
}

#Preview {
    ContentView()
}


```

**このアプリは何をするものか：**　iTunes Search APIを利用して、入力したアーティスト名から楽曲を検索し、一覧表示するアプリです。


## コードの詳細解説

### データモデル（Codable構造体）

```swift
struct SearchResponse: Codable {
    let results: [Song]
}

struct Song: Codable, Identifiable {
    let trackId: Int
    let trackName: String
    let artistName: String
    let artworkUrl100: String
    let previewUrl: String?

    var id: Int { trackId }
}
```

**何をしているか：**
iTunes APIから返ってくるJSONデータを受け取るための「箱（データ構造）」を定義しています。

**なぜこう書くのか：**
Codable のおかげでネットから届いたデータを、「全自動でアプリ用に整理」してくれるから。

Identifiable のおかげで曲の一つ一つに「名札（ID）」をつけてくれるから、これがあると画面にズラッと並べるときにアプリが「どの曲がどれか」迷わない。

**もしこう書かなかったら：**
手作業でJSONを解読するハメになる（Codableがない場合）
もし : Codable を使わなかったら、インターネットから届いた文字の塊から、1つずつ手作業でデータを取り出す長大なコードを書くことになります。
Codable は、何十行にもなる危険で面倒なコードを たった1つの単語で全自動化 してくれているのです。

画面を作るときに毎回エラーで怒られる（Identifiableがない場合）
もし Song に : Identifiable と var id: Int { trackId } を書かなかったら、データを扱う準備（裏側の処理）はできても、いざ画面に表示しようとした時に毎回面倒な指定が必要になります。

---

### API通信の処理

```swift
// MARK: - API通信

    func searchMusic() async {
        guard let encodedText = searchText.addingPercentEncoding(
            withAllowedCharacters: .urlQueryAllowed
        ) else { return }

        let urlString = "https://itunes.apple.com/search?term=\(encodedText)&media=music&country=jp&limit=25"

        guard let url = URL(string: urlString) else { return }

        isLoading = true

        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            let response = try JSONDecoder().decode(SearchResponse.self, from: data)
            songs = response.results
        } catch {
            print("エラー: \(error.localizedDescription)")
            songs = []
        }

        isLoading = false
    }
```

**何をしているか：**
入力された文字（アーティスト名など）を使ってAppleのサーバーに「この曲を探して！」とリクエストを送り、返ってきたデータ（JSON）をアプリで使える形（[Song]の配列）に変換して保存しています。
同時に、通信中の「くるくるマーク（ロード中）」のオン・オフや、通信に失敗したときのエラー対応もまとめて行っています。

**なぜこう書くのか：**
インターネット通信は「いつ終わるか分からない」し、「失敗するかもしれない」からです。
そのため、async / await という仕組みを使って「通信が終わるまで裏側で待つ（画面を固めない）」ようにし、do / catch を使って「もし失敗してもアプリが落ちないように安全に処理する」という、ユーザーにとって快適でバグの起きにくい書き方をしています。

**もしこう書かなかったら：**
async / await を使わなかったら:
通信が終わるまでの数秒間、画面が完全にフリーズしてしまい、スクロールもボタン操作も一切できなくなります（最悪の場合、OSから「応答なし」と判断されてアプリが落とされます）。

do / catch（エラー処理）を書かなかったら:
地下鉄などで電波が悪い時や、Appleのサーバーが落ちている時に、アプリが突然強制終了（クラッシュ）してしまいます。

日本語のURL変換（addingPercentEncoding）をしなかったら:
英語での検索はできても、「米津玄師」など日本語で検索した瞬間にエラーになり、一切検索ができなくなります（インターネットのURLには日本語をそのまま使えないというルールがあるためです）。

---

### ビューの構成

```swift
// 1. メイン画面の構成（ContentViewの中）
    var body: some View {
        NavigationStack {
            VStack {
                // ▼ 検索バー（横並び）
                HStack {
                    TextField("アーティスト名を入力", text: $searchText)
                        .textFieldStyle(.roundedBorder)

                    Button("検索") {
                        Task {
                            await searchMusic()
                        }
                    }
                    .buttonStyle(.borderedProminent)
                    .disabled(searchText.isEmpty) // 入力が空ならボタンを押せなくする
                }
                .padding(.horizontal)

                // ▼ 検索結果の表示エリア（状態によって出し分ける）
                if isLoading {
                    ProgressView("検索中...")
                        .padding()
                    Spacer()
                } else if songs.isEmpty {
                    ContentUnavailableView(
                        "曲を検索してみよう",
                        systemImage: "music.note",
                        description: Text("アーティスト名を入力して検索ボタンを押してください")
                    )
                } else {
                    List(songs) { song in
                        SongRow(song: song) // ここで2の「行ビュー」を呼び出している
                    }
                }
            }
            .navigationTitle("Music Search")
        }
    }
```

**何をしているか：**
画面のレイアウト（骨組み）の作成: VStack（縦並び）や HStack（横並び）という透明なボックスを組み合わせて、検索バーやリストの位置を決めています。

状態に応じた画面の自動切り替え: 「検索中（isLoading）」「結果が空（songs.isEmpty）」「検索成功」というアプリの今の状態に合わせて、表示する画面を3パターンに自動で出し分けています。

部品の切り出し（コンポーネント化）: リストの1行分のデザインを SongRow という別の部品として独立させ、左側にジャケット画像（AsyncImage）、右側に曲名とアーティスト名（VStack）が並ぶように構成しています。

**なぜこう書くのか：**
「データ ＝ 画面」にするため（宣言的UI）: SwiftUIは「この状態の時は、この画面を表示する」と設計図に宣言しておくだけで、データが変わった瞬間にシステムが勝手に画面を書き換えてくれます。これが現代のアプリ開発で最もバグが起きにくいスマートな書き方だからです。

コードの整理整頓（可読性）: 1行分のデザイン（SongRow）を外に追い出すことで、メイン画面（ContentView）のコードがスッキリし、どこに何が書かれているかが一目で分かるようになります。

心地よいユーザー体験（UX）のため:
画像がネットから届くまでの「身代わりのグレー背景（placeholder）」を用意する
文字が長すぎるときは自動で「...」と省略する（.lineLimit(1)）
検索文字が空のときはボタンを押せなくする（.disabled）

これらすべての親切設計が、この書き方をすることでわずか数行で安全に実現できるからです。

**もしこう書かなかったら：**
画面の「表示バグ」が多発する: 昔の古い書き方のように「ボタンが押されたら、手動でぐるぐるマークを表示して、リストを非表示にして、データが来たらぐるぐるを消して…」という手順をすべて命令文で書くことになります。その結果、「通信が終わったのにぐるぐるが消えずに残り続ける」「前の検索結果の上に新しいデータが重なって表示される」といった画面の矛盾（バグ）が頻発します。

コードが巨大な迷宮（スパゲティコード）になる: 全ての画面構成を1つの場所にダラダラと書き連ねると、階層（{ } の波カッコ）が深くなりすぎて、どこを触れば画像や文字のデザインを変えられるのかが自分でも分からなくなります。

「ガタガタで不親切」なアプリになる: 画像が読み込まれた瞬間に画面のレイアウトが急にガタッと動いて崩れたり、文字が長すぎて画面からはみ出したり、空の状態で検索ボタンが連打されてアプリがフリーズするなど、クオリティの低いアプリになってしまいます。

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`Codable` | JSONデータとSwiftの構造体を相互変換するプロトコル | `struct Song: Codable { ... }` |
| 例：`async/await` | 非同期処理を同期的に書ける構文 | `let data = try await URLSession.shared.data(from: url)` |
| | | |
| | | |
| | | |

## 自分の実験メモ

（模範コードを改変して試したことを書く）

**実験1：**
- やったこと：
- 結果：
- わかったこと：

**実験2：**
- やったこと：
- 結果：
- わかったこと：

## AIに聞いて特に理解が深まった質問 TOP3

1. **質問：**
   **得られた理解：**

2. **質問：**
   **得られた理解：**

3. **質問：**
   **得られた理解：**

## この章のまとめ

（この章で学んだ最も重要なことを、未来の自分が読み返したときに役立つように書く）
