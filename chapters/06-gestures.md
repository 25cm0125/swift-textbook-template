# 第6章：ジェスチャー操作

> 執筆者：（氏名）
> 最終更新：YYYY-MM-DD

## この章で学ぶこと

（この章で扱うトピックの概要を2〜3行で書く。自分の言葉で。）

例：この章では、ユーザーの指の動きを検出するジェスチャー認識の方法を学ぶ。タップ・ロングプレス・ドラッグ・拡大縮小・回転など、各ジェスチャーの実装方法を学び、最終的にTinder風のスワイプUIで複数のジェスチャーを組み合わせた実装を題材にする。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
// ============================================
// 第6章（基本）：ジェスチャーで操作するカードアプリ
// ============================================
// タップ、ロングプレス、ドラッグ、ピンチ、回転の
// 各ジェスチャーを実際に体験しながら学びます。
// ============================================

import SwiftUI

// MARK: - メインビュー

struct ContentView: View {
    var body: some View {
        NavigationStack {
            List {
                NavigationLink("タップ & ロングプレス") {
                    TapDemoView()
                }
                NavigationLink("ドラッグ") {
                    DragDemoView()
                }
                NavigationLink("ピンチ（拡大縮小）") {
                    MagnifyDemoView()
                }
                NavigationLink("回転") {
                    RotateDemoView()
                }
                NavigationLink("組み合わせ") {
                    CombinedDemoView()
                }
            }
            .navigationTitle("ジェスチャー体験")
        }
    }
}

// MARK: - タップ & ロングプレス

struct TapDemoView: View {
    @State private var tapCount = 0
    @State private var backgroundColor: Color = .blue
    @State private var isPressed = false

    var body: some View {
        VStack(spacing: 30) {
            Text("タップ回数: \(tapCount)")
                .font(.title)

            // シングルタップ
            RoundedRectangle(cornerRadius: 16)
                .fill(backgroundColor)
                .frame(width: 200, height: 200)
                .overlay {
                    Text("タップしてね")
                        .foregroundStyle(.white)
                        .font(.headline)
                }
                .onTapGesture {
                    tapCount += 1
                    backgroundColor = Color(
                        hue: Double.random(in: 0...1),
                        saturation: 0.7,
                        brightness: 0.9
                    )
                }

            // ロングプレス
            Circle()
                .fill(isPressed ? .green : .orange)
                .frame(width: 120, height: 120)
                .scaleEffect(isPressed ? 1.3 : 1.0)
                .overlay {
                    Text(isPressed ? "成功!" : "長押し")
                        .foregroundStyle(.white)
                        .font(.headline)
                }
                .animation(.spring(duration: 0.3), value: isPressed)
                .onLongPressGesture(minimumDuration: 1.0) {
                    isPressed = true
                    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                        isPressed = false
                    }
                }
        }
        .navigationTitle("タップ & ロングプレス")
    }
}

// MARK: - ドラッグ

struct DragDemoView: View {
    @State private var offset: CGSize = .zero
    @State private var lastOffset: CGSize = .zero

