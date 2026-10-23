# BOXP-275: b0xp.io をソフトウェアエンジニアのポートフォリオサイトとして作り直す

チケット: BOXP (Obsidian vault) `Tickets/BOXP-275`
作成日: 2026-10-23

## 背景

- 現行 b0xp.io は 2014 年起源の ClojureScript/Om 製 SPA（boxp/shanghai-next）。アイコンリンク 3 つのみで、エンジニアとしての情報が無い。Om 0.x はメンテ不能で依存更新が滞留。
- 配信は Cloudflare Pages（手動作成の `shanghai` プロジェクト、`shanghai-4t4.pages.dev`）。DNS は boxp/arch の Terraform（`terraform/cloudflare/b0xp.io/portfolio/dns.tf`）で CNAME 管理。

## 方針

- 新規リポジトリ boxp/b0xp.io を Astro 7 + Tailwind CSS 4 で作成（boxp/iikanji と同じ構成、Node 22）。shanghai-next は改修しない。
- Cloudflare Pages プロジェクト `boxp-portfolio` を boxp/arch の `portfolio/pages.tf` に追加し、workflow の `--project-name` と一致させる（iikanji で起きた不一致を避ける）。
- デプロイは GitHub Actions。PR ではビルド + Lighthouse CI のみ、`main` push で `wrangler pages deploy`。Actions は SHA pin。
- コンテンツはリポジトリ内の Content Collections で管理し、GitHub / LinkedIn API への実行時依存は持たない。
- 経歴は LinkedIn を正本（source of truth）とし、会社名・役職・在籍期間・担当領域の要約のみを 1 社 1 Markdown に転記する。詳細は LinkedIn へのリンクで案内する。
- 主要プロジェクトは Star 数ではなく最終 push 日を主キーに並べ、最終 push が 2 年以上前のものは「過去の取り組み」として折りたたむ。
- 多言語対応は初期スコープ外（日本語ページ + 英語の一言紹介のみ）。

## フェーズ分け（2026-10-23 本人承認）

本人承認「一旦 stage 環境も別サブドメインで用意してそこで公開するまでは進めさせてほしい」に基づき、現フェーズは本番（b0xp.io / www.b0xp.io）を変更せず、独立した stage を実公開して確認するところまでとする。

- stage: Cloudflare Pages プロジェクト `boxp-portfolio-stage`、カスタムドメイン `portfolio-stage.b0xp.io`（boxp/arch の `terraform/cloudflare/b0xp.io/portfolio/` で管理。既存の stage 命名 `hitohub-stage.b0xp.io` に揃えた。2026-10-23 時点で Terraform・実 DNS ともに未使用を確認）。
- stage ビルドは `SITE_ENV=stage SITE_URL=https://portfolio-stage.b0xp.io`。`<meta name="robots" content="noindex, nofollow">`、robots.txt `Disallow: /`、画面上部に stage バナーを出す。noindex は検索エンジン向けの表示抑制であり秘密保護ではないため、未確認の経歴・メールアドレス・非公開原稿・LinkedIn エクスポート原本は stage にも載せない（経歴セクションは「準備中」表示）。
- `main` push は stage へのみデプロイする。本番プロジェクト `boxp-portfolio` へのデプロイ・本番 DNS 切替・旧サイト / 旧 Pages プロジェクトの削除・shanghai-next の archive は未承認のため行わない。
- Cloudflare API トークンは codex-workspace から取得できず、新規発行・権限拡大も未承認のため、初回 stage デプロイは boxp/arch の既存 Secrets を使う `deploy-portfolio-stage.yml`（workflow_dispatch）で行う。本リポジトリの Secrets には公開値の `CLOUDFLARE_ACCOUNT_ID` のみ設定済み。`CLOUDFLARE_API_TOKEN`（Pages: Edit）は本人に設定を依頼し、設定後は `main` push で自動デプロイされる（未設定時はスキップして notice）。

## Content Collections

| collection | loader | 主なフィールド |
| --- | --- | --- |
| experience | glob `src/content/experience/*.md`（`_` 始まりは除外） | company, title, start, end, location, summary, linkedinSyncedAt |
| skills | file `skills.json` | category, order, items[{name, evidence[]}] |
| projects | file `projects.json` | repo, name, description, language, tags, lastUpdated, stars, archived |
| articles | file `articles.json` | title, url, publishedAt, publisher, summary |
| talks | file `talks.json` | title, url, date, event, category(engineering/hobby) |

profile（名前・ハンドル・一言紹介・SNS リンク）は `src/content/profile.json` を直接 import する。

## 転記元と取得日

