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

## 請求書の作成
新しい請求書を作成する手順は以下の通りです
1. ユーザーの入力を取り込むフォームを作成します。
2. Server Actionを作成し、フォームから呼び出します。
3. Server Action 内で、formData オブジェクトからデータを抽出します。
4. データを検証し、データベースに挿入する準備をします。
5. データを挿入し、エラーを処理します。
6. キャッシュを再検証し、ユーザーを請求書ページにリダイレクトします。

### 1. 新しいルートとフォームを作成する
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

### 2. サーバーアクションの作成
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

#### 知っておいて損はない
HTMLでは、action属性にURLを渡します。このURLは、フォームのデータを送信する先（通常はAPIエンドポイント）になります。

しかしReactでは、action属性は特別なpropとみなされます。つまり、Reactはアクションを呼び出すことができるように、action属性の上に構築します。

裏では、Server ActionsはPOST APIエンドポイントを作成します。これが、Server Actionsを使用する際にAPIエンドポイントを手動で作成する必要がない理由です。

### 3. formDataからデータを取り出す
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

### 4. データの検証と準備
フォームデータをデータベースに送信する前に、正しいフォーマットと正しいデータ型であることを確認します。このコースの最初のほうで、請求書テーブルが次のような形式のデータを想定していることを思い出してください

```typescript
export type Invoice = {
  id: string; // Will be created on the database
  customer_id: string;
  amount: number; // Stored in cents
  status: 'pending' | 'paid';
  date: string;
};
```
今のところ、フォームから得られるのは customer_id、金額、ステータスだけです。

#### 型の検証と強制
フォームからのデータがデータベースで期待される型と一致していることを検証することは重要です。例えば、アクションの中にconsole.logを追加するとします

```typescript
console.log(typeof rawFormData.amount);
```

amountが数値型ではなく文字列型であることにお気づきだろう。これは、type="number" の入力要素は、実際には数値ではなく文字列を返すからです！

型検証を処理するには、いくつかの選択肢があります。手作業で型を検証することもできますが、型検証ライブラリを使えば時間と労力を節約できます。この例では、このタスクを簡略化できるTypeScriptファーストの検証ライブラリであるZodを使用します。

actions.tsファイルでZodをインポートし、フォームオブジェクトの形に合ったスキーマを定義します。このスキーマは、データベースに保存する前にformDataを検証します。

```typescript
// app/lib/actions.ts

'use server';

import { z } from 'zod';

const FormSchema = z.object({
  id: z.string(),
  customerId: z.string(),
  amount: z.coerce.number(),
  status: z.enum(['pending', 'paid']),
  date: z.string(),
});

const CreateInvoice = FormSchema.omit({ id: true, date: true });

export async function createInvoice(formData: FormData) {
  // ...
}
```

金額フィールドは特に、文字列から数値に強制（変更）されるように設定されており、同時にその型も検証されます。

その後、rawFormDataをCreateInvoiceに渡して型を検証することができます

```typescript
// app/lib/actions.ts

// ...
export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
}
```

#### セント単位での値の保存
JavaScriptの浮動小数点エラーを排除し、より高い精度を確保するために、データベース内の金銭的な値をセントで格納することは、通常良い習慣です。

金額をセントに変換してみましょう

```typescript
// app/lib/actions.ts

// ...
export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
  const amountInCents = amount * 100;
}
```

#### 新しい日付の作成
最後に、請求書の作成日として「YYYY-MM-DD」のフォーマットで新しい日付を作成してみましょう

```typescript
// app/lib/actions.ts

// ...
export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split('T')[0];
}
```

### 5. データベースにデータを挿入する
データベースに必要な値がすべて揃ったので、新しい請求書をデータベースに挿入するSQLクエリを作成し、変数を渡します

```typescript
// app/lib/actions.ts

import { z } from 'zod';
import { sql } from '@vercel/postgres';

// ...

export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split('T')[0];

  await sql`
    INSERT INTO invoices (customer_id, amount, status, date)
    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
  `;
}
```

今はまだ、エラーを処理していない。次の章で処理しよう。とりあえず、次のステップに進みましょう。

### 6. 再検証とリダイレクト
Next.jsには、ルートセグメントをユーザーのブラウザに一時的に保存するクライアントサイドルーターキャッシュがあります。プリフェッチとともに、このキャッシュはサーバーへのリクエスト数を減らしながら、ユーザーがルート間をすばやく移動できるようにします。

請求書ルートに表示されるデータを更新するので、このキャッシュをクリアして、サーバーへの新しいリクエストをトリガーします。Next.jsのrevalidatePath関数でこれを行うことができます

```typescript
// app/lib/actions.ts

'use server';

import { z } from 'zod';
import { sql } from '@vercel/postgres';
import { revalidatePath } from 'next/cache';

// ...

export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split('T')[0];

  await sql`
    INSERT INTO invoices (customer_id, amount, status, date)
    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
  `;

  revalidatePath('/dashboard/invoices');
}
```

データベースが更新されると、/dashboard/invoicesのパスが再検証され、サーバーから新しいデータが取得されます。

この時点で、ユーザーを /dashboard/invoices ページにリダイレクトします。Next.jsのredirect関数を使用します。

```typescript
// app/lib/actions.ts

'use server';

import { z } from 'zod';
import { sql } from '@vercel/postgres';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

// ...

export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split('T')[0];

  await sql`
    INSERT INTO invoices (customer_id, amount, status, date)
    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
  `;

  revalidatePath('/dashboard/invoices');
  redirect('/dashboard/invoices');
}
```

おめでとうございます！最初のサーバーアクションを実装しました。新しい請求書を追加してテストしてみてください
1. 送信すると、/dashboard/invoicesルートにリダイレクトされます。
2. テーブルの一番上に新しい請求書が表示されるはずです。

## 請求書の更新
請求書の更新フォームは請求書の作成フォームと似ていますが、データベースのレコードを更新するために請求書IDを渡す必要がある点が異なります。請求書IDを取得し、渡す方法を見てみましょう。

請求書を更新する手順は以下のとおりです

1. 請求書 ID を持つ新しい動的ルートセグメントを作成します。
2. ページパラメータから請求書IDを読み取ります。
3. データベースから特定の請求書を取得します。
4. 請求書データをフォームに事前に入力します。
5. データベースの請求書データを更新します。

### 1. 請求書IDでダイナミックルートセグメントを作成する
Next.jsでは、正確なセグメント名がわからず、データに基づいてルートを作成したい場合に、ダイナミックルートセグメントを作成できます。たとえば、ブログ記事のタイトルや商品ページなどです。フォルダの名前を角括弧で囲むことで、動的なルートセグメントを作成できます。例えば、[id]、[post]、[slug]などです。

invoicesフォルダに、[id]という新しいダイナミックルートを作成し、editという新しいルートとpage.tsxファイルを作成します。ファイル構造は次のようになります

![ファイル構造](./images/image18.png)

 `<Table>`コンポーネントの中に、テーブルのレコードから請求書のIDを受け取る`<UpdateInvoice />`ボタンがあることに注目してください。

```tsx
// app/ui/invoices/table.tsx

export default async function InvoicesTable({
  query,
  currentPage,
}: {
  query: string;
  currentPage: number;
}) {
  return (
    // ...
    <td className="flex justify-end gap-2 whitespace-nowrap px-6 py-4 text-sm">
      <UpdateInvoice id={invoice.id} />
      <DeleteInvoice id={invoice.id} />
    </td>
    // ...
  );
}
```

`<UpdateInvoice />`コンポーネントに移動し、idプロパティを受け入れるようにリンクのhrefを更新します。動的なルートセグメントにリンクするためにテンプレートリテラルを使うことができます

```tsx
// app/ui/invoices/buttons.tsx

import { PencilIcon, PlusIcon, TrashIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';

// ...

export function UpdateInvoice({ id }: { id: string }) {
  return (
    <Link
      href={`/dashboard/invoices/${id}/edit`}
      className="rounded-md border p-2 hover:bg-gray-100"
    >
      <PencilIcon className="w-5" />
    </Link>
  );
}
```

### 2. ページパラメーターから請求書IDを読み取る
`<Page>`コンポーネントに戻り、以下のコードを貼り付ける

```tsx
// app/dashboard/invoices/[id]/edit/page.tsx

import Form from '@/app/ui/invoices/edit-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import { fetchCustomers } from '@/app/lib/data';

export default async function Page() {
  return (
    <main>
      <Breadcrumbs
        breadcrumbs={[
          { label: 'Invoices', href: '/dashboard/invoices' },
          {
            label: 'Edit Invoice',
            href: `/dashboard/invoices/${id}/edit`,
            active: true,
          },
        ]}
      />
      <Form invoice={invoice} customers={customers} />
    </main>
  );
}
```

（edit-form.tsxファイルから）別のフォームをインポートすることを除いて、`/create` invoiceページと似ていることに注目してください。このフォームには、顧客の名前、請求書の金額、ステータスのdefaultValueがあらかじめ入力されているはずです。フォームフィールドに事前に入力するには、idを使用して特定の請求書を取得する必要があります。

searchParamsに加えて、ページコンポーネントはparamsと呼ばれるプロップも受け付けます。propを受け取るように`<Page>`コンポーネントを更新してください

```tsx
// app/dashboard/invoices/[id]/edit/page.tsx

import Form from '@/app/ui/invoices/edit-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import { fetchCustomers } from '@/app/lib/data';

export default async function Page(props: { params: Promise<{ id: string }> }) {
  const params = await props.params;
  const id = params.id;
  // ...
}
```

### 3. 特定の請求書を取得する
次に
* fetchInvoiceByIdという新しい関数をインポートし、引数としてidを渡します。
* fetchCustomersをインポートして、ドロップダウン用の顧客名を取得します。

Promise.allを使用すると、請求書と顧客の両方を並行して取得できます

