# Plán implementace Astro portfolia v monorepo struktuře

Tento dokument popisuje detailní plán implementace pro migraci portfolia z Jekyll na Astro v rámci monorepo struktury.

## 1. Základní komponenty UI balíčku

### ThemeToggle komponenta

Komponenta pro přepínání mezi světlým a tmavým režimem:

```ts
// packages/ui/components/ThemeToggle/ThemeToggle.astro
---
---
<button id="themeToggle" aria-label="Přepnout téma">
  <svg width="20px" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
    <path class="sun" fill-rule="evenodd" d="M12 17.5a5.5 5.5 0 1 0 0-11 5.5 5.5 0 0 0 0 11zm0 1.5a7 7 0 1 0 0-14 7 7 0 0 0 0 14zm12-7a.8.8 0 0 1-.8.8h-2.4a.8.8 0 0 1 0-1.6h2.4a.8.8 0 0 1 .8.8zM4 12a.8.8 0 0 1-.8.8H.8a.8.8 0 0 1 0-1.6h2.5a.8.8 0 0 1 .8.8zm16.5-8.5a.8.8 0 0 1 0 1l-1.8 1.8a.8.8 0 0 1-1-1l1.7-1.8a.8.8 0 0 1 1 0zM6.3 17.7a.8.8 0 0 1 0 1l-1.7 1.8a.8.8 0 1 1-1-1l1.7-1.8a.8.8 0 0 1 1 0zM12 0a.8.8 0 0 1 .8.8v2.5a.8.8 0 0 1-1.6 0V.8A.8.8 0 0 1 12 0zm0 20a.8.8 0 0 1 .8.8v2.4a.8.8 0 0 1-1.6 0v-2.4a.8.8 0 0 1 .8-.8zM3.5 3.5a.8.8 0 0 1 1 0l1.8 1.8a.8.8 0 1 1-1 1L3.5 4.6a.8.8 0 0 1 0-1zm14.2 14.2a.8.8 0 0 1 1 0l1.8 1.7a.8.8 0 0 1-1 1l-1.8-1.7a.8.8 0 0 1 0-1z"/>
    <path class="moon" fill-rule="evenodd" d="M16.5 6A10.5 10.5 0 0 1 4.7 16.4 8.5 8.5 0 1 0 16.4 4.7l.1 1.3zm-1.7-2a9 9 0 0 1 .2 2 9 9 0 0 1-11 8.8 9.4 9.4 0 0 1-.8-.3c-.4 0-.8.3-.7.7a10 10 0 0 0 .3.8 10 10 0 0 0 9.2 6 10 10 0 0 0 4-19.2 9.7 9.7 0 0 0-.9-.3c-.3-.1-.7.3-.6.7a9 9 0 0 1 .3.8z"/>
  </svg>
</button>

<style>
  #themeToggle {
    border: 0;
    background: none;
    cursor: pointer;
    padding: 0;
  }
  .sun { fill: black; }
  .moon { fill: transparent; }

  :global(.dark) .sun { fill: transparent; }
  :global(.dark) .moon { fill: white; }
</style>

<script>
  document.addEventListener('astro:page-load', () => {
    const theme = (() => {
      if (typeof localStorage !== 'undefined' && localStorage.getItem('theme')) {
        return localStorage.getItem('theme');
      }
      if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
        return 'dark';
      }
      return 'light';
    })();

    if (theme === 'light') {
      document.documentElement.classList.remove('dark');
    } else {
      document.documentElement.classList.add('dark');
    }

    window.localStorage.setItem('theme', theme);

    const handleToggleClick = () => {
      const element = document.documentElement;
      element.classList.toggle("dark");

      const isDark = element.classList.contains("dark");
      localStorage.setItem("theme", isDark ? "dark" : "light");
    }

    document.getElementById("themeToggle").addEventListener("click", handleToggleClick);
  });
</script>
```

### LanguageSelector komponenta

Komponenta pro přepínání mezi jazyky:

```ts
// packages/ui/components/LanguageSelector/LanguageSelector.astro
---
---
<div class="language-select">
  <button class="lang-btn" data-lang="cs">🇨🇿 CS</button>
  <button class="lang-btn" data-lang="en">🇬🇧 EN</button>
</div>

<style>
  .language-select {
    display: flex;
    gap: 0.5rem;
  }
  .lang-btn {
    padding: 0.5rem;
    border: none;
    background: var(--background-color);
    color: var(--text-color);
    cursor: pointer;
    border-radius: 4px;
  }
  .lang-btn:hover {
    background: var(--hover-color);
  }
  .lang-btn.active {
    background: var(--active-color);
  }
</style>

<script>
  document.addEventListener('astro:page-load', () => {
    const lang = localStorage.getItem('lang') || 'cs';
    document.documentElement.lang = lang;

    const buttons = document.querySelectorAll('.lang-btn');
    buttons.forEach(btn => {
      if (btn.getAttribute('data-lang') === lang) {
        btn.classList.add('active');
      }

      btn.addEventListener('click', () => {
        const newLang = btn.getAttribute('data-lang');
        localStorage.setItem('lang', newLang);
        window.location.reload();
      });
    });
  });
</script>
```

