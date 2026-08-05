# 第2章：地図アプリの基本

> 執筆者：山田 羽根
> 最終更新：2026-05-13

## この章で学ぶこと

この章では、SwiftUIでMapKitフレームワークを使い、地図を表示する方法を学びます。
位置情報を持つ構造体の定義や地図上へのMarkerの配置などを学習していきます。


## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
// ============================================
// 第2章（基本）：MapKitで地図を表示するアプリ
// ============================================
// 東京の観光スポットを地図上にマーカーで表示します。
// マーカーをタップすると詳細情報が表示されます。
// ============================================

import SwiftUI
import MapKit

// MARK: - データモデル

struct Landmark: Identifiable {
    let id = UUID()
    let name: String
    let description: String
    let coordinate: CLLocationCoordinate2D
    let category: Category

    enum Category: String, CaseIterable {
        case temple = "寺社"
        case tower = "タワー"
        case park = "公園"

        var iconName: String {
            switch self {
            case .temple: return "building.columns"
            case .tower: return "antenna.radiowaves.left.and.right"
            case .park: return "leaf"
            }
        }

        var color: Color {
            switch self {
            case .temple: return .red
            case .tower: return .blue
            case .park: return .green
            }
        }
    }
}

// MARK: - サンプルデータ

extension Landmark {
    static let sampleData: [Landmark] = [
        Landmark(
            name: "浅草寺",
            description: "東京都内最古の寺院。雷門が有名。",
            coordinate: CLLocationCoordinate2D(latitude: 35.7148, longitude: 139.7967),
            category: .temple
        ),
        Landmark(
            name: "東京タワー",
            description: "1958年に完成した高さ333mの電波塔。",
            coordinate: CLLocationCoordinate2D(latitude: 35.6586, longitude: 139.7454),
            category: .tower
        ),
        Landmark(
            name: "東京スカイツリー",
            description: "高さ634mの世界一高い自立式電波塔。",
            coordinate: CLLocationCoordinate2D(latitude: 35.7101, longitude: 139.8107),
            category: .tower
        ),
        Landmark(
            name: "明治神宮",
            description: "明治天皇と昭憲皇太后を祀る神社。",
            coordinate: CLLocationCoordinate2D(latitude: 35.6764, longitude: 139.6993),
            category: .temple
        ),
        Landmark(
            name: "上野恩賜公園",
            description: "美術館や動物園がある広大な公園。",
            coordinate: CLLocationCoordinate2D(latitude: 35.7146, longitude: 139.7732),
            category: .park
        ),
        Landmark(
            name: "新宿御苑",
            description: "都心にある広さ58.3ヘクタールの庭園。",
            coordinate: CLLocationCoordinate2D(latitude: 35.6852, longitude: 139.7100),
            category: .park
        ),
    ]
}

// MARK: - メインビュー

struct ContentView: View {
    @State private var cameraPosition: MapCameraPosition = .region(
        MKCoordinateRegion(
            center: CLLocationCoordinate2D(latitude: 35.6812, longitude: 139.7671),
            span: MKCoordinateSpan(latitudeDelta: 0.08, longitudeDelta: 0.08)
        )
    )
    @State private var selectedLandmark: Landmark?
    @State private var selectedCategories: Set<Landmark.Category> = Set(Landmark.Category.allCases)

    var filteredLandmarks: [Landmark] {
        Landmark.sampleData.filter { selectedCategories.contains($0.category) }
    }

    var body: some View {
        ZStack(alignment: .bottom) {
            // 地図
            Map(position: $cameraPosition) {
                ForEach(filteredLandmarks) { landmark in
                    Marker(
                        landmark.name,
                        systemImage: landmark.category.iconName,
                        coordinate: landmark.coordinate
                    )
                    .tint(landmark.category.color)
                }
            }
            .mapStyle(.standard(elevation: .realistic))

            // カテゴリフィルター
            VStack(spacing: 8) {
                if let landmark = selectedLandmark {
                    LandmarkCard(landmark: landmark)
                        .transition(.move(edge: .bottom))
                }

                CategoryFilter(selectedCategories: $selectedCategories)
            }
            .padding()
        }
        .onMapCameraChange { context in
            // 地図の操作に応じた処理を追加できる
        }
    }
}

// MARK: - カテゴリフィルター

struct CategoryFilter: View {
    @Binding var selectedCategories: Set<Landmark.Category>

    var body: some View {
        HStack(spacing: 8) {
            ForEach(Landmark.Category.allCases, id: \.self) { category in
                Button {
                    if selectedCategories.contains(category) {
                        selectedCategories.remove(category)
                    } else {
                        selectedCategories.insert(category)
                    }
                } label: {
                    HStack(spacing: 4) {
                        Image(systemName: category.iconName)
                        Text(category.rawValue)
                    }
                    .font(.caption)
                    .padding(.horizontal, 10)
                    .padding(.vertical, 6)
                    .background(
                        selectedCategories.contains(category)
                            ? category.color.opacity(0.2)
                            : Color.gray.opacity(0.1)
                    )
                    .foregroundStyle(
                        selectedCategories.contains(category)
                            ? category.color
                            : .gray
                    )
                    .clipShape(Capsule())
                }
            }
        }
        .padding(8)
        .background(.ultraThinMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 16))
    }
}

// MARK: - ランドマーク詳細カード

struct LandmarkCard: View {
    let landmark: Landmark

