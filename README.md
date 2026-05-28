# GitHub Pages LP Template (AI Agent Instructions)

このリポジトリは、**GitHub Project Pages 用LPを最短で作るためのテンプレート**です。  
AIエージェントはこのREADMEを「実行指示書」として扱ってください。

---

## 0. Goal

1. repoごとのLPを高速作成する  
2. GitHub Pages特有の404事故（`baseurl`）を回避する  
3. SEO実装 + 運用（Search Console）まで完了させる  
4. 手順を再現可能にする（他repoへ横展開）

---

## Prerequisites

- Bun (推奨: `1.3.14+`)
- Docker（`build-*.sh` を使う場合）
- `npx`（`serve` / `lighthouse` 実行用）
- GitHub Pages の公開権限（repo admin）

---

## 1. Input Contract (最初に埋める)

以下を必須入力とする。

```txt
owner=<GitHub owner>
repo=<repository name>
product_name=<product name>
lp_title=<title for <title> and H1>
lp_description=<meta description>
lang=<ja or en>
primary_cta_url=<usually GitHub repo URL>
secondary_cta_url=<README or Quick Start URL>
primary_color=<hex>
accent_color=<hex>
```

推奨:

```txt
pages_url=https://<owner>.github.io/<repo>/
baseurl=/<repo>
github_url=https://github.com/<owner>/<repo>
```

---

## 2. Directory Rules (不変)

- `site/` = source（編集対象）
- `docs/` = build artifact（**直編集禁止**）
- `.preview/` = local preview artifact（gitignore）
- `reports/` = Lighthouse output（gitignore）
- `scripts/` = automation scripts

**Rule:** LP本文・レイアウト・CSSの編集は `site/` のみで行う。

---

## 3. Files You Must Touch

最低限更新するファイル:

1. `_config.yml`
2. `site/index.md`
3. `site/_layouts/default.html`
4. `site/assets/css/lp.css`
5. `site/assets/img/*`（必要に応じて差し替え）

必要に応じて:

- `SEO_CHECKLIST.md`
- `github-pages-lp-seo-playbook.md`

---

## 4. Fast Execution Workflow (標準手順)

### Step 1: Project metadata を設定

`_config.yml` を更新:

- `title`
- `description`
- `lang`
- `url` (`https://<owner>.github.io`)
- `baseurl` (`/<repo>`)

### Step 2: LP本文を更新

`site/index.md` を更新:

- Hero (H1 + one-liner)
- value proposition
- use cases
- quick flow
- CTA（プロダクト導線）

禁止:

- CTAを実装ディレクトリ（`/tree/main/github-pages`）へ向ける

### Step 3: 画像を最適化

```bash
bun run scripts/optimize-hero-image.ts
```

生成物:

- `site/assets/img/knowledge-distillation-hero.webp`（表示用）
- `site/assets/img/og-image.jpg`（OG/Twitter）

### Step 4: ローカルプレビュー

```bash
./build-preview.sh
cd .preview
npx serve .
```

確認ポイント:

- desktop/mobile レイアウト
- CTAリンク
- 画像表示
- コンソールエラーなし

### Step 5: 本番artifact生成

```bash
# npx serve を止めて repo root に戻ってから実行
cd /path/to/this-repo
./build-dist.sh
```

### Step 6: Lighthouse品質ゲート

```bash
cd /path/to/this-repo
bash scripts/run-lighthouse.sh
bun run scripts/assert-lighthouse.ts reports/lighthouse.json 90 100
```

基準:

- Performance >= 90
- SEO = 100

---

## 5. Baseurl / 404 Troubleshooting

症状:

- `npx serve .` で CSS/画像が404

原因:

- 本番は `baseurl=/<repo>` 前提

対策:

- `docs/` ではなく `.preview/` を serve する
- HTML/CSS中のパスは `relative_url` を使う

例:

```liquid
<link rel="stylesheet" href="{{ '/assets/css/lp.css' | relative_url }}">
<img src="{{ '/assets/img/knowledge-distillation-hero.webp' | relative_url }}" alt="...">
```

---

## 6. SEO Requirements (Technical)

`default.html` で以下を満たす:

- `canonical`
- `og:*`
- `twitter:*`
- `robots` / `googlebot`
- `theme-color`
- `manifest`
- `favicon`
- `hreflang`

`site/robots.txt`:

- `User-agent: *`
- `Allow: /`
- `Sitemap: https://<owner>.github.io/<repo>/sitemap.xml`

---

## 7. SEO Requirements (Ops)

実装だけで完了しない。以下を必ず実施:

1. Search Console で URL-prefix property 追加
2. sitemap 送信
3. URL検査でトップページを検査
4. インデックス登録リクエスト
5. T+7 / T+14 / T+30 で状態確認

---

## 8. Definition of Done

以下すべて満たしたら完了:

- [ ] `site/` だけを編集した
- [ ] `.preview` で表示確認した
- [ ] `docs/` を再生成した
- [ ] Lighthouse gate を通過した
- [ ] `canonical / robots / sitemap` が正しい
- [ ] CTAがプロダクト導線
- [ ] Search Console運用を実施またはタスク化

---

## 9. Commands (Quick Reference)

```bash
# image optimization
bun run scripts/optimize-hero-image.ts

# local preview build
./build-preview.sh
npx serve .preview

# production artifact build
./build-dist.sh

# lighthouse audit
bash scripts/run-lighthouse.sh
bun run scripts/assert-lighthouse.ts reports/lighthouse.json 90 100
```

---

## 10. GitHub Actions

- Pages deploy: `.github/workflows/pages.yml`
- SEO audit: `.github/workflows/seo-audit.yml`

推奨:

- PRで `github-pages/**` 変更時にSEO監査を実行
- `main` pushでPages deploy

---

## 11. Template Repo Bootstrap (first setup)

このテンプレートrepoを新規作成した直後は以下を実行:

```bash
git init
git add .
git commit -m "Initial commit: GitHub Pages LP template"
```

その後、各プロダクトrepoへコピーして `owner/repo/baseurl` を差し替える。
