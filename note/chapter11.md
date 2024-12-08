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

