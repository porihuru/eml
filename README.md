# eml

見積入力（Firebase勉強用 / 1画面完結）

## 概要

このプロジェクトは、Firebase Authentication と Firestore を使った学習用のシンプルな見積入力アプリケーションです。
単一の HTML ファイル（`index.html`）で完結しており、Firebase の基本的な機能を手軽に試すことができます。

## 主な機能

### Authentication（認証）
- **ログイン**: メールアドレスとパスワードでログイン
- **新規登録**: 新規ユーザーアカウントの作成
- **パスワード再設定**: メールアドレス宛にパスワード再設定リンクを送信
- **ログアウト**: 現在のセッションからログアウト

### Firestore（データベース）
- **追加**: 新しい見積データを `quotes` コレクションに追加
- **更新**: 既存の見積データを編集・更新
- **削除**: 見積データを削除
- **一覧表示**: リアルタイム購読（onSnapshot）で自分のデータのみを表示
  - `ownerUid` フィールドでログイン中のユーザーのデータのみをフィルタリング
  - 作成日時の降順（新しい順）で表示

## Firestore データ仕様

`quotes` コレクションには以下のフィールドが含まれます：

| フィールド名 | 型 | 説明 |
|------------|-----|------|
| `title` | string | 見積の件名 |
| `total` | number | 見積の合計金額 |
| `ownerUid` | string | 作成者のユーザーID（Firebase Auth の uid） |
| `createdAt` | timestamp | 作成日時（サーバータイムスタンプ） |
| `updatedAt` | timestamp | 更新日時（サーバータイムスタンプ） |

## セットアップ手順

### 1. Firebase プロジェクトの作成

1. [Firebase Console](https://console.firebase.google.com/) にアクセス
2. 「プロジェクトを追加」をクリックして新しいプロジェクトを作成

### 2. Firebase Authentication の有効化

1. Firebase Console で作成したプロジェクトを開く
2. 左メニューから「Authentication」を選択
3. 「始める」をクリック
4. 「Sign-in method」タブで「メール/パスワード」を有効化

### 3. Firestore の有効化

1. Firebase Console の左メニューから「Firestore Database」を選択
2. 「データベースの作成」をクリック
3. セキュリティルールは「本番環境モード」または「テストモード」を選択
   - **重要**: 本番環境では必ず適切なセキュリティルールを設定してください（後述）

### 4. Firebase 設定の取得と差し替え

1. Firebase Console のプロジェクト設定（⚙️ > プロジェクトの設定）を開く
2. 「マイアプリ」セクションで Web アプリを追加（まだ追加していない場合）
3. 表示される `firebaseConfig` オブジェクトをコピー
4. `index.html` の 116～132 行目あたりの `firebaseConfig` を差し替え

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

### 5. アプリの起動

**ローカルで開く場合:**
- `index.html` をブラウザで直接開く

**静的ホスティングを使う場合:**
- Firebase Hosting や GitHub Pages などの静的ホスティングサービスにデプロイ
- 例: `firebase deploy --only hosting`

## 注意事項

### Firestore セキュリティルールの設定

本番環境で使用する場合は、必ず適切なセキュリティルールを設定してください。
以下は推奨されるルールの例です：

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /quotes/{quoteId} {
      // ログインユーザーのみ自分のデータを読み取り可能
      allow read: if request.auth != null && resource.data.ownerUid == request.auth.uid;
      // ログインユーザーのみ自分のデータとして作成・更新可能
      allow create, update: if request.auth != null && request.resource.data.ownerUid == request.auth.uid;
      // ログインユーザーのみ自分のデータを削除可能
      allow delete: if request.auth != null && resource.data.ownerUid == request.auth.uid;
    }
  }
}
```

### firebaseConfig の取り扱い

- `firebaseConfig` に含まれる `apiKey` は公開されても問題ありませんが、適切なセキュリティルールを設定することが重要です
- セキュリティルールが未設定の場合、誰でもデータベースにアクセスできてしまうため、必ず設定してください
- 公開リポジトリにプッシュする場合は、本番用の `firebaseConfig` を直接コミットしないよう注意してください

## 技術スタック

- **Firebase SDK**: v9.23.0 compat（CDN経由で読み込み）
- **Firebase Authentication**: メール/パスワード認証
- **Firestore**: NoSQL データベース
- **HTML/CSS/JavaScript**: バニラ JavaScript（フレームワーク不要）

## ライセンス

このプロジェクトは学習用途のサンプルです。