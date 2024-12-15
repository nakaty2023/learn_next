# 第15章 認証を追加する
前の章では、フォームのバリデーションを追加し、アクセシビリティを改善することで請求書ルートの構築を終えました。この章では、ダッシュボードに認証を追加します。

## この章では...

この章で扱うトピックは次のとおりです。

* 認証とは何か
* NextAuth.jsを使ってアプリに認証を追加する方法。
* ミドルウェアを使用してユーザーをリダイレクトし、ルートを保護する方法。
* ReactのuseActionStateを使用して、保留状態やフォームエラーを処理する方法。

## 認証とは？
認証は今日、多くのウェブ・アプリケーションで重要な役割を担っています。ユーザーが本人であるかどうかをシステムがチェックする方法です。

安全なウェブサイトでは、ユーザーの身元を確認するために複数の方法を使用することがよくあります。例えば、ユーザー名とパスワードを入力した後、サイトがユーザーのデバイスに認証コードを送信したり、Google Authenticatorのような外部アプリを使用したりします。この2要素認証（2FA）は、セキュリティの強化に役立ちます。たとえパスワードを知られたとしても、トークンがなければアカウントにアクセスできません。

### 認証 vs. 認可
ウェブ開発において、認証と認可は異なる役割を果たします

* 認証とは、ユーザーが本人であることを確認することです。ユーザー名とパスワードのようなもので、本人であることを証明します。
* 認可は次のステップです。ユーザーの身元が確認されると、認可はアプリケーションのどの部分の使用を許可するかを決定する。

つまり、認証はあなたが誰であるかをチェックし、認可はあなたがアプリケーションで何ができるか、何にアクセスできるかを決定します。

## ログインルートの作成
まず、アプリケーションに `/login` という新しいルートを作成し、次のコードを貼り付けます

```tsx
// app/login/page.tsx

import AcmeLogo from '@/app/ui/acme-logo';
import LoginForm from '@/app/ui/login-form';

export default function LoginPage() {
  return (
    <main className="flex items-center justify-center md:h-screen">
      <div className="relative mx-auto flex w-full max-w-[400px] flex-col space-y-2.5 p-4 md:-mt-32">
        <div className="flex h-20 w-full items-end rounded-lg bg-blue-500 p-3 md:h-36">
          <div className="w-32 text-white md:w-36">
            <AcmeLogo />
          </div>
        </div>
        <LoginForm />
      </div>
    </main>
  );
}
```
ページが`<LoginForm />`をインポートしていることに気づくでしょう。

## NextAuth.js
アプリケーションに認証を追加するために、NextAuth.jsを使います。NextAuth.jsは、セッション管理、サインイン、サインアウトなど、認証にかかわる複雑な作業の多くを抽象化してくれます。これらの機能を手動で実装することもできますが、そのプロセスには時間がかかり、エラーも発生しがちです。NextAuth.jsはこのプロセスを単純化し、Next.jsアプリケーションにおける認証のための統一されたソリューションを提供します。

## NextAuth.jsのセットアップ
ターミナルで次のコマンドを実行して、NextAuth.jsをインストールしてください

```bash
pnpm i next-auth@beta
```

ここでは、Next.js 14と互換性のあるNextAuth.jsのベータ版をインストールします。

次に、アプリケーションの秘密鍵を生成します。このキーはクッキーを暗号化し、ユーザーセッションのセキュリティを確保するために使用されます。ターミナルで次のコマンドを実行してください

```bash
openssl rand -base64 32
```

次に、.envファイルで、生成した鍵をAUTH_SECRET変数に追加する

```
AUTH_SECRET=your-secret-key
```

authを本番で動作させるには、Vercelプロジェクトの環境変数も更新する必要があります。Vercelで環境変数を追加する方法については、こちらのガイドをご覧ください。

### ページ・オプションの追加
プロジェクトのルートにauth.config.tsファイルを作成し、authConfigオブジェクトをエクスポートします。このオブジェクトには、NextAuth.jsの設定オプションが含まれます。今のところ、pagesオプションだけが含まれています

```typescript
// auth.config.ts

import type { NextAuthConfig } from 'next-auth';

export const authConfig = {
  pages: {
    signIn: '/login',
  },
} satisfies NextAuthConfig;
```

pages オプションを使って、カスタムサインイン、サインアウト、エラーページのルートを指定できます。これは必須ではありませんが、pages オプションに signIn: '/login' を追加することで、ユーザーは NextAuth.js のデフォルトページではなく、カスタムログインページにリダイレクトされます。

## Next.jsミドルウェアでルートを保護する
次に、ルートを保護するロジックを追加します。ログインしていないユーザーがダッシュボードのページにアクセスできないようにします。

```typescript
// auth.config.ts
import type { NextAuthConfig } from 'next-auth';

export const authConfig = {
  pages: {
    signIn: '/login',
  },
  callbacks: {
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user;
      const isOnDashboard = nextUrl.pathname.startsWith('/dashboard');
      if (isOnDashboard) {
        if (isLoggedIn) return true;
        return false; // Redirect unauthenticated users to login page
      } else if (isLoggedIn) {
        return Response.redirect(new URL('/dashboard', nextUrl));
      }
      return true;
    },
  },
  providers: [], // Add providers with an empty array for now
} satisfies NextAuthConfig;
```