### MainLayout komponenta

Základní layout pro stránky:

```ts
// packages/ui/layouts/MainLayout/MainLayout.astro
---
import { ThemeToggle } from '../../components/ThemeToggle';
import { LanguageSelector } from '../../components/LanguageSelector';

interface Props {
  title: string;
  description?: string;
  lang?: string;
}

const {
  title,
  description = "Miloš Macek - Portfolio",
  lang = "cs"
} = Astro.props;
---

<!doctype html>
<html lang={lang} class="scroll-smooth">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="description" content={description} />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="generator" content={Astro.generator} />
    <title>{title}</title>
  </head>
  <body class="min-h-screen bg-white dark:bg-slate-900 text-slate-900 dark:text-white">
    <header class="sticky top-0 z-10 bg-white dark:bg-slate-900 shadow-sm">
      <div class="container mx-auto py-4 px-6 flex justify-between items-center">
        <a href="/" class="text-xl font-bold">Miloš Macek</a>
        <nav class="flex items-center gap-6">
          <ul class="hidden md:flex space-x-6">
            <li><a href="#about" class="hover:text-primary-600">O mně</a></li>
            <li><a href="#experience" class="hover:text-primary-600">Zkušenosti</a></li>
            <li><a href="#projects" class="hover:text-primary-600">Projekty</a></li>
            <li><a href="#contact" class="hover:text-primary-600">Kontakt</a></li>
          </ul>
          <div class="flex items-center gap-4">
            <LanguageSelector />
            <ThemeToggle />
          </div>
        </nav>
      </div>
    </header>
    <main>
      <slot />
    </main>
    <footer class="bg-slate-100 dark:bg-slate-800 py-8">
      <div class="container mx-auto px-6">
        <div class="flex flex-col md:flex-row justify-between items-center">
          <p>© {new Date().getFullYear()} Miloš Macek</p>
          <div class="flex gap-4 mt-4 md:mt-0">
            <a href="https://github.com/miccy" target="_blank" rel="noopener noreferrer">
              GitHub
            </a>
            <a href="https://linkedin.com/in/miccy" target="_blank" rel="noopener noreferrer">
              LinkedIn
            </a>
          </div>
        </div>
      </div>
    </footer>
  </body>
</html>
```

## 2. TailwindCSS konfigurace

Sdílená konfigurace TailwindCSS:

```js
// packages/config/tailwind/index.js
const colors = require("tailwindcss/colors");

module.exports = {
  content: ["./src/**/*.{astro,html,js,jsx,md,mdx,svelte,ts,tsx,vue}"],
  darkMode: "class",
  theme: {
    extend: {
      colors: {
        primary: colors.indigo,
        secondary: colors.rose
      },
      typography: {
        DEFAULT: {
          css: {
            "code::before": {
              content: '""'
            },
            "code::after": {
              content: '""'
            }
          }
        }
      }
    }
  },
  plugins: [require("@tailwindcss/typography"), require("@tailwindcss/forms")]
};
```

## 3. Struktura portfolia

```
apps/portfolio/
├── src/
│   ├── components/    # Specifické komponenty portfolia
│   │   ├── About.astro
│   │   ├── Experience.astro
│   │   ├── Hero.astro
│   │   ├── Projects.astro
│   │   └── TechStack.astro
│   ├── content/       # Content Collections
│   │   ├── projects/  # Projekty
│   │   └── config.ts  # Konfigurace obsahu
│   ├── i18n/          # Překlady
│   │   ├── cs.json
│   │   └── en.json
│   ├── layouts/       # Specifické layouty
│   ├── pages/         # Stránky a routy
│   │   ├── index.astro
│   │   ├── [lang]/index.astro
│   │   └── cv.pdf.ts  # API endpoint pro generování PDF
│   └── styles/        # Styly specifické pro portfolio
└── public/            # Statické soubory
```

## 4. Konfigurace vícejazyčnosti

Použijeme knihovnu `astro-i18n-aut` pro správu překladu:

```bash
cd apps/portfolio
bun add astro-i18n-aut
```

Konfigurace:

