# 第12章 データの変更
前の章では、URL Search ParamsとNext.js APIを使って検索とページ分割を実装しました。請求書を作成、更新、削除する機能を追加して、請求書ページの作業を続けましょう！

## この章では...

この章で取り上げるトピックは次のとおりです。

* React Server Actionsとは何か、そしてそれを使ってデータを変更する方法。
* フォームとサーバーコンポーネントの扱い方
* ネイティブの formData オブジェクトを使用する際のベストプラクティス。
* revalidatePath API を使用してクライアントキャッシュを再検証する方法。
* 特定の ID を持つ動的なルートセグメントを作成する方法

## サーバーアクションとは？
React Server Actionsを使用すると、非同期コードをサーバー上で直接実行できます。これにより、データを変更するためのAPIエンドポイントを作成する必要がなくなります。その代わりに、サーバー上で実行される非同期関数を記述し、クライアントまたはサーバーコンポーネントから呼び出すことができます。

Webアプリケーションは様々な脅威にさらされやすいため、セキュリティは最優先事項です。そこでサーバーアクションの出番です。サーバーアクションは効果的なセキュリティソリューションを提供し、さまざまなタイプの攻撃から保護し、データを保護し、許可されたアクセスを保証します。Server Actionsは、POSTリクエスト、暗号化されたクロージャ、厳密な入力チェック、エラーメッセージのハッシュ化、ホストの制限などの技術によってこれを実現し、これらすべてが連携してアプリの安全性を大幅に高めます。

## サーバー・アクションでフォームを使う
React では、`<form>` 要素の action 属性を使用してアクションを呼び出すことができます。アクションは自動的に、取り込んだデータを含むネイティブの FormData オブジェクトを受け取ります。

例えば

```typescript
// Server Component
export default function Page() {
  // Action
  async function create(formData: FormData) {
    'use server';

    // Logic to mutate data...
  }

  // Invoke the action using the "action" attribute
  return <form action={create}>...</form>;
}
```

サーバー コンポーネント内でサーバー アクションを呼び出す利点は、プログレッシブ拡張機能です。

フォームは、クライアントでJavaScriptが無効になっていても動作します。

## Next.jsとサーバーアクション
サーバーアクションもNext.jsのキャッシュと深く統合されています。サーバーアクションでフォームが送信されると、アクションを使用してデータを変更できるだけでなく、revalidatePath や revalidateTag などの API を使用して、関連するキャッシュを再検証することもできます。

それがどのように機能するか見てみよう！

### 請求書の作成
新しい請求書を作成する手順は以下の通りです
1. ユーザーの入力を取り込むフォームを作成します。
2. Server Actionを作成し、フォームから呼び出します。
3. Server Action 内で、formData オブジェクトからデータを抽出します。
4. データを検証し、データベースに挿入する準備をします。
5. データを挿入し、エラーを処理します。
6. キャッシュを再検証し、ユーザーを請求書ページにリダイレクトします。

**1. 新しいルートとフォームを作成する**

まず、/invoices フォルダの中に、/create という新しいルートセグメントを page.tsx ファイルとともに追加します

このルートで新しい請求書を作成します。page.tsxファイルの中に、以下のコードを貼り付けてください

```tsx
// dashboard/invoices/create/page.tsx

import Form from '@/app/ui/invoices/create-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import { fetchCustomers } from '@/app/lib/data';

export default async function Page() {
  const customers = await fetchCustomers();

  return (
    <main>
      <Breadcrumbs
        breadcrumbs={[
          { label: 'Invoices', href: '/dashboard/invoices' },
          {
            label: 'Create Invoice',
            href: '/dashboard/invoices/create',
            active: true,
          },
        ]}
      />
      <Form customers={customers} />
    </main>
  );
}
```

あなたのページはサーバーコンポーネントであり、顧客情報を取得して `<Form>` コンポーネントへ渡している。時間短縮のため、`<Form>` コンポーネントは既に用意してある。

`<Form>` コンポーネントへ移動すると、フォームは以下の内容になっていることが確認できる。
* 顧客一覧を表示する `<select>`（ドロップダウン）
* type="number" の `<input>` 要素（金額用）
* type="radio" の `<input>` 要素が2つ（ステータス用）
* type="submit" のボタンが1つ

http://localhost:3000/dashboard/invoices/create にアクセスすると、上記のUIが表示される。

**2. サーバーアクションの作成**

それでは、フォームが送信されたときに呼び出されるサーバーアクションを作成しましょう。

libディレクトリに移動し、actions.tsという名前の新しいファイルを作成します。このファイルの先頭に、React use server ディレクティブを追加します

```typescript
// app/lib/actions.ts

'use server';
```
use server "を追加することで、ファイル内にエクスポートされたすべての関数をサーバーアクションとしてマークします。これらのサーバー関数は、クライアントコンポーネントやサーバーコンポーネントにインポートして使用することができます。

また、サーバーコンポーネントの中に直接サーバーアクションを記述することもできます。しかし、このコースでは、それらをすべて別のファイルに整理しておきます。

actions.tsファイルに、formDataを受け取る新しい非同期関数を作成します

```typescript
// app/lib/actions.ts

'use server';

export async function createInvoice(formData: FormData) {}
```

次に、`<Form>`コンポーネントに、actions.tsファイルからcreateInvoiceをインポートします。`<Form>`要素にaction属性を追加し、createInvoiceアクションを呼び出します。

```tsx
// app/ui/invoices/create-form.tsx

import { customerField } from '@/app/lib/definitions';
import Link from 'next/link';
import {
  CheckIcon,
  ClockIcon,
  CurrencyDollarIcon,
  UserCircleIcon,
} from '@heroicons/react/24/outline';
import { Button } from '@/app/ui/button';
import { createInvoice } from '@/app/lib/actions';

export default function Form({
  customers,
}: {
  customers: customerField[];
}) {
  return (
    <form action={createInvoice}>
      // ...
  )
}
```

**知っておいて損はない**

HTMLでは、action属性にURLを渡します。このURLは、フォームのデータを送信する先（通常はAPIエンドポイント）になります。

しかしReactでは、action属性は特別なpropとみなされます。つまり、Reactはアクションを呼び出すことができるように、action属性の上に構築します。

裏では、Server ActionsはPOST APIエンドポイントを作成します。これが、Server Actionsを使用する際にAPIエンドポイントを手動で作成する必要がない理由です。

**3. formDataからデータを取り出す**

actions.tsファイルに戻り、formDataの値を抽出する必要がありますが、使用できるメソッドがいくつかあります。この例では、.get(name)メソッドを使います。

```typescript
// app/lib/actions.ts

'use server';

export async function createInvoice(formData: FormData) {
  const rawFormData = {
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  };
  // Test it out:
  console.log(rawFormData);
}
```

ヒント: 多数のフィールドを持つフォームを扱う場合は、JavaScript の Object.fromEntries() を使用して entries() メソッドを使用することをお勧めします。たとえば

```javascript
const rawFormData = Object.fromEntries(formData.entries())
```

すべてが正しく接続されていることを確認するために、先に進んでフォームを試してみてください。送信すると、フォームに入力したデータがターミナルに記録されます。

これでデータがオブジェクトの形になったので、作業がとても簡単になります。
