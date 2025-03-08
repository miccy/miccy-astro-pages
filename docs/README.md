# Migrace portfolia z Jekyll na Astro v monorepo

Tato dokumentace popisuje plán a postup migrace osobního portfolia z Jekyll na Astro framework v rámci monorepo struktury. Modernější Astro framework poskytuje lepší výkon, flexibilitu a možnosti pro budoucí rozšiřování.

## Obsah dokumentace

1. **[TODO.md](./TODO.md)** - Kompletní seznam úkolů pro vytvoření monorepo s Astro projekty
2. **[MONOREPO.md](./MONOREPO.md)** - Detailní popis monorepo struktury a konfigurace Turborepo
3. **[IMPLEMENTATION.md](./IMPLEMENTATION.md)** - Podrobný plán implementace komponent a migrace dat
4. **[DEPLOYMENT.md](./DEPLOYMENT.md)** - Postup nasazení Astro projektu z monorepo na GitHub Pages
5. **[I18N.md](./I18N.md)** - Implementace vícejazyčnosti v Astro projektu

## Proč monorepo s Astro?

Monorepo struktura nabízí několik výhod:

- **Sdílení kódu** - Snadné sdílení komponent, konfigurací a utilit mezi projekty
- **Konzistence** - Jednotný styl a přístup napříč více projekty
- **Snadnější údržba** - Jednodušší aktualizace závislostí a refaktoring
- **Centralizovaná CI/CD** - Jednotné workflow pro všechny projekty

Astro 5 pak přináší:

- **Vynikající výkon** - Minimální JavaScript, optimalizované načítání
- **Content Collections API 2.0** - Lepší správa obsahu
- **Server Islands** - Progresivní vylepšení pro interaktivitu
- **Flexibilní integraci** s jinými technologiemi (React, Vue, Svelte)
- **Automatické generování OG obrázků** pro SEO

## Plánovaná struktura projektu

```
miccy-astro/
├── apps/
│   ├── portfolio/        # Hlavní portfolio web
│   ├── blog/             # Blog (budoucí)
│   └── docs/             # Dokumentační web (budoucí)
├── packages/
│   ├── ui/               # Sdílené UI komponenty
│   ├── config/           # Sdílené konfigurace
│   ├── utils/            # Sdílené utility funkce
│   └── content/          # Sdílené obsahové utility
├── .github/              # GitHub workflow konfigurace
└── scripts/              # Pomocné skripty
```

## Rychlý start

```bash
# Vytvořit nový monorepo
bunx create-turbo@latest miccy-astro

# Přejít do adresáře
cd miccy-astro

# Vytvořit první Astro aplikaci
cd apps
bunx create astro@latest portfolio

# Přidat potřebné integrace
cd portfolio
bunx astro add tailwind
bunx astro add mdx
bunx astro add sitemap

# Instalovat další závislosti
bun add astro-i18n-aut
```

## Hlavní funkce nového portfolia

- 🌓 Přepínání světlého/tmavého režimu
- 🌐 Podpora vícejazyčnosti (CS/EN)
- 📱 Responzivní design pro všechna zařízení
- 📄 Generování PDF verze portfolia
- 🚀 Automatické nasazení na GitHub Pages
- 🔍 SEO optimalizace a metadata
- ♻️ Sdílené komponenty pro budoucí projekty

## Plán rozšíření

1. **Porfolio** - První fáze migrace
2. **Blog** - Druhá fáze pro sdílení obsahu
3. **Dokumentace** - Automatické načítání dokumentace z GitHub projektů
4. **Experimentální projekty** - Playground pro testování nových technologií

## Technologie a nástroje

- **Astro 5** - Hlavní framework
- **TailwindCSS** - Stylování
- **TypeScript** - Typová bezpečnost
- **Turborepo** - Správa monorepo
- **Bun** - Rychlý JavaScript runtime a package manager
- **GitHub Actions** - CI/CD
- **astro-i18n-aut** - Vícejazyčnost
- **Content Collections** - Správa obsahu

## Migrace obsahu

1. **Extrakce dat z Jekyll** - Extrakce obsahu z původních Markdown souborů
2. **Transformace do komponent** - Přeměna na Astro komponenty
3. **Stylování s TailwindCSS** - Reimplementace designu
4. **Lokalizace** - Rozdělení obsahu na CS/EN verze

## Sledování výkonu

- **Lighthouse skóre** - Měření výkonu, dostupnosti, SEO
- **Core Web Vitals** - Optimalizace uživatelského zážitku
- **Testování napříč prohlížeči** - Zajištění kompatibility

## Licence

Tento projekt je licencován pod MIT licencí.
