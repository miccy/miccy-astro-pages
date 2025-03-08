# TODO: Migrace portfolia z Jekyll na Astro

## 1. Nastavení nového monorepo

- [ ] Vytvořit nový privátní repozitář `miccy-astro`
- [ ] Nastavit Turborepo strukturu
  ```bash
  bunx create-turbo@latest miccy-astro
  ```
- [ ] Vytvořit základní strukturu
  ```
  miccy-astro/
  ├── apps/
  │   └── portfolio/        # Hlavní portfolio
  ├── packages/
  │   ├── ui/               # Sdílené UI komponenty
  │   ├── config/           # Sdílené konfigurace
  │   └── utils/            # Sdílené utility funkce
  ```
- [ ] Nastavit sdílené konfigurace
  - [ ] ESLint, Prettier
  - [ ] TypeScript
  - [ ] TailwindCSS

## 2. Nastavení Astro projektu v portfolio

- [ ] Vytvořit nový Astro projekt s nejnovější verzí (5.x)
  ```bash
  cd apps
  bunx create astro@latest portfolio
  ```
- [ ] Instalace základních balíčků
  ```bash
  cd portfolio
  bunx astro add tailwind
  bunx astro add mdx
  bunx astro add sitemap
  bun add astro-i18n-aut # nebo jiná i18n knihovna
  ```
- [ ] Nastavení TypeScript konfigurace pro Astro
- [ ] Nastavení `.env` a `.env.example` pro konfiguraci
- [ ] Vytvoření `.gitignore` a `.editorconfig`

## 3. Implementace základních komponent v packages/ui

- [ ] Vytvoření sdílených komponent
  - [ ] ThemeToggle (tmavý/světlý režim)
  - [ ] LanguageSelector (CS/EN)
  - [ ] Header a navigace
  - [ ] Footer
  - [ ] Card komponenty
  - [ ] Button komponenty

## 4. Implementace portfolia

- [ ] Vytvoření hlavního layoutu
- [ ] Migrace obsahu z Jekyll
  - [ ] O mně sekce
  - [ ] Tech Stack
  - [ ] Vzdělání
  - [ ] Pracovní zkušenosti
  - [ ] Projekty
  - [ ] Kontakt
- [ ] Implementace vícejazyčnosti (CS/EN)
- [ ] Stylování pomocí TailwindCSS
- [ ] Implementace dark/light mode

## 5. Konfigurace a nasazení

- [ ] Nastavení GitHub Actions pro CI/CD
  - [ ] Vytvoření workflow souboru
  - [ ] Konfigurace pro automatické nasazení
- [ ] Konfigurace `astro.config.mjs` pro GitHub Pages
- [ ] Testovací build a nasazení

## 6. Rozšíření projektu (budoucí plány)

- [ ] Vytvoření blog aplikace
  ```bash
  cd apps
  bunx create astro@latest blog
  ```
- [ ] Vytvoření dokumentační aplikace
  ```bash
  cd apps
  bunx create astro@latest docs
  ```
- [ ] Implementace automatického načítání dokumentace z ostatních projektů
- [ ] Integrační skripty pro načítání Markdown souborů

## 7. Sdílený obsah a content pipeline

- [ ] Vytvoření packages/content
- [ ] Implementace Content Collection API
- [ ] Skripty pro stahování a transformaci Markdown
- [ ] Generování metadat pro obsah

## 8. Monitoring a analytika

- [ ] Integrace Umami nebo jiné privacy-friendly analytiky
- [ ] Monitorování výkonu webu
- [ ] Testování Core Web Vitals

## 9. Další funkce

- [ ] Generování PDF verze portfolia
- [ ] Implementace RSS feedu pro blog
- [ ] SEO optimalizace
- [ ] Automatické generování OG obrázků

## 10. Cleanup a dokumentace

- [ ] Kompletní dokumentace projektu
- [ ] Sepsání contribution guidelines
- [ ] Zveřejnění potřebných částí repozitáře
