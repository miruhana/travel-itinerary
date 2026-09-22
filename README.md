# 旅程と精算

旅行の日程（タイムライン）と、立て替え払いの精算をまとめて管理する、ブラウザだけで使えるアプリです。
アカウント登録・ログインは不要で、共有リンクを開くだけで旅行メンバー全員が同じ内容を見たり編集したりできます。

- 旅程タブ：日ごとの予定をタイムラインで追加・確認・削除
- メンバータブ：参加者の追加・削除、受け取り方法（現金／PayPay／銀行振込／何でもよい）の設定
- 精算タブ：立て替えの記録（均等割り・個別金額指定）、メンバー別の収支、最終的な送金一覧

要件定義の詳細は [`reference/`](reference/) フォルダの資料を参照してください。

## しくみ

- 静的な1枚の `index.html` を GitHub Pages でホストします。
- 旅程・メンバー・支払いのデータは **Firebase Firestore**（無料枠）に保存し、誰かが編集すると開いている全員の画面に自動的に反映されます。
- ログインの代わりに、URLに含まれるランダムな旅行ID（例：`?trip=abc123...`）で「その旅行のデータ」を区別します。**このURLを知っている人は誰でも閲覧・編集できます**（Googleドキュメントの「リンクを知っている全員が編集可」と同じ考え方です）。そのためURLは旅行メンバーだけに共有してください。

## セットアップ手順（最初に1回だけ）

このアプリを動かすには、無料のFirebaseプロジェクトを1つ作成し、その接続情報を `index.html` に貼り付ける必要があります。

1. [Firebase コンソール](https://console.firebase.google.com/) にGoogleアカウントでログインし、「プロジェクトを追加」で新しいプロジェクトを作成する（プロジェクト名は任意、Google Analyticsは無効でよい）。
2. 左メニューの「構築」→「Firestore Database」→「データベースの作成」を選択。
   - ロケーションは任意（例：`asia-northeast1` 東京）
   - セキュリティルールは「本番環境モード」でよい（後述のルールに書き換えます）
3. 左メニューの「プロジェクトの概要」横の歯車アイコン→「プロジェクトの設定」を開き、「マイアプリ」で「ウェブアプリを追加」（`</>` アイコン）。アプリ名は任意（例：`travel-app`）。Firebase Hosting は使わないのでチェックしなくてOK。
4. 表示された `firebaseConfig` の値（`apiKey` や `projectId` など）をコピーする。
5. このリポジトリの `index.html` をGitHub上で開き、`<script type="module">` の中にある以下の部分を、コピーした値に書き換えて保存する。

   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT.firebaseapp.com",
     projectId: "YOUR_PROJECT",
     storageBucket: "YOUR_PROJECT.appspot.com",
     messagingSenderId: "YOUR_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   ```

6. Firebaseコンソールの「Firestore Database」→「ルール」タブを開き、以下の内容に書き換えて「公開」する。

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /trips/{tripId} {
         allow read, write: if true;
       }
     }
   }
   ```

   > ⚠️ このルールは「ログイン不要で誰でも編集できる」という要件を満たすため、認証なしの読み書きを許可しています。旅行IDはランダムな長い文字列なので他人が偶然アクセスすることはまずありませんが、URLの取り扱いには注意してください（不特定多数に公開しない）。

以上で設定は完了です。GitHub Pagesの公開URLを開くと、自動的に新しい旅行ID付きのURL（例：`https://xxxx.github.io/travel-itinerary/?trip=ab12cd34ef56`）にリダイレクトされます。このURLを旅行メンバーに共有してください。

## 使い方

1. 幹事が公開URLを開く（自動で旅行専用のURLが発行されます）。
2. 画面上部の「共有リンクをコピー」ボタンでURLをコピーし、LINEなどでメンバーに送る。
3. 全員が同じURLを開けば、以後は誰が編集しても全員の画面にリアルタイムで反映されます。

同じ公開URLでも、`?trip=` の値が異なれば別の旅行として扱われます。新しい旅行を作りたいときは、`?trip=` を付けずにトップURLを開き直してください。

## 開発メモ

- フレームワークやビルド手順はなく、`index.html` 単体で完結します。GitHub上で直接編集して保存するだけで、GitHub Pages に自動反映されます。
- データ構造（Member / Event / Expense）や精算ロジックは `reference/` の要件定義書（v1.2）に準拠しています。
- スコープ外（要件定義書 11章）：実際の送金・決済連携、複数通貨、ログイン認証、地図連携は今回は未実装です。