```js
// apps/portfolio/astro.config.mjs
import { defineConfig } from "astro/config";
import mdx from "@astrojs/mdx";
import tailwind from "@astrojs/tailwind";
import sitemap from "@astrojs/sitemap";
import astroI18n from "astro-i18n-aut/integration";

export default defineConfig({
  integrations: [
    mdx(),
    tailwind({
      config: { path: "../../packages/config/tailwind/index.js" }
    }),
    sitemap(),
    astroI18n({
      defaultLanguage: "cs",
      supportedLanguages: ["cs", "en"]
    })
  ],
  site: "https://miccy.github.io"
});
```

## 5. Migrace obsahu z Jekyll

Pro migraci obsahu z Jekyll budeme postupovat následovně:

1. **Extrakce dat z Markdown** - Parsování původního souboru `index.md`
2. **Transformace do komponent** - Vytvoření Astro komponent pro každou sekci
3. **Přidání překladů** - Rozdělení obsahu do jazykových souborů

Příklad migrace sekce O mně:

```astro
// apps/portfolio/src/components/About.astro
---
import { useTranslations } from 'astro-i18n-aut';

const t = useTranslations(Astro);
---

<section id="about" class="py-20">
  <div class="container mx-auto px-6">
    <h2 class="text-3xl font-bold mb-8">{t('about.title')}</h2>
    <div class="prose dark:prose-invert lg:prose-lg mx-auto">
      <p>
        {t('about.description')}
      </p>
      <p>
        {t('about.skills')}
      </p>
    </div>
  </div>
</section>
```

S příslušnými překlady:

```json
// apps/portfolio/src/i18n/cs.json
{
  "about": {
    "title": "O mně",
    "description": "Jsem full stack vývojář...",
    "skills": "Mé hlavní dovednosti zahrnují..."
  }
}
```

```json
// apps/portfolio/src/i18n/en.json
{
  "about": {
    "title": "About Me",
    "description": "I am a full stack developer...",
    "skills": "My main skills include..."
  }
}
```

## 6. Implementace stránek

Hlavní stránka portfolia:

```astro
// apps/portfolio/src/pages/index.astro
---
import { MainLayout } from '@miccy/ui/layouts';
import Hero from '../components/Hero.astro';
import About from '../components/About.astro';
import TechStack from '../components/TechStack.astro';
import Experience from '../components/Experience.astro';
import Projects from '../components/Projects.astro';
import Contact from '../components/Contact.astro';
---

<MainLayout title="Miloš Macek - Portfolio">
  <Hero />
  <About />
  <TechStack />
  <Experience />
  <Projects />
  <Contact />
</MainLayout>
```

## 7. Implementace generování PDF

Pro generování PDF z portfolia použijeme API endpoint:

```ts
// apps/portfolio/src/pages/cv.pdf.ts
import type { APIRoute } from "astro";
import puppeteer from "puppeteer";

export const GET: APIRoute = async ({ params, request }) => {
  // Získat jazyk z query parametru nebo defaultně čeština
  const url = new URL(request.url);
  const lang = url.searchParams.get("lang") || "cs";

  // URL portfolia pro vykreslení
  const portfolioUrl = `https://miccy.github.io/${lang}`;

  try {
    // Spustit Puppeteer
    const browser = await puppeteer.launch();
    const page = await browser.newPage();

    // Nastavit viewport pro PDF
    await page.setViewport({ width: 1200, height: 1600 });

    // Načíst stránku
    await page.goto(portfolioUrl, { waitUntil: "networkidle2" });

    // Vygenerovat PDF
    const pdf = await page.pdf({
      format: "A4",
      printBackground: true,
      margin: { top: "1cm", right: "1cm", bottom: "1cm", left: "1cm" }
    });

    await browser.close();

    // Vrátit PDF jako odpověď
    return new Response(pdf, {
      status: 200,
      headers: {
        "Content-Type": "application/pdf",
        "Content-Disposition": `attachment; filename="milos-macek-cv-${lang}.pdf"`
      }
    });
  } catch (error) {
    console.error("Chyba při generování PDF:", error);
    return new Response("Chyba při generování PDF", { status: 500 });
  }
};
```

## 8. Nasazení na GitHub Pages

Pro nasazení na GitHub Pages použijeme GitHub Actions:

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

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

## 9. Další kroky

1. **Sledování výkonu** - Implementace Lighthouse CI pro měření výkonu
2. **Analytika** - Přidání privacy-friendly analytiky (např. Umami)
3. **RSS feed** - Vytvoření RSS feedu pro blog (po jeho implementaci)
4. **OG obrázky** - Automatické generování OG obrázků pro lepší sdílení

## Závěr

Tento implementační plán poskytuje podrobný návod, jak postupně migrovat Jekyll portfolio na Astro v rámci monorepo struktury. Plán zdůrazňuje využití sdílených komponent, konfigurací a utilit, což vede k udržitelnějšímu a škálovatelnějšímu řešení.
