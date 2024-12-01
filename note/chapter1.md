# 第1章 入門
## 新規プロジェクトの作成
npmやyarnよりも高速で効率的なので、pnpmをパッケージマネージャーとして使うことをお勧めします。pnpmがインストールされていない場合は、グローバルにインストールすることができます
```bash
npm install -g pnpm
```

Next.jsアプリを作成するには、ターミナルを開き、プロジェクトを保存したいフォルダにcdして、次のコマンドを実行します
```bash
npx create-next-app@latest nextjs-dashboard --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example" --use-pnpm
```
このコマンドは、Next.jsアプリケーションをセットアップするコマンドラインインターフェイス（CLI）ツールであるcreate-next-appを使用します。上のコマンドでは、--exampleフラグにこのコースのスターターサンプルを指定しています。

因みに、自分の開発環境では下記のような実行結果が表示された。
```
Success! Created nextjs-dashboard at /app/nextjs-dashboard
Inside that directory, you can run several commands:

  pnpm run dev
    Starts the development server.

  pnpm run build
    Builds the app for production.

  pnpm start
    Runs the built app in production mode.

We suggest that you begin by typing:

  cd nextjs-dashboard
  pnpm run devSuccess! Created nextjs-dashboard at /app/nextjs-dashboard
Inside that directory, you can run several commands:

  pnpm run dev
    Starts the development server.

  pnpm run build
    Builds the app for production.

  pnpm start
    Runs the built app in production mode.

We suggest that you begin by typing:

  cd nextjs-dashboard
  pnpm run dev
```
## プロジェクトの探索
ゼロからコードを書くチュートリアルとは異なり、このコースのコードの多くはすでに書かれています。これは、既存のコードベースで作業する可能性が高い、実際の開発現場をよりよく反映しています。

私たちの目標は、すべてのアプリケーションコードを書くことなく、Next.jsの主な機能の学習に集中できるようにすることです。

インストール後、コードエディタでプロジェクトを開き、nextjs-dashboardに移動します。

プロジェクトについて少し調べてみよう。

## フォルダー構造
* /app： アプリケーションのすべてのルート、コンポーネント、ロジックを含みます。
* /app/lib： 再利用可能なユーティリティ関数やデータフェッチ関数など、アプリケーションで使われる関数が含まれます。
* /app/ui： カード、テーブル、フォームなど、アプリケーションのすべてのUIコンポーネントが含まれます。時間を節約するために、これらのコンポーネントはあらかじめスタイル設定されています。
* /public： 画像など、アプリケーションのすべての静的アセットが格納されます。
* 設定ファイル： next.config.jsのような設定ファイルもアプリケーションのルートにあります。これらのファイルのほとんどは、create-next-appを使用して新しいプロジェクトを開始するときに作成され、事前に設定されます。このコースでは、これらのファイルを変更する必要はありません。
これらのフォルダを自由に探索し、コードが行っていることのすべてをまだ理解していなくても心配しないでください。

## プレースホルダ・データ
ユーザー・インタフェースを構築する際、プレースホルダ・データがあると便利です。データベースやAPIがまだ利用できない場合は、次のような方法があります

* JSON形式またはJavaScriptオブジェクトとしてプレースホルダデータを使用する。
* mockAPIのようなサードパーティーのサービスを使う。

このプロジェクトでは、app/lib/placeholder-data.tsにいくつかのプレースホルダー・データを用意した。ファイル内の各JavaScriptオブジェクトは、データベースのテーブルを表します。例えば、invoicesテーブルの場合：
```typescript
const invoices = [
  {
    customer_id: customers[0].id,
    amount: 15795,
    status: 'pending',
    date: '2022-12-06',
  },
  {
    customer_id: customers[1].id,
    amount: 20348,
    status: 'pending',
    date: '2022-11-14',
  },
  // ...
];
```
データベースのセットアップの章では、このデータを使ってデータベースのシード（初期データの投入）を行います。

## TypeScript
また、ほとんどのファイルの接尾辞が.tsまたは.tsxであることに気づくかもしれない。これはプロジェクトがTypeScriptで書かれているからです。私たちは、現代のウェブの状況を反映したコースを作りたかったのです。

必要に応じてTypeScriptのコードスニペットを提供します。

とりあえず、/app/lib/definitions.tsファイルを見てください。ここでは、データベースから返される型を手動で定義しています。例えば、invoicesテーブルには以下の型があります。
```typescript
export type Invoice = {
  id: string;
  customer_id: string;
  amount: number;
  date: string;
  // TypeScriptでは、これを文字列ユニオン型と呼ぶ。
  // これは、「status 」プロパティが、'pending'または'paid'の2つの文字列のうちの1つにしかなり得ないことを意味する。
  status: 'pending' | 'paid';
};
```
TypeScriptを使用することで、コンポーネントやデータベースに間違ったデータ形式を誤って渡さないようにすることができます。例えば、請求書の金額に数字の代わりに文字列を渡すような場合です。

**TypeScript開発者なら：**

データ型を手動で宣言していますが、型の安全性を高めるには、データベーススキーマに基づいて型を自動生成するPrismaやDrizzleをお勧めします。
Next.jsは、プロジェクトがTypeScriptを使用しているかどうかを検出し、必要なパッケージと設定を自動的にインストールします。また、Next.jsにはコードエディタ用のTypeScriptプラグインが付属しており、自動補完と型安全性の向上に役立ちます。

## 開発サーバーの実行
pnpm i を実行して、プロジェクトのパッケージをインストールする。
```bash
pnpm i
```
pnpm devで開発サーバーを起動する。
```bash
pnpm dev
```
pnpm devは、Next.jsの開発サーバーをポート3000で起動します。機能しているか確認してみましょう。

ブラウザで http://localhost:3000 を開いてください。

第1章終了
おめでとうございます！スターターサンプルを使ってNext.jsアプリケーションを作成し、開発サーバーを実行しました。
