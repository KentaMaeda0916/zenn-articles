---
title: "【SwiftUI × Firebase】開発環境・本番環境を完全分離する実装ガイド"
emoji: "🔥"
type: "tech"
topics: ["swift", "firebase", "ios", "xcode", "swiftui"]
published: false
---

# はじめに

iOSアプリ開発でFirebaseを利用する際、こんな経験はありませんか？

- 開発中のテストデータが本番環境に混入してしまった
- 本番APIキーを開発中に使ってしまった
- デバッグ時に本番データを誤って削除してしまった

本記事では、同一デバイス上で開発版と本番版を**別アプリとして同時に動作**させ、完全に環境を分離する方法を実践的に解説します。

## この記事で実現できること

✅ 開発版・本番版を同一デバイスに同時インストール
✅ アプリ名・アイコンで視覚的に区別（例: "MyApp DEV" / "MyApp"）
✅ Firebase接続先の自動切り替え
✅ 本番データへの誤操作を完全防止

## 想定読者

- SwiftUIでiOSアプリを開発している方
- Firebaseを利用したバックエンド連携を実装中の方
- 開発環境と本番環境の分離方法を知りたい方
- Xcodeのビルド設定をより深く理解したい方

# 実装の全体像

環境分離を実現するために、以下の3つの要素を組み合わせます：

1. **Firebase側**: 開発用・本番用プロジェクトの分離
2. **Xcode側**: xcconfigファイルによるビルド設定の分岐
3. **コード側**: コンパイル条件によるplistファイルの動的読み込み

---

# Part 1: Firebaseプロジェクトの設定

## 1-1. Firebaseプロジェクトの作成

