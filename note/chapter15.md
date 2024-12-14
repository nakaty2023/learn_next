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