```tsx
// dashboard/invoices/[id]/edit/page.tsx

import Form from '@/app/ui/invoices/edit-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import { fetchInvoiceById, fetchCustomers } from '@/app/lib/data';

export default async function Page(props: { params: Promise<{ id: string }> }) {
  const params = await props.params;
  const id = params.id;
  const [invoice, customers] = await Promise.all([
    fetchInvoiceById(id),
    fetchCustomers(),
  ]);
  // ...
}
```

invoiceが未定義の可能性があるため、ターミナルでinvoiceプロップに対して一時的なTSエラーが表示されます。エラー処理を追加する次の章で解決します。

素晴らしい！さて、すべてが正しく配線されていることをテストしてください。http://localhost:3000/dashboard/invoices にアクセスし、鉛筆のアイコンをクリックして請求書を編集します。ナビゲーションの後、請求書の詳細があらかじめ入力されたフォームが表示されるはずです

![請求書の編集ページ](./images/image19.png)

URLも以下のようにidで更新する必要がある： http://localhost:3000/dashboard/invoice/uuid/edit

#### UUID vs. 自動インクリメント・キー

私たちはキーのインクリメント（1、2、3など）の代わりにUUIDを使用しています。このためURLは長くなりますが、UUIDはIDの衝突のリスクを排除し、グローバルに一意であり、列挙攻撃のリスクを軽減します。

しかし、よりすっきりとしたURLを好むのであれば、自動インクリメントのキーを使う方がよいでしょう。

### 4. サーバーアクションにidを渡す
最後に、Server Actionにidを渡して、データベースの正しいレコードを更新できるようにします。このようにidを引数として渡すことはできません

```tsx
// app/ui/invoices/edit-form.tsx

// Passing an id as argument won't work
<form action={updateInvoice(id)}>
```

代わりに、JS bindを使ってidをServer Actionに渡すことができます。これにより、Server Actionに渡される値はすべてエンコードされます。

```tsx
// app/ui/invoices/edit-form.tsx

// ...
import { updateInvoice } from '@/app/lib/actions';

export default function EditInvoiceForm({
  invoice,
  customers,
}: {
  invoice: InvoiceForm;
  customers: CustomerField[];
}) {
  const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);

  return <form action={updateInvoiceWithId}></form>;
}
```

注：フォームに隠し入力フィールドを使用することもできます（例：`<input type=「hidden」 name=「id」 value={invoice.id} />`）。ただし、値はHTMLソースにフルテキストとして表示されるので、IDのような機密データには不向きです。

次に、actions.tsファイルに、updateInvoiceという新しいアクションを作成します

```typescript
// app/lib/actions.ts

// Use Zod to update the expected types
const UpdateInvoice = FormSchema.omit({ id: true, date: true });

// ...

export async function updateInvoice(id: string, formData: FormData) {
  const { customerId, amount, status } = UpdateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });

  const amountInCents = amount * 100;

  await sql`
    UPDATE invoices
    SET customer_id = ${customerId}, amount = ${amountInCents}, status = ${status}
    WHERE id = ${id}
  `;

  revalidatePath('/dashboard/invoices');
  redirect('/dashboard/invoices');
}
```
createInvoiceアクションと同様です

1. formDataからデータを抽出する。
2. Zodで型を検証します。
3. 金額をセントに変換します。
4. 変数をSQLクエリに渡します。
5. revalidatePath をコールしてクライアントのキャッシュをクリアし、 新しいサーバリクエストを作成します。
6. redirectを呼び出し、ユーザーを請求書のページにリダイレクトします。

請求書を編集してテストしてみましょう。フォームを送信すると、請求書のページにリダイレクトされ、請求書が更新されるはずです。

### 5.請求書の削除
Server Actionを使って請求書を削除するには、削除ボタンを`<form>`要素で囲み、bindを使ってidをServer Actionに渡します

```tsx
// app/ui/invoices/buttons.tsx

import { deleteInvoice } from '@/app/lib/actions';

// ...

export function DeleteInvoice({ id }: { id: string }) {
  const deleteInvoiceWithId = deleteInvoice.bind(null, id);

  return (
    <form action={deleteInvoiceWithId}>
      <button type="submit" className="rounded-md border p-2 hover:bg-gray-100">
        <span className="sr-only">Delete</span>
        <TrashIcon className="w-4" />
      </button>
    </form>
  );
}
```

actions.tsファイルの中に、deleteInvoiceという新しいアクションを作成します。

```typescript
// app/lib/actions.ts

export async function deleteInvoice(id: string) {
  await sql`DELETE FROM invoices WHERE id = ${id}`;
  revalidatePath('/dashboard/invoices');
}
```

このアクションは /dashboard/invoices パスで呼び出されるので、redirect を呼び出す必要はありません。revalidatePath をコールすると、新しいサーバリクエストが発生し、テーブルが再レンダリングされます。

## さらに読む
この章では、Server Actionsを使用してデータを変更する方法を学びました。また、revalidatePath APIを使用してNext.jsキャッシュを再検証する方法と、リダイレクトしてユーザーを新しいページにリダイレクトする方法についても学びました。

さらに、Server Actionsを使用したセキュリティについてもお読みください。

第12章終了

おめでとうございます！フォームと React Server Actions を使ってデータを変更する方法を学びました。
