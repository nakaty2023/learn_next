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