# 第14章 アクセシビリティの向上
前の章では、エラー（404エラーを含む）をキャッチし、ユーザーにフォールバックを表示する方法について見てきました。しかし、パズルのもう1つのピースであるフォームバリデーションについてはまだ説明する必要があります。サーバーアクションを使ってサーバーサイドのバリデーションを実装する方法と、ReactのuseActionStateフックを使ってフォームエラーを表示する方法を、アクセシビリティを考慮しながら見ていきましょう！

## この章では...

この章で扱うトピックは次のとおりです。

* Next.jsでeslint-plugin-jsx-a11yを使用してアクセシビリティのベストプラクティスを実装する方法。
* サーバーサイドのフォームバリデーションを実装する方法
* ReactのuseActionStateフックを使用してフォームエラーを処理し、ユーザーに表示する方法。

## アクセシビリティとは何ですか？
アクセシビリティとは、障害のある人を含め、誰もが使用できるWebアプリケーションを設計し、実装することを指します。キーボードナビゲーション、セマンティックHTML、画像、色、動画など、多くの領域をカバーする広大なトピックです。

このコースではアクセシビリティについて深く掘り下げませんが、Next.jsで利用可能なアクセシビリティ機能と、アプリケーションをよりアクセシブルにするための一般的なプラクティスについて説明します。

アクセシビリティについてもっと学びたい方は、web.dev.jpのLearn Accessibilityコースをおすすめします。

## Next.jsでESLintアクセシビリティプラグインを使う
Next.jsでは、アクセシビリティの問題を早期に発見するために、ESLint設定にeslint-plugin-jsx-a11yプラグインが含まれています。たとえば、altテキストがない画像や、aria-*属性やrole属性の間違った使い方などを警告します。

オプションとして、これを試したい場合は、package.jsonファイルにスクリプトとしてnext lintを追加してください

```json
// package.json

"scripts": {
    "build": "next build",
    "dev": "next dev",
    "start": "next start",
    "lint": "next lint"
},
```

それからターミナルで`pnpm lint`を実行する

これはあなたのプロジェクトにESLintをインストールし、設定するためのガイドです。今 pnpm lint を実行すると、次のような出力が表示されるはずです

```bash
✔ No ESLint warnings or errors
```

しかし、altテキストがない画像があったらどうなるでしょうか？調べてみましょう！

/app/ui/invoices/table.tsxに行き、画像からalt propを削除してください。エディタの検索機能を使えば、`<Image>`をすぐに見つけることができます

```tsx
// app/ui/invoices/table.tsx

<Image
  src={invoice.image_url}
  className="rounded-full"
  width={28}
  height={28}
  alt={`${invoice.name}'s profile picture`} // Delete this line
/>
```

もう一度pnpm lintを実行すると、次のような警告が表示されるはずだ

```bash
./app/ui/invoices/table.tsx
45:25  Warning: Image elements must have an alt prop,
either with meaningful text, or an empty string for decorative images. jsx-a11y/alt-text
```

リンターの追加と設定は必須のステップではありませんが、開発プロセスでアクセシビリティの問題を発見するのに役立ちます。