authorizedコールバックは、リクエストがNext.jsミドルウェア経由でページにアクセスすることを許可されているかどうかを確認するために使用されます。リクエストが完了する前に呼び出され、auth プロパティと request プロパティを持つオブジェクトを受け取ります。authプロパティにはユーザーのセッションが含まれ、requestプロパティには受信リクエストが含まれます。

プロバイダオプションは配列で、さまざまなログインオプションを列挙します。今のところ、NextAuthの設定を満たすための空の配列です。これについては、資格情報プロバイダの追加で詳しく説明します。

次に、authConfigオブジェクトをMiddlewareファイルにインポートします。プロジェクトのルートに middleware.ts というファイルを作成し、以下のコードを貼り付けます

```typescript
// middleware.ts

import NextAuth from 'next-auth';
import { authConfig } from './auth.config';

export default NextAuth(authConfig).auth;

export const config = {
  // https://nextjs.org/docs/app/building-your-application/routing/middleware#matcher
  matcher: ['/((?!api|_next/static|_next/image|.*\\.png$).*)'],
};
```

ここでは、NextAuth.jsをauthConfigオブジェクトで初期化し、authプロパティをエクスポートしています。また、Middleware の matcher オプションを使って、特定のパスで実行するように指定しています。

このタスクに Middleware を使用する利点は、Middleware が認証を確認するまで保護されたルートのレンダリングが開始されないため、アプリケーションのセキュリティとパフォーマンスの両方が向上することです。

### パスワードのハッシュ化
パスワードをデータベースに保存する前にハッシュ化するのは良い習慣です。ハッシュ化することで、パスワードがランダムな固定長の文字列に変換され、ユーザーのデータが公開されてもセキュリティのレイヤーが提供されます。

seed.jsファイルでは、bcryptというパッケージを使って、ユーザーのパスワードをデータベースに保存する前にハッシュ化しています。この章の後半で、ユーザーが入力したパスワードがデータベースのパスワードと一致するかどうかを比較するために、再びこのパッケージを使用します。ただし、bcryptパッケージ用に別のファイルを作成する必要がある。これは、bcryptがNext.jsミドルウェアでは利用できないNode.js APIに依存しているためです。

auth.tsという新しいファイルを作成し、authConfigオブジェクトを展開します

```typescript
// auth.ts

import NextAuth from 'next-auth';
import { authConfig } from './auth.config';

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
});
```

## 認証情報プロバイダーの追加
次に、NextAuth.jsのプロバイダオプションを追加します。プロバイダとは、GoogleやGitHubなど、さまざまなログインオプションを列挙する配列です。このコースでは、Credentials プロバイダーのみに焦点を当てます。

Credentials プロバイダを使うと、ユーザ名とパスワードでログインできるようになります。

```typescript
// auth.ts

import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
import Credentials from 'next-auth/providers/credentials';

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [Credentials({})],
});
```

知っておいて損はない：

ここではCredentialsプロバイダを使用していますが、一般的にはOAuthやメールプロバイダなどの代替プロバイダを使用することを推奨します。オプションの一覧は、NextAuth.jsのドキュメントをご覧ください。

### サインイン機能の追加
認証ロジックを処理するためにauthorize関数を使用することができます。Server Actionsと同様に、zodを使用して、ユーザがデータベースに存在するかどうかをチェックする前に、電子メールとパスワードを検証することができます

```typescript
// auth.ts

import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
import Credentials from 'next-auth/providers/credentials';
import { z } from 'zod';

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        const parsedCredentials = z
          .object({ email: z.string().email(), password: z.string().min(6) })
          .safeParse(credentials);
      },
    }),
  ],
});
```


認証情報を検証した後、データベースからユーザーを問い合わせる新しいgetUser関数を作成します。

```typescript
// auth.ts

import NextAuth from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import { authConfig } from './auth.config';
import { z } from 'zod';
import { sql } from '@vercel/postgres';
import type { User } from '@/app/lib/definitions';
import bcrypt from 'bcrypt';

async function getUser(email: string): Promise<User | undefined> {
  try {
    const user = await sql<User>`SELECT * FROM users WHERE email=${email}`;
    return user.rows[0];
  } catch (error) {
    console.error('Failed to fetch user:', error);
    throw new Error('Failed to fetch user.');
  }
}

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        const parsedCredentials = z
          .object({ email: z.string().email(), password: z.string().min(6) })
          .safeParse(credentials);

        if (parsedCredentials.success) {
          const { email, password } = parsedCredentials.data;
          const user = await getUser(email);
          if (!user) return null;
        }

        return null;
      },
    }),
  ],
});
```

次に、bcrypt.compareを呼び出して、パスワードが一致するかどうかをチェックする

```typescript
// auth.ts

import NextAuth from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import { authConfig } from './auth.config';
import { sql } from '@vercel/postgres';
import { z } from 'zod';
import type { User } from '@/app/lib/definitions';
import bcrypt from 'bcrypt';

// ...

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        // ...

        if (parsedCredentials.success) {
          const { email, password } = parsedCredentials.data;
          const user = await getUser(email);
          if (!user) return null;
          const passwordsMatch = await bcrypt.compare(password, user.password);

          if (passwordsMatch) return user;
        }

        console.log('Invalid credentials');
        return null;
      },
    }),
  ],
});
```

最後に、パスワードが一致すればユーザーを返し、そうでなければnullを返してユーザーがログインできないようにする。

### ログインフォームの更新
次に、認証ロジックをログインフォームに接続する必要があります。actions.tsファイルに、authenticateという新しいアクションを作成します。このアクションは、auth.tsからsignIn関数をインポートする必要があります

```typescript
// app/lib/actions.ts

'use server';

import { signIn } from '@/auth';
import { AuthError } from 'next-auth';

// ...

export async function authenticate(
  prevState: string | undefined,
  formData: FormData,
) {
  try {
    await signIn('credentials', formData);
  } catch (error) {
    if (error instanceof AuthError) {
      switch (error.type) {
        case 'CredentialsSignin':
          return 'Invalid credentials.';
        default:
          return 'Something went wrong.';
      }
    }
    throw error;
  }
}
```

CredentialsSignin' エラーが発生した場合、適切なエラーメッセージを表示したい。NextAuth.jsのエラーについてはドキュメントを参照してください。

最後に、login-form.tsx コンポーネントで、React の useActionState を使用して、サーバーアクションを呼び出し、フォームエラーを処理し、フォームの保留状態を表示します

```tsx
// app/ui/login-form.tsx

'use client';

import { lusitana } from '@/app/ui/fonts';
import {
  AtSymbolIcon,
  KeyIcon,
  ExclamationCircleIcon,
} from '@heroicons/react/24/outline';
import { ArrowRightIcon } from '@heroicons/react/20/solid';
import { Button } from '@/app/ui/button';
import { useActionState } from 'react';
import { authenticate } from '@/app/lib/actions';

export default function LoginForm() {
  const [errorMessage, formAction, isPending] = useActionState(
    authenticate,
    undefined,
  );

  return (
    <form action={formAction} className="space-y-3">
      <div className="flex-1 rounded-lg bg-gray-50 px-6 pb-4 pt-8">
        <h1 className={`${lusitana.className} mb-3 text-2xl`}>
          Please log in to continue.
        </h1>
        <div className="w-full">
          <div>
            <label
              className="mb-3 mt-5 block text-xs font-medium text-gray-900"
              htmlFor="email"
            >
              Email
            </label>
            <div className="relative">
              <input
                className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
                id="email"
                type="email"
                name="email"
                placeholder="Enter your email address"
                required
              />
              <AtSymbolIcon className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
            </div>
          </div>
          <div className="mt-4">
            <label
              className="mb-3 mt-5 block text-xs font-medium text-gray-900"
              htmlFor="password"
            >
              Password
            </label>
            <div className="relative">
              <input
                className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
                id="password"
                type="password"
                name="password"
                placeholder="Enter password"
                required
                minLength={6}
              />
              <KeyIcon className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
            </div>
          </div>
        </div>
        <Button className="mt-4 w-full" aria-disabled={isPending}>
          Log in <ArrowRightIcon className="ml-auto h-5 w-5 text-gray-50" />
        </Button>
        <div
          className="flex h-8 items-end space-x-1"
          aria-live="polite"
          aria-atomic="true"
        >
          {errorMessage && (
            <>
              <ExclamationCircleIcon className="h-5 w-5 text-red-500" />
              <p className="text-sm text-red-500">{errorMessage}</p>
            </>
          )}
        </div>
      </div>
    </form>
  );
}
```

## ログアウト機能の追加
`<SideNav />`にログアウト機能を追加するには、`<form>`要素内でauth.tsのsignOut関数を呼び出します

```tsx
// ui/dashboard/sidenav.tsx

import Link from 'next/link';
import NavLinks from '@/app/ui/dashboard/nav-links';
import AcmeLogo from '@/app/ui/acme-logo';
import { PowerIcon } from '@heroicons/react/24/outline';
import { signOut } from '@/auth';

export default function SideNav() {
  return (
    <div className="flex h-full flex-col px-3 py-4 md:px-2">
      // ...
      <div className="flex grow flex-row justify-between space-x-2 md:flex-col md:space-x-0 md:space-y-2">
        <NavLinks />
        <div className="hidden h-auto w-full grow rounded-md bg-gray-50 md:block"></div>
        <form
          action={async () => {
            'use server';
            await signOut();
          }}
        >
          <button className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3">
            <PowerIcon className="w-6" />
            <div className="hidden md:block">Sign Out</div>
          </button>
        </form>
      </div>
    </div>
  );
}
```

## 試してみる
では、試してみてください。以下の認証情報を使用して、アプリケーションにログインおよびログアウトできるはずです

* 電子メール: user@nextmail.com
* パスワード: 123456

第15章を完了しました

アプリケーションに認証を追加し、ダッシュボードのルートを保護しました。