    var body: some View {
        VStack {
            Text("カードをドラッグしてみよう")
                .font(.headline)
                .padding()

            Spacer()

            RoundedRectangle(cornerRadius: 20)
                .fill(
                    LinearGradient(
                        colors: [.purple, .blue],
                        startPoint: .topLeading,
                        endPoint: .bottomTrailing
                    )
                )
                .frame(width: 200, height: 280)
                .shadow(radius: 8)
                .overlay {
                    VStack {
                        Image(systemName: "hand.draw")
                            .font(.system(size: 40))
                        Text("ドラッグ")
                            .font(.title3)
                    }
                    .foregroundStyle(.white)
                }
                .offset(offset)
                .gesture(
                    DragGesture()
                        .onChanged { value in
                            offset = CGSize(
                                width: lastOffset.width + value.translation.width,
                                height: lastOffset.height + value.translation.height
                            )
                        }
                        .onEnded { _ in
                            lastOffset = offset
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    offset = .zero
                    lastOffset = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("ドラッグ")
    }
}

// MARK: - ピンチ（拡大縮小）

struct MagnifyDemoView: View {
    @State private var scale: CGFloat = 1.0
    @State private var lastScale: CGFloat = 1.0

    var body: some View {
        VStack {
            Text("ピンチで拡大縮小")
                .font(.headline)
                .padding()

            Text(String(format: "倍率: %.1fx", scale))
                .font(.caption)
                .foregroundStyle(.secondary)

            Spacer()

            Image(systemName: "star.fill")
                .font(.system(size: 100))
                .foregroundStyle(.yellow)
                .scaleEffect(scale)
                .gesture(
                    MagnifyGesture()
                        .onChanged { value in
                            scale = lastScale * value.magnification
                        }
                        .onEnded { _ in
                            lastScale = scale
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    scale = 1.0
                    lastScale = 1.0
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("ピンチ")
    }
}

// MARK: - 回転

struct RotateDemoView: View {
    @State private var angle: Angle = .zero
    @State private var lastAngle: Angle = .zero

    var body: some View {
        VStack {
            Text("2本指で回転")
                .font(.headline)
                .padding()

            Text(String(format: "角度: %.0f°", angle.degrees))
                .font(.caption)
                .foregroundStyle(.secondary)

            Spacer()

            Image(systemName: "arrow.up")
                .font(.system(size: 80))
                .foregroundStyle(.red)
                .rotationEffect(angle)
                .gesture(
                    RotateGesture()
                        .onChanged { value in
                            angle = lastAngle + value.rotation
                        }
                        .onEnded { _ in
                            lastAngle = angle
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    angle = .zero
                    lastAngle = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("回転")
    }
}

// MARK: - 組み合わせ

struct CombinedDemoView: View {
    @State private var offset: CGSize = .zero
    @State private var lastOffset: CGSize = .zero
    @State private var scale: CGFloat = 1.0
    @State private var lastScale: CGFloat = 1.0
    @State private var angle: Angle = .zero
    @State private var lastAngle: Angle = .zero

    var body: some View {
        VStack {
            Text("ドラッグ・ピンチ・回転を同時に")
                .font(.headline)
                .padding()

            Spacer()

            Image(systemName: "photo.artframe")
                .font(.system(size: 120))
                .foregroundStyle(.indigo)
                .scaleEffect(scale)
                .rotationEffect(angle)
                .offset(offset)
                .gesture(
                    DragGesture()
                        .onChanged { value in
                            offset = CGSize(
                                width: lastOffset.width + value.translation.width,
                                height: lastOffset.height + value.translation.height
                            )
                        }
                        .onEnded { _ in
                            lastOffset = offset
                        }
                )
                .gesture(
                    MagnifyGesture()
                        .onChanged { value in
                            scale = lastScale * value.magnification
                        }
                        .onEnded { _ in
                            lastScale = scale
                        }
                )
                .gesture(
                    RotateGesture()
                        .onChanged { value in
                            angle = lastAngle + value.rotation
                        }
                        .onEnded { _ in
                            lastAngle = angle
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    offset = .zero
                    lastOffset = .zero
                    scale = 1.0
                    lastScale = 1.0
                    angle = .zero
                    lastAngle = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("組み合わせ")
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**

このアプリは、スマートフォン特有の直感的なタッチ操作（ジェスチャー）を実際に画面上で試しながら学べる「ジェスチャー体験アプリ」です。

## コードの詳細解説

### 基本ジェスチャー（タップ、ロングプレス）

```swift
// シングルタップ
RoundedRectangle(cornerRadius: 16)
    // ...見た目の設定...
    .onTapGesture {
        tapCount += 1
        backgroundColor = Color(
            hue: Double.random(in: 0...1),
            saturation: 0.7,
            brightness: 0.9
        )
    }

// ロングプレス
Circle()
    // ...見た目の設定...
    .onLongPressGesture(minimumDuration: 1.0) {
        isPressed = true
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
            isPressed = false
        }
    }
```

**何をしているか：**

四角形（RoundedRectangle）に .onTapGesture を付け、タップされたらカウントを増やして色をランダムに変更しています。
円（Circle）には .onLongPressGesture を付け、minimumDuration: 1.0（1秒間）押し続けられた時に状態を「成功（isPressed = true）」にし、その1秒後に元の状態に戻しています。

**なぜこう書くのか：**

通常、タップ操作には Button を使いますが、図形や画像など「標準のボタンではないUI要素」にタッチ操作の機能を持たせたい場合には、これらのジェスチャーモディファイアを使うのがSwiftUIにおける最もシンプルで標準的な方法だからです。

**もしこう書かなかったら：**

ロングプレスで minimumDuration: 1.0 を指定しなかった場合、デフォルトの非常に短い時間（約0.5秒）で反応してしまい、ユーザーからすると「ただゆっくりタップしただけなのに長押しと判定された」と誤操作を招きやすくなります。

---

### ドラッグジェスチャーとオフセット管理

```swift
@State private var offset: CGSize = .zero
@State private var lastOffset: CGSize = .zero

// ...
RoundedRectangle(cornerRadius: 20)
    .offset(offset)
    .gesture(
        DragGesture()
            .onChanged { value in
                offset = CGSize(
                    width: lastOffset.width + value.translation.width,
                    height: lastOffset.height + value.translation.height
                )
            }
            .onEnded { _ in
                lastOffset = offset
            }
    )
```

**何をしているか：**

ユーザーが指を動かしている最中（.onChanged）に、移動量（value.translation）をリアルタイムに取得し、図形の位置（offset）を更新しています。指を離した時（.onEnded）に、その時点の最終的な位置を lastOffset に保存（記憶）しています。

**なぜこう書くのか：**

value.translation は、「画面に指を置いた開始地点から、どれだけ動かしたか」という相対的な距離しか教えてくれません。そのため、前回の終了位置（lastOffset）を基準にして移動量を足し合わせないと、正しい現在位置が計算できないからです。

**もしこう書かなかったら：**

lastOffset を使わずに offset = value.translation （移動した分だけをそのままオフセットにする）と書いた場合、1回目のドラッグはうまくいきます。しかし、指を離して2回目のドラッグを始めようと画面に触れた瞬間、図形が最初の初期位置に一瞬でワープ（瞬間移動）してしまいます。 前回の位置情報がリセットされてしまうためです。

---

### 拡大縮小と回転

```swift
@State private var scale: CGFloat = 1.0
@State private var lastScale: CGFloat = 1.0

// ...
Image(systemName: "star.fill")
    .scaleEffect(scale)
    .gesture(
        MagnifyGesture()
            .onChanged { value in
                scale = lastScale * value.magnification
            }
            .onEnded { _ in
                lastScale = scale
            }
    )
```

**何をしているか：**

2本指でピンチイン・ピンチアウトするジェスチャー（MagnifyGesture）を検知し、指の動きによる倍率（value.magnification）を、前回までの倍率（lastScale）に掛け合わせて現在の scale を更新しています。回転（RotateGesture）も同様に、前回の角度に足し合わせています。

**なぜこう書くのか：**

ドラッグの lastOffset と全く同じ理由です。ジェスチャーが新しく始まるたびに倍率や角度の計算は基準値（1.0や0度）からスタートするため、「前回どこまで拡大したか・どこまで回したか」を記憶しておく変数（lastScale, lastAngle）が必須になります。足し算ではなく掛け算（*）にしているのは、倍率（スケール）の計算だからです。

**もしこう書かなかったら：**

2回目の拡大をしようと指を置いた瞬間、星のサイズが等倍（1.0）に戻ってしまいます。連続して少しずつ拡大したり回転させたりすることができなくなります。

---

### ジェスチャーの組み合わせとアニメーション

```swift
Image(systemName: "photo.artframe")
    .scaleEffect(scale)
    .rotationEffect(angle)
    .offset(offset)
    .gesture(DragGesture()...)
    .gesture(MagnifyGesture()...)
    .gesture(RotateGesture()...)

// リセットボタン
Button("リセット") {
    withAnimation(.spring) {
        offset = .zero
        lastOffset = .zero
        scale = 1.0
        lastScale = 1.0
        angle = .zero
        lastAngle = .zero
    }
}
```

**何をしているか：**

1つの画像に対して .gesture() を3つ連続で付け足すことで、ドラッグ・拡大縮小・回転を同時に受け付けるようにしています。
また、「リセット」ボタンが押されたとき、withAnimation ブロックの中で全ての状態変数を初期値（.zero や 1.0）に戻しています。

**なぜこう書くのか：**

SwiftUIでは、ジェスチャーをモディファイアとして直列に並べるだけで、OS側がよしなに複合的な操作（マルチタッチ）として処理してくれます。
また、状態を元に戻す処理を withAnimation で囲むだけで、現在の変則的な位置や角度・大きさから、初期状態へ向かって自動的にアニメーション（この場合はバネのような弾力のある .spring アニメーション）が計算されて滑らかに戻るからです。

**もしこう書かなかったら：**

withAnimation を付けずに変数のリセットだけを行った場合、ボタンを押した瞬間にパッと一瞬で初期状態に切り替わってしまい、味気なく、ユーザーからすると「何が起きたのか」少し把握しづらい動きになってしまいます。アニメーションを挟むことで、UIに連続性が生まれます。

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`DragGesture` | ドラッグジェスチャーを認識するジェスチャーレコグナイザー | `.gesture(DragGesture().onChanged { ... })` |
| 例：`MagnificationGesture` | ピンチジェスチャーで拡大・縮小を認識 | `.gesture(MagnificationGesture().onChanged { scale in ... })` |
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
