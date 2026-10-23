# AGENTS.md

このリポジトリは Keitaro Takeuchi (BOXP) のポートフォリオサイト https://b0xp.io/ のソースです。

## 構成

- Astro 7 + Tailwind CSS 4（`@tailwindcss/vite`）。クライアント JS は使わない（静的 HTML + CSS のみ）。
- コンテンツは `src/content/` の Content Collections（スキーマ: `src/content.config.ts`）。
- ページは `src/pages/index.astro` の 1 ページ構成。セクション共通部品は `src/components/`。
- OGP 画像は `npm run og` で `public/og.png` を生成する（`@resvg/resvg-js` + 同梱 Inter）。
- デプロイは `.github/workflows/deploy.yml`。PR ではビルド + Lighthouse CI のみ、`main` push で Cloudflare Pages `boxp-portfolio` へデプロイ。
- DNS / Pages プロジェクトは boxp/arch の Terraform（`terraform/cloudflare/b0xp.io/portfolio/`）で管理。ここでは触らない。

## 作業ルール

- `npm run build` が通ること（`astro check` を含む）。
- 依存を追加するときは理由を PR に書く。ランタイムで外部 API（GitHub / LinkedIn 等）を叩く実装は入れない。
- GitHub Actions は SHA でピン留めし、コメントにバージョンを残す。
- 経歴（`src/content/experience/`）は LinkedIn を正本とし、本人が提供したエクスポートから転記する。推測で書かない。年収・退職理由・社外秘になり得る情報は書かない。
- プロジェクト一覧は最終更新日（`lastUpdated`）降順。最終 push が 2 年以上前のものは `archived: true` にする。
- Lighthouse（モバイル）で Performance / Accessibility / SEO が 90 を下回る変更は入れない。
- 設計メモは `docs/project_docs/<チケット番号>/plan.md` に残す。

## 開発コマンド

```sh
npm ci
npm run dev
npm run build
npm run og
```
