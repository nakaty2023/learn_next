# 第7章 データの取得
データベースの作成とシードが完了したので、アプリケーションのデータを取得するさまざまな方法について説明し、ダッシュボードの概要ページを作成しましょう。

## この章では...

この章で扱うトピックは以下のとおりです。

* データを取得するためのいくつかのアプローチについて学びます： API、ORM、SQL など。
* Server Components を使用すると、バックエンドのリソースにより安全にアクセスできるようになります。
* ネットワーク・ウォーターフォールとは何か
* JavaScriptパターンを使って並列データフェッチを実装する方法

## データ取得方法の選択
### APIレイヤー
APIは、アプリケーション・コードとデータベースの間の中間層です。APIを使うケースはいくつかある

APIを提供しているサードパーティーのサービスを利用している場合。
クライアントからデータを取得する場合、データベースの秘密がクライアントに公開されないように、サーバー上で動作するAPIレイヤーを用意したい。
Next.jsでは、ルートハンドラを使ってAPIエンドポイントを作成できます。

### データベースクエリ
フルスタックのアプリケーションを作るときには、データベースとやりとりするロジックも書く必要があります。Postgresのようなリレーショナルデータベースの場合、SQLやORMを使ってこれを行うことができます。

データベースのクエリを書かなければならないケースもいくつかある

* APIエンドポイントを作成する際に、データベースとやり取りするロジックを記述する必要があります。
* React Server Components（サーバー上でデータを取得する）を使用している場合は、APIレイヤーをスキップして、データベースの秘密をクライアントに公開するリスクを冒すことなく、データベースに直接問い合わせることができます。

それでは、React Server Componentsについて詳しく説明しよう。

## データ取得にServer Componentsを使用する
デフォルトでは、Next.jsアプリケーションはReact Server Componentsを使用します。Server Componentsを使用したデータ取得は比較的新しいアプローチですが、Server Componentsを使用する利点がいくつかあります

Server Componentsはpromiseをサポートし、データ取得のような非同期タスクのためのシンプルなソリューションを提供します。useEffectやuseState、データ取得ライブラリに手を伸ばすことなく、async/await構文を使用できます。

Server Componentsはサーバー上で実行されるため、高価なデータ取得やロジックをサーバー上に保持し、結果のみをクライアントに送信することができます。

前述のように、Server Componentsはサーバー上で実行されるため、APIレイヤーを追加することなく、データベースに直接問い合わせることができます。

## SQLの使用
ダッシュボードプロジェクトでは、Vercel Postgres SDKとSQLを使用してデータベースクエリを記述します。SQLを使う理由はいくつかあります

SQLはリレーショナルデータベースをクエリするための業界標準です（例えば、ORMはフードの下でSQLを生成します）。
SQLの基本を理解することで、リレーショナル・データベースの基本を理解することができ、その知識を他のツールに応用することができます。
SQLは汎用性が高く、特定のデータを取得して操作することができます。
Vercel Postgres SDKはSQLインジェクションからの保護を提供します。
SQLを使ったことがなくても心配しないでください。

app/lib/data.tsにアクセスしてください。ここでは@vercel/postgresからsql関数をインポートしていることがわかります。この関数を使うと、データベースに問い合わせることができます

どのサーバーコンポーネント内でもsqlを呼び出すことができます。しかし、より簡単にコンポーネントをナビゲートできるように、すべてのデータクエリをdata.tsファイルに保持し、コンポーネントにインポートできるようにしています。

**注意**：第6章で独自のデータベース・プロバイダを使用した場合、あなたのプロバイダで動作するようにデータベース・クエリを更新する必要があります。クエリは/app/lib/data.tsにあります。

## ダッシュボードの概要ページ用にデータをフェッチする
データをフェッチするさまざまな方法を理解したところで、ダッシュボードの概要ページのデータをフェッチしてみましょう。/app/dashboard/page.tsxに移動し、以下のコードを貼り付けて、しばらく時間をかけて探索してください

```tsx
// app/dashboard/page.tsx

import { Card } from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import { lusitana } from '@/app/ui/fonts';

export default async function Page() {
  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
        Dashboard
      </h1>
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        {/* <Card title="Collected" value={totalPaidInvoices} type="collected" /> */}
        {/* <Card title="Pending" value={totalPendingInvoices} type="pending" /> */}
        {/* <Card title="Total Invoices" value={numberOfInvoices} type="invoices" /> */}
        {/* <Card
          title="Total Customers"
          value={numberOfCustomers}
          type="customers"
        /> */}
      </div>
      <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
        {/* <RevenueChart revenue={revenue}  /> */}
        {/* <LatestInvoices latestInvoices={latestInvoices} /> */}
      </div>
    </main>
  );
}
```

上のコードでは
* Pageは非同期コンポーネントである。これにより、awaitを使ってデータを取得することができる。
* また、データを受け取るコンポーネントが3つあります： `<Card>`, `<RevenueChart>`, `<LatestInvoices>` です。これらは現在、アプリケーションのエラーを防ぐためにコメントアウトされています。