    var body: some View {
        VStack(alignment: .leading, spacing: 6) {
            HStack {
                Image(systemName: landmark.category.iconName)
                    .foregroundStyle(landmark.category.color)
                Text(landmark.name)
                    .font(.headline)
                Spacer()
            }
            Text(landmark.description)
                .font(.caption)
                .foregroundStyle(.secondary)
        }
        .padding()
        .background(.ultraThinMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**
東京の主要な観光スポット（寺社、タワー、公園）を地図上にピンとして表示するアプリです。
画面下部にはカテゴリーごとのフィルターボタンがあり、タップすることで表示するスポットの絞り込み（オン・オフ）ができます。


## コードの詳細解説

### データモデル（ランドマーク構造体）

```swift
struct Landmark: Identifiable {
    let id = UUID()
    let name: String
    let description: String
    let coordinate: CLLocationCoordinate2D
    let category: Category

    enum Category: String, CaseIterable {
        case temple = "寺社"
        // ... 
    }
}
```

**何をしているか：**
地図上に表示するスポット（Landmark）のデータ設計図を作っています。
それぞれに固有のID、名前、説明、緯度経度、そして「カテゴリー（寺社・タワー・公園）」を持たせています。カテゴリーには専用のアイコンと色が紐づいています。

**なぜこう書くのか：**
SwiftUIの ForEach という機能は、データを元に、同じ見た目の部品（今回は地図のピン）をたくさん作るのが得意な工場のようなものです。
しかし、工場（システム）は「いま作っているピンが、どのデータのものか」を正確に把握しておく必要があります。

**もしこう書かなかったら：**
（この部分を省略したり変えたりすると何が起きるか。実際に試した結果があればここに書く）

---

### 地図の表示とカメラ制御

```swift
@State private var cameraPosition: MapCameraPosition = .region(
    MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 35.6812, longitude: 139.7671),
        span: MKCoordinateSpan(latitudeDelta: 0.08, longitudeDelta: 0.08)
    )
)

Map(position: $cameraPosition)
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### マーカーの表示

```swift
Map(position: $cameraPosition) {
    ForEach(filteredLandmarks) { landmark in
        Marker(
            landmark.name,
            systemImage: landmark.category.iconName,
            coordinate: landmark.coordinate
        )
        .tint(landmark.category.color)
    }
}
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### フィルター機能

```swift
@State private var selectedCategories: Set<Landmark.Category> = Set(Landmark.Category.allCases)

var filteredLandmarks: [Landmark] {
    Landmark.sampleData.filter { selectedCategories.contains($0.category) }
}
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`Map` | SwiftUIで地図を表示するビューコンポーネント | `Map(position: .constant(.region(region)))` |
| 例：`Marker` | 地図上に位置をマーキングするコンポーネント | `Marker("名前", coordinate: coordinate)` |
| `Annotation` | 標準のピンではなく、自由にデザインしたカスタムビューを地図上に配置するコンポーネント | Annotation("名前", coordinate: coord) { Image(systemName: "star") } |
| `MapCircle` | 指定した座標を中心に、地図上に円（半径をメートル単位で指定）を描画するコンポーネント | MapCircle(center: coord, radius: 1000).foregroundStyle(.blue.opacity(0.3)) |
| `.mapControls` | 地図上にコンパス、縮尺、現在地ボタンなどのApple標準のUIパーツを配置するモディファイア | .mapControls { MapCompass() MapScaleView() } |

## 自分の実験メモ

（模範コードを改変して試したことを書く）

**実験1：**
- やったこと：
- 模範コードの Marker を Annotation に書き換え、中身を Text("🗼") などの絵文字や、画像（Image）を表示するカスタムビューに変更してみた。
- 結果：
- <img width="379" height="778" alt="スクリーンショット 2026-07-29 14 34 05" src="https://github.com/user-attachments/assets/f8a3ef2b-51e2-423d-a210-a6dc3cc1c3fd" /><img width="371" height="784" alt="スクリーンショット 2026-08-05 15 24 41" src="https://github.com/user-attachments/assets/35cc2c1e-38bc-40a0-bdf9-fb9ce5cf8975" />


- わかったこと：
- アプリ独自の世界観を出したい時や、特定のスポットを目立たせたい時、ユーザーのプロフィールアイコンなどを直接地図に置きたい場合は Annotation を使うと自由にカスタマイズできることがわかった。

**実験2：**
- やったこと：
- 地図に MapCircle を追加し、東京タワーの座標を中心に、半径 1000（1km）の半透明の赤い円を描画してみた。
- 結果：
- <img width="375" height="777" alt="スクリーンショット 2026-07-29 14 35 42" src="https://github.com/user-attachments/assets/d245e703-92eb-4271-84ec-b26daff5b3a7" />

- わかったこと：
- 点（Marker）で場所を示すだけでなく、面（MapCircle）を描画することで、「ここから徒歩圏内のエリア」「配達可能なエリア」といった情報をユーザーに視覚的・直感的に伝えられるようになることがわかった。同時に、Map内で複数の異なる要素（ピンと円など）を組み合わせられることも理解できた。

## AIに聞いて特に理解が深まった質問 TOP3

1. **質問：**
   **得られた理解：**

2. **質問：**
   **得られた理解：**

3. **質問：**
   **得られた理解：**

## この章のまとめ

（この章で学んだ最も重要なことを、未来の自分が読み返したときに役立つように書く）
