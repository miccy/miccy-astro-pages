# Nasazení Astro projektu na GitHub Pages z monorepo

Tento dokument popisuje, jak nasadit Astro projekt z monorepo struktury na GitHub Pages pomocí GitHub Actions.

## Konfigurace pro monorepo strukturu

### 1. Úprava konfiguračního souboru Astro

Pro portfolio aplikaci upravíme `apps/portfolio/astro.config.mjs`:

```js
import { defineConfig } from "astro/config";
import tailwind from "@astrojs/tailwind";
import mdx from "@astrojs/mdx";
import sitemap from "@astrojs/sitemap";

export default defineConfig({
  integrations: [
    tailwind({
      config: { path: "../../packages/config/tailwind/index.js" }
    }),
    mdx(),
    sitemap()
  ],
  site: "https://miccy.github.io", // Změňte na vaši GitHub Pages URL
  // Pokud publikujete na uživatelské GitHub Pages (username.github.io), ponechte base jako '/'
  // Pokud publikujete do podadresáře, změňte na '/nazev-podadresare'
  base: "/",

  output: "static" // GitHub Pages podporuje pouze statický výstup
});
```

### 2. Nastavení GitHub Actions pro monorepo

Vytvoříme workflow soubor `.github/workflows/deploy.yml` v kořenovém adresáři monorepo:

```yaml
name: Deploy Portfolio to GitHub Pages

on:
  # Spustit při každém push do hlavní větve
  push:
    branches: [main]
    paths:
      - "apps/portfolio/**"
      - "packages/**"
      - ".github/workflows/deploy.yml"
  # Povolit manuální spuštění z GitHub Actions
  workflow_dispatch:

# Nastavit oprávnění GITHUB_TOKEN pro deployment
permissions:
  contents: read
  pages: write
  id-token: write

# Povolit pouze jeden concurrent deployment
concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Bun
        uses: oven-sh/setup-bun@v1
        with:
          bun-version: latest

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Install dependencies
        run: bun install

      - name: Build portfolio
        run: bun turbo build --filter=portfolio

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./apps/portfolio/dist

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## Nastavení GitHub Pages v repozitáři

### 1. Vytvoření nebo konfigurace GitHub repozitáře

Máte dvě možnosti:

#### A. Použití uživatelského GitHub Pages

Pokud chcete publikovat na `username.github.io`:

1. Vytvořte repozitář pojmenovaný přesně `username.github.io` (nahraďte `username` vaším GitHub uživatelským jménem)
2. Nastavte `base: "/"` v `astro.config.mjs`
3. Nastavte `site: "https://username.github.io"` v `astro.config.mjs`

#### B. Použití projektového GitHub Pages

Pokud chcete publikovat na `username.github.io/nazev-projektu`:

1. Vytvořte repozitář s libovolným názvem (např. `miccy-astro`)
2. Nastavte `base: "/nazev-projektu/"` v `astro.config.mjs`
3. Nastavte `site: "https://username.github.io"` v `astro.config.mjs`

### 2. Konfigurace repozitáře pro GitHub Pages

1. Přejděte do nastavení vašeho repozitáře (záložka "Settings")
2. V levém menu klikněte na "Pages"
3. V sekci "Build and deployment":
   - V části "Source" vyberte "GitHub Actions"

## Vlastní doména

Pokud chcete použít vlastní doménu (např. `miccy.dev`):

1. Přejděte do nastavení GitHub repozitáře a v sekci "Pages"
2. V části "Custom domain" zadejte vaši doménu
3. Nastavte DNS záznamy u vašeho poskytovatele domény:
   - Pro apex doménu (např. `miccy.dev`): vytvořte A záznamy směřující na IP adresy GitHub Pages
   - Pro subdoménu (např. `www.miccy.dev`): vytvořte CNAME záznam směřující na `username.github.io`
4. Zaškrtněte "Enforce HTTPS" pro zabezpečení

Nezapomeňte také upravit `site` v `astro.config.mjs`:

```js
site: 'https://miccy.dev',
```

## Speciální konfigurace pro monorepo

### 1. Sdílené konfigurace a komponenty

Pokud máte v monorepo sdílené balíčky (v adresáři `packages`), ujistěte se, že:

1. Workflow instaluje všechny závislosti
2. Build proces má přístup ke všem potřebným balíčkům
3. Turborepo je správně nakonfigurováno pro spuštění buildu

### 2. Optimalizace workflow pro monorepo

Pro rychlejší buildy a nasazení můžete optimalizovat workflow:

```yaml
# Přidejte do job build
steps:
  # ... existující kroky ...

  - name: Cache dependencies
    uses: actions/cache@v3
    with:
      path: |
        node_modules
        apps/*/node_modules
        packages/*/node_modules
        ~/.bun/install/cache
      key: ${{ runner.os }}-modules-${{ hashFiles('**/bun.lockb') }}
```

### 3. Paralelní nasazení více aplikací

Pokud potřebujete nasadit více aplikací z monorepo (např. portfolio i blog), můžete vytvořit více workflow souborů nebo použít matici:

```yaml
jobs:
  build:
    strategy:
      matrix:
        app: [portfolio, blog]
    # ... zbytek konfigurace ...

    steps:
      # ... ostatní kroky ...

      - name: Build ${{ matrix.app }}
        run: bun turbo build --filter=${{ matrix.app }}

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          name: ${{ matrix.app }}-artifact
          path: ./apps/${{ matrix.app }}/dist
```

## Řešení potíží

### 1. Problémy se sdílenými balíčky

Pokud se objeví chyby související se sdílenými balíčky:

```
Error: Cannot find module '@miccy/ui'
```

Ujistěte se, že:

1. Máte správně nakonfigurované workspaces v kořenovém `package.json`
2. Používáte správné cesty importu
3. Sdílené balíčky jsou přidány jako závislosti v `package.json` aplikace

### 2. Problémy s Turborepo

Pokud Turborepo neprovádí build správně:

1. Zkontrolujte `turbo.json` v kořenovém adresáři
2. Ujistěte se, že máte správně nastavené závislosti mezi balíčky
3. Spusťte build lokálně s `--dry` příznakem pro kontrolu procesu

```bash
bun turbo build --filter=portfolio --dry
```

### 3. Problémy s cestami assetů

Pokud se assety (obrázky, CSS) nezobrazují správně:

1. Ujistěte se, že všechny cesty začínají správným prefixem base
2. Použijte import.meta.env.BASE_URL pro dynamické cesty:

```astro
<img src={`${import.meta.env.BASE_URL}assets/logo.png`} alt="Logo" />
```

## Monitorování a analytika

1. **Přístup k UI pro GitHub Actions** - Sledujte průběh nasazení přes záložku Actions v repozitáři
2. **GitHub Pages statistiky** - Základní statistiky jsou dostupné v sekci Pages v nastavení repozitáře
3. **Pokročilá analytika** - Integrujte privacy-friendly analytický nástroj jako Umami

## Automatizace kontroly před nasazením

Pro zajištění kvality před nasazením přidejte tyto kroky do workflow:

```yaml
# Přidejte před build krok
- name: Check types
  run: bun turbo typecheck --filter=portfolio

- name: Lint code
  run: bun turbo lint --filter=portfolio

- name: Run tests
  run: bun turbo test --filter=portfolio
```

## Závěr

Tímto způsobem můžete efektivně nasadit Astro aplikaci z monorepo struktury na GitHub Pages. Nastavení může vyžadovat trochu více práce než u běžného projektu, ale poskytuje flexibilitu pro správu více souvisejících aplikací a sdílení kódu mezi nimi.