| セクション | 転記元 | 取得日 | 状態 |
| --- | --- | --- | --- |
| 経歴（experience） | LinkedIn プロフィール（本人提供の PDF または Data export、vault `Projects/BOXP-275/linkedin/`） | 未提供 | **未着手**。提供後に転記し、`linkedinSyncedAt` に転記日を記録して本人レビューを受ける |
| スキル（skills） | GitHub 公開リポジトリの言語・内容（`gh repo list boxp`） | 2026-10-23 | 暫定。LinkedIn の Skills と突き合わせて更新する |
| プロジェクト（projects） | `gh repo list boxp --source --no-archived`（fork 除外）の pushedAt / description | 2026-10-23 | 完了 |
| 記事（articles） | Medium eureka-engineering の RSS（2025-12-23 公開「自宅k8s clusterインシデント」）と 2024 年記事 URL | 2026-10-23 | 2024 年記事の公開日は月までしか特定できず `2024-12`。Lunar レビュー記事は公開 URL 未確認のため未掲載 |
| 登壇（talks） | speakerdeck.com/boxp の各デッキページ（published date） | 2026-10-23 | 完了（13 本。VR/アバター系 6 本は hobby として折りたたみ） |
| SNS リンク | GitHub プロフィール（X `b0xp2`、YouTube `@BOXPKETARO`） | 2026-10-23 | LinkedIn URL は未提供（`links.linkedin: null`。設定すると自動で表示） |

## 経歴の掲載粒度（既定、本人が変更可）

- 会社名・役職・期間（年月）・担当領域の要約（1〜3 行）のみ。
- 年収・退職理由・社外秘になり得るプロジェクト名は載せない。
- 公開情報から分かっている範囲（参考）: GitHub プロフィールの company は `@eure`、2024〜2025 年の記事は eureka-engineering の Medium publication に掲載。これらは LinkedIn エクスポートで裏取りするまで experience には書かない。

## 作業手順

1. [x] 新リポジトリ boxp/b0xp.io 作成、Astro + Tailwind 雛形、README / AGENTS.md / renovate.json / deploy.yml / Lighthouse CI
2. [x] プロジェクト / 記事 / 登壇 / SNS リンクのコンテンツ実装、OGP / favicon / sitemap / robots.txt
3. [ ] LinkedIn エクスポート提供後に experience / skills を転記し本人レビュー
4. [ ] boxp/arch に Pages プロジェクト `boxp-portfolio`（本番、未使用）と `boxp-portfolio-stage` + `portfolio-stage.b0xp.io` を追加（PR #13821）、tfaction apply 後に arch の `deploy-portfolio-stage.yml` で stage へ初回デプロイし https://portfolio-stage.b0xp.io/ の表示・HTTPS・noindex を確認。`CLOUDFLARE_API_TOKEN` の本リポジトリへの設定は本人に依頼
5. [ ] boxp/arch に DNS 切替 PR（`dns.tf` の CNAME を `cloudflare_pages_project.portfolio.subdomain` へ、`cloudflare_pages_domain` を b0xp.io / www.b0xp.io に追加）。旧 `shanghai` プロジェクトからカスタムドメインを外してから apply
6. [ ] 後片付け: boxp/boxp プロフィール README のリンク確認、shanghai-next の archive または README 誘導、旧 Pages プロジェクト `shanghai` の削除可否

## 検証

- `npm run build`（`astro check` 0 errors / 0 warnings）
- Lighthouse はローカルに Chromium が無いため PR の Lighthouse CI ジョブ（モバイル、3 回計測、Performance / Accessibility / SEO ≥ 0.9 をアサート）で確認する。PR #1 の初回計測（2026-10-23）は Performance / Accessibility / Best Practices / SEO すべて 100（3 回とも）
- stage ビルド（`npm run build:stage`）で robots.txt が `Disallow: /`、index.html に noindex メタと stage バナー、canonical / sitemap が `https://portfolio-stage.b0xp.io` になることをローカルで確認（2026-10-23）
- 本番切替前に https://portfolio-stage.b0xp.io/ で表示・OGP を本人が確認する

## 未決事項（本人判断が必要）

- LinkedIn プロフィール URL と経歴エクスポートの提供（vault `Projects/BOXP-275/linkedin/`）
- メールアドレスを連絡先として載せるか（現状は SNS の DM 案内のみ）
- shanghai-next を archive するか
- Lunar レビュー記事の公開 URL（未公開なら掲載しない）
- 本リポジトリ Secrets への `CLOUDFLARE_API_TOKEN`（Pages: Edit）設定（設定されるまでは boxp/arch の workflow_dispatch で stage デプロイ）
- 本番切替（b0xp.io / www.b0xp.io の DNS・カスタムドメイン切替、本番プロジェクトへのデプロイ追加）の承認