1. [Firebase Console](https://console.firebase.google.com/)にアクセス
2. 「プロジェクトを追加」をクリック
3. 本番環境用のプロジェクトを作成する。名称: `prod-myapp`（任意の名前）
4. 開発環境用のプロジェクトを作成する。名称: `dev-myapp`（任意の名前）

## 1-2. 各プロジェクトにiOSアプリを登録

### 本番用プロジェクトの設定

1. Firebaseコンソールで本番用プロジェクトを開く
2. プロジェクト概要画面で「アプリを追加」→ iOSアイコンをクリック
3. 以下の情報を入力:
   - **Apple バンドルID**: `com.example.myapp`
   - **アプリのニックネーム**: `MyApp`（任意）
4. 「アプリを登録」をクリック
5. **GoogleService-Info.plistをダウンロード**
6. ダウンロードしたファイルを `GoogleService-Info-release.plist` にリネーム
7. プロジェクトルートディレクトリに配置

### 開発用プロジェクトの設定

1. Firebaseコンソールで開発用プロジェクトを開く
2. プロジェクト概要画面で「アプリを追加」→ iOSアイコンをクリック
3. 以下の情報を入力:
   - **Apple バンドルID**: `com.example.myapp.debug`
   - **アプリのニックネーム**: `MyApp DEV`（任意）
4. 「アプリを登録」をクリック
5. **GoogleService-Info.plistをダウンロード**
6. ダウンロードしたファイルを `GoogleService-Info-debug.plist` にリネーム
7. プロジェクトルートディレクトリに配置

:::message alert
Bundle IDには必ず `.debug` サフィックスを付けてください。
これが開発版と本番版を区別する重要なポイントです。
:::

### ファイル配置確認

プロジェクトルートに以下のファイルが配置されているか確認:

```
/your-project-root/
├── GoogleService-Info-debug.plist   # 開発用
└── GoogleService-Info-release.plist # 本番用
```

---

# Part 2: Xcodeプロジェクトの設定

## 2-1. xcconfigファイルの作成

xcconfigファイルは、ビルド設定を環境別に管理するための設定ファイルです。

### Debug.xcconfig を作成

1. Xcodeでプロジェクトを開く
2. File → New → File...
3. 「Configuration Settings File」を選択
4. ファイル名: `Debug.xcconfig`
5. 保存場所: プロジェクトルート
6. 以下の内容を記述:

```xcconfig:Debug.xcconfig
// Bundle ID設定
PRODUCT_BUNDLE_IDENTIFIER = com.example.myapp.debug

// アプリ表示名
DISPLAY_NAME = MyApp DEV

// コンパイル条件
SWIFT_ACTIVE_COMPILATION_CONDITIONS = DEBUG
```

### Release.xcconfig を作成

1. File → New → File...
2. 「Configuration Settings File」を選択
3. ファイル名: `Release.xcconfig`
4. 保存場所: プロジェクトルート
5. 以下の内容を記述:

```xcconfig:Release.xcconfig
// Bundle ID設定
PRODUCT_BUNDLE_IDENTIFIER = com.example.myapp

// アプリ表示名
DISPLAY_NAME = MyApp

// 最適化設定
SWIFT_OPTIMIZATION_LEVEL = -O
```

:::message
`DISPLAY_NAME` は後ほどInfo.plistから参照します。
`SWIFT_ACTIVE_COMPILATION_CONDITIONS` は、コード内で `#if DEBUG` を使うために必要です。
:::

## 2-2. Configurationsへxcconfigファイルを適用

1. Xcodeのプロジェクトナビゲーターでプロジェクトファイル（青いアイコン）を選択
2. **PROJECT** → プロジェクト名 を選択
3. 「Info」タブを選択
4. 「Configurations」セクションを展開
5. 各Configurationにxcconfigファイルを設定:
   - **Debug**: `Debug.xcconfig` を選択
   - **Release**: `Release.xcconfig` を選択

## 2-3. Target Build Settingsの確認・修正

1. Xcodeのプロジェクトナビゲーターでプロジェクトファイルを選択
2. **TARGETS** → アプリ名 を選択
3. 「Build Settings」タブを選択
4. 検索ボックスで以下を検索し、設定を確認:

### Product Bundle Identifier

- 検索: `PRODUCT_BUNDLE_IDENTIFIER`
- 値が `$(PRODUCT_BUNDLE_IDENTIFIER)` になっているか確認
- なっていない場合は手動で `$(PRODUCT_BUNDLE_IDENTIFIER)` に変更

:::message
`$(変数名)` という記法で、xcconfigファイルで定義した変数を参照できます。
:::

### Product Name

- 検索: `PRODUCT_NAME`
- デフォルトのまま `$(TARGET_NAME)` でOK

## 2-4. Info.plistにDisplay Name設定を追加

1. TARGETS → アプリ名 → 「Info」タブを選択
2. 「Custom iOS Target Properties」セクションを確認
3. `Bundle display name` キーを探す
   - **存在する場合**: 値を `$(DISPLAY_NAME)` に変更
   - **存在しない場合**:
     - 「+」ボタンをクリック
     - Key: `Bundle display name`
     - Type: `String`
     - Value: `$(DISPLAY_NAME)`

これにより、ホーム画面に表示されるアプリ名が環境別に変わります：
- Debug: **MyApp DEV**
- Release: **MyApp**

## 2-5. GoogleService-Info.plistファイルをプロジェクトに追加

### 両plistファイルをプロジェクトに追加

1. Xcodeのプロジェクトナビゲーターで右クリック
2. 「Add Files to "プロジェクト名"...」を選択
3. `GoogleService-Info-debug.plist` を選択
4. オプション設定:
   - ✅ **Copy items if needed**: チェックを外す（既にプロジェクトルートにあるため）
   - ✅ **Add to targets**: アプリ名に**チェックを入れる**
5. 「Add」をクリック

同様の手順で `GoogleService-Info-release.plist` も追加します。

:::message alert
`Copy items if needed` をチェックすると、ファイルが重複してしまう可能性があります。
既にプロジェクトルートに配置済みの場合はチェックを外してください。
:::

### Build Phasesでのリソース確認

1. TARGETS → アプリ名 → 「Build Phases」タブを選択
2. 「Copy Bundle Resources」セクションを展開
3. 以下のファイルが含まれているか確認:
   - `GoogleService-Info-debug.plist`
   - `GoogleService-Info-release.plist`
4. 含まれていない場合:
   - 「+」ボタンをクリック
   - ファイルを選択して追加

## 2-6. Schemeの確認

### Debug/Release Schemeの確認

1. Xcode上部のSchemeセレクターをクリック
2. 「Edit Scheme...」を選択
3. 左側から「Run」を選択
4. 「Info」タブで **Build Configuration** が `Debug` になっているか確認
5. 左側から「Archive」を選択
6. **Build Configuration** が `Release` になっているか確認

:::message
通常はデフォルトで正しく設定されていますが、念のため確認してください。
:::

---

# Part 3: コード実装

## 3-1. Firebase初期化コードの実装

`AppDelegate.swift` または `App.swift` で、環境別にplistファイルを読み込むコードを実装します。

```swift:MyApp.swift
import SwiftUI
import FirebaseCore

@main
struct MyApp: App {
    init() {
        configureFirebase()
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }

    private func configureFirebase() {
        let resorcePath = getResorcePath()
        if let path = Bundle.main.path(forResource: resorcePath, ofType: "plist"),
           let options = FirebaseOptions(contentsOfFile: path) {
            FirebaseApp.configure(options: options)
        } else {
            FirebaseApp.configure()
        }
    }

    private func getResorcePath() -> String {
        #if DEBUG
            return "GoogleService-Info-debug"
        #else
            return "GoogleService-Info-release"
        #endif
    }
}
```

:::message
`#if DEBUG` は、xcconfigファイルで定義した `SWIFT_ACTIVE_COMPILATION_CONDITIONS = DEBUG` によって有効になります。
:::

## 3-2. アイコン差別化（オプション）

開発版と本番版を視覚的に区別しやすくするため、アイコンを差別化できます。

1. `Assets.xcassets` を開く
2. 既存の `AppIcon` を右クリック → Duplicate
3. 新しいアイコンセットの名前を `AppIcon-Debug` に変更
4. 開発版アイコンに「DEV」バッジや色付きオーバーレイを追加
5. Build Settings → **Asset Catalog Compiler** → **Primary App Icon Set Name**:
   - Debug: `AppIcon-Debug`
   - Release: `AppIcon`

これにより、ホーム画面で一目で開発版と本番版を区別できます。

# ビルド
Xcodeのschame設定を切り替えるだけでFirebaseの接続先を変更できるようになりました！
