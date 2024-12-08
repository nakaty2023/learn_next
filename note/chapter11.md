# 第11章 検索とページネーションの追加
前の章では、ストリーミングによってダッシュボードの初期読み込みパフォーマンスを改善しました。次に /invoices ページに移動して、検索とページ分割を追加する方法を学びましょう！

## この章では...

この章で取り上げるトピックは次のとおりです。

* Next.jsのAPI、useSearchParams、usePathname、useRouterの使い方を学びます。
* URL検索パラメータを使用して、検索とページ分割を実装します。

## 開始コード
dashboard/invoices/page.tsxファイルの中に、以下のコードを貼り付けます

```tsx
// app/dashboard/invoices/page.tsx

import Pagination from '@/app/ui/invoices/pagination';
import Search from '@/app/ui/search';
import Table from '@/app/ui/invoices/table';
import { CreateInvoice } from '@/app/ui/invoices/buttons';
import { lusitana } from '@/app/ui/fonts';
import { InvoicesTableSkeleton } from '@/app/ui/skeletons';
import { Suspense } from 'react';

export default async function Page() {
  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      {/*  <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense> */}
      <div className="mt-5 flex w-full justify-center">
        {/* <Pagination totalPages={totalPages} /> */}
      </div>
    </div>
  );
}
```

このページと、これから扱うコンポーネントに慣れるのに時間をかけましょう

1. `<Search/>`はユーザーが特定の請求書を検索できるようにします。
2. `<Pagination/>`は、ユーザーが請求書のページ間を移動できるようにします。
3. `<Table/>`は請求書を表示します。

検索機能はクライアントとサーバーにまたがります。ユーザーがクライアント上で請求書を検索すると、URLパラメータが更新され、サーバー上でデータが取得され、新しいデータでテーブルがサーバー上で再レンダリングされます。

## なぜURL検索パラメータを使うのか？
上述したように、検索状態を管理するためにURL検索パラメータを使用します。クライアントサイドのステートで検索を行うことに慣れていると、このパターンは新鮮かもしれません。

URLパラメータを使って検索を実装することには、いくつかの利点があります

* ブックマーク可能なURLと共有可能なURL： 検索パラメータはURL内にあるので、ユーザーは検索クエリとフィルタを含むアプリケーションの現在の状態をブックマークして、将来の参照や共有に利用することができます。
* サーバーサイドレンダリングと初期ロード： 初期状態をレンダリングするために、URLパラメータをサーバー上で直接消費することができるため、サーバーレンダリングの処理が容易になります。
* アナリティクスとトラッキング: 検索クエリとフィルタを直接URL内に持つことで、クライアントサイドのロジックを追加することなく、ユーザーの行動をトラッキングしやすくなります。

## 検索機能の追加
次に示すのは、検索機能を実装するために使用する Next.js のクライアントフックです。
* useSearchParams: 現在のURLのパラメータにアクセスできます。たとえば、/dashboard/invoices?page=1&query=pending というURLの場合、{page: '1', query: 'pending'} のような検索パラメータになります。
* usePathname: 現在のURLのパス名を取得できます。たとえば、ルートが /dashboard/invoices の場合、usePathname は '/dashboard/invoices' を返します。
* useRouter: クライアントコンポーネント内でプログラム的にルート間のナビゲーションを可能にします。利用できるメソッドはいくつか存在します。

以下は、実装手順の概要です。
1.	ユーザーの入力を取得する
2.	URLを検索パラメータ付きで更新する
3.	入力フィールドとURLを同期させる
4.	テーブルを検索クエリに応じて更新する

### 1. ユーザーの入力を取得する

`<Search>`コンポーネント（/app/ui/search.tsx）に移動します。以下の点に注目してください。
* "use client": このコンポーネントはクライアントコンポーネントであるため、イベントリスナーやフックを使用できます。
* `<input>`: これは検索入力フィールドです。

handleSearch という新しい関数を作成し、`<input>`要素にonChangeリスナーを追加します。onChangeは、入力値が変わるたびにhandleSearchを呼び出します。

```tsx
// app/ui/search.tsx

'use client';

import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';

export default function Search({ placeholder }: { placeholder: string }) {
  function handleSearch(term: string) {
    console.log(term);
  }

  return (
    <div className="relative flex flex-1 flex-shrink-0">
      <label htmlFor="search" className="sr-only">
        Search
      </label>
      <input
        className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
        placeholder={placeholder}
        onChange={(e) => {
          handleSearch(e.target.value);
        }}
      />
      <MagnifyingGlassIcon className="absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
    </div>
  );
}
```

デベロッパーツールのコンソールを開き、検索フィールドに文字を入力して、正しく動作していることをテストしてください。コンソールに検索語が記録されるはずです。

素晴らしい！ユーザーの検索入力をキャプチャしています。次に、検索語でURLを更新する必要があります。
