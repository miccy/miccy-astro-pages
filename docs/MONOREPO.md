# Monorepo struktura pro Astro projekty

Tento dokument popisuje strukturu, konfiguraci a správu monorepo pro Astro projekty.

## Proč monorepo?

Monorepo řešení poskytuje několik výhod:

- **Sdílení kódu** - snadné sdílení komponent, utility funkcí a konfigurací
- **Konzistence** - jednotný styl a přístup napříč projekty
- **Jednodušší správa změn** - změny, které ovlivňují více projektů, lze provádět v jednom commitu
- **Centralizovaná CI/CD** - jednotné workflow pro všechny projekty
- **Snadnější refactoring** - možnost změnit sdílené části bez nutnosti aktualizovat několik repozitářů

## Struktura projektu

```
miccy-astro/
├── apps/                    # Jednotlivé aplikace
│   ├── portfolio/           # Hlavní portfolio web
│   ├── blog/                # Blog (budoucí)
│   └── docs/                # Dokumentační web (budoucí)
│
├── packages/                # Sdílené balíčky
│   ├── ui/                  # Sdílené UI komponenty
│   │   ├── components/      # Základní UI komponenty
│   │   ├── layouts/         # Sdílené layouty
│   │   └── styles/          # Sdílené styly
│   │
│   ├── config/              # Sdílené konfigurace
│   │   ├── eslint/          # ESLint konfigurace
│   │   ├── tailwind/        # TailwindCSS konfigurace
│   │   └── tsconfig/        # TypeScript konfigurace
│   │
│   ├── utils/               # Sdílené utility
│   │   ├── formatting/      # Formátování dat, textů
│   │   ├── i18n/            # Lokalizační utility
│   │   └── seo/             # SEO utility
│   │
│   └── content/             # Sdílené obsahové utility
│       ├── markdown/        # Zpracování Markdown souborů
│       ├── api/             # API utility pro načítání obsahu
│       └── schemas/         # Content schémata
│
├── scripts/                 # Pomocné skripty
│   ├── setup-project.js     # Skript pro inicializaci projektu
│   └── fetch-docs.js        # Skript pro načítání dokumentace
│
├── .github/                 # GitHub konfigurace
│   └── workflows/           # CI/CD workflow
│
├── turbo.json               # Konfigurace Turborepo
├── package.json             # Kořenový package.json
└── README.md                # Dokumentace
```

## Nastavení nového monorepo

### 1. Inicializace Turborepo

```bash
# Vytvořit nový Turborepo projekt
bunx create-turbo@latest miccy-astro

# Přejít do vytvořeného adresáře
cd miccy-astro

# Inicializovat Git (pokud není)
git init
```

### 2. Přidání první Astro aplikace

```bash
# Přejít do apps adresáře
cd apps

# Vytvořit nový Astro projekt
bunx create astro@latest portfolio

# Přidat TailwindCSS
cd portfolio
bunx astro add tailwind
```

### 3. Nastavení sdílených balíčků

```bash
# Vytvořit adresáře pro sdílené balíčky
cd ../../packages
mkdir -p ui/components config/tailwind utils/formatting content/markdown
```

## Sdílené balíčky

### UI balíček

UI balíček by měl obsahovat sdílené komponenty, které lze použít ve všech aplikacích:

```bash
cd packages/ui
bun init -y
```

Upravit `package.json`:

```json
{
  "name": "@miccy/ui",
  "version": "0.1.0",
  "type": "module",
  "exports": {
    ".": "./src/index.ts",
    "./components": "./src/components/index.ts",
    "./layouts": "./src/layouts/index.ts"
  }
}
```

### Config balíček

Config balíček poskytuje sdílené konfigurace:

```bash
cd ../config
bun init -y
```

Upravit `package.json`:

```json
{
  "name": "@miccy/config",
  "version": "0.1.0",
  "type": "module",
  "exports": {
    "./tailwind": "./tailwind/index.js",
    "./eslint": "./eslint/index.js",
    "./tsconfig": "./tsconfig/base.json"
  }
}
```

## Konfigurace Turborepo

Upravit `turbo.json` v kořenovém adresáři:

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {},
    "test": {
      "dependsOn": ["build"]
    }
  }
}
```

## Nastavení workspaces

Upravit kořenový `package.json`:

```json
{
  "name": "miccy-astro",
  "version": "0.0.0",
  "private": true,
  "workspaces": ["apps/*", "packages/*"],
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "lint": "turbo run lint",
    "test": "turbo run test"
  },
  "devDependencies": {
    "turbo": "latest"
  }
}
```

## CI/CD konfigurace

Vytvořit `.github/workflows/main.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1
        with:
          bun-version: latest
      - name: Install dependencies
        run: bun install
      - name: Build
        run: bun run build

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1
        with:
          bun-version: latest
      - name: Install dependencies
        run: bun install
      - name: Build
        run: bun run build
      # Přidat kroky pro nasazení na GitHub Pages nebo jiný hosting
```

## Sdílení typů

Pro TypeScript podporu napříč balíčky vytvořte `packages/tsconfig/base.json`:

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "display": "Default",
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true
  },
  "exclude": ["node_modules"]
}
```

## Instalace sdílených balíčků v aplikacích

```bash
cd apps/portfolio
bun add @miccy/ui @miccy/config
```

## Vytvoření skriptu pro stahování dokumentace

Vytvořte skript `scripts/fetch-docs.js`:

```javascript
import { execSync } from "child_process";
import fs from "fs";
import path from "path";

// Konfigurační soubor s cestami k repozitářům
const repos = [
  { url: "https://github.com/miccy/projekt1", docs: ["README.md", "docs/"] },
  { url: "https://github.com/miccy/projekt2", docs: ["README.md", "docs/"] }
];

// Adresář pro dokumentaci
const docsDir = path.resolve("apps/docs/src/content/docs");

// Vytvořit adresář, pokud neexistuje
if (!fs.existsSync(docsDir)) {
  fs.mkdirSync(docsDir, { recursive: true });
}

// Dočasný adresář pro klonování
const tmpDir = path.resolve(".tmp");
if (!fs.existsSync(tmpDir)) {
  fs.mkdirSync(tmpDir, { recursive: true });
}

// Stáhnout dokumentaci
for (const repo of repos) {
  const repoName = repo.url.split("/").pop();
  const repoDir = path.join(tmpDir, repoName);

  console.log(`Stahování dokumentace z ${repo.url}...`);

  // Klonovat repozitář
  if (fs.existsSync(repoDir)) {
    execSync(`cd ${repoDir} && git pull`);
  } else {
    execSync(`git clone ${repo.url} ${repoDir} --depth 1`);
  }

  // Vytvořit cílový adresář
  const targetDir = path.join(docsDir, repoName);
  if (!fs.existsSync(targetDir)) {
    fs.mkdirSync(targetDir, { recursive: true });
  }

  // Kopírovat soubory
  for (const doc of repo.docs) {
    const sourcePath = path.join(repoDir, doc);
    if (fs.existsSync(sourcePath)) {
      if (fs.statSync(sourcePath).isDirectory()) {
        // Kopírovat adresář
        execSync(`cp -r ${sourcePath}/* ${targetDir}`);
      } else {
        // Kopírovat soubor
        execSync(`cp ${sourcePath} ${targetDir}`);
      }
    }
  }

  console.log(`Dokumentace z ${repoName} byla úspěšně stažena.`);
}

console.log("Stahování dokumentace dokončeno.");
```

## Závěr

Tato monorepo struktura poskytuje solidní základ pro vývoj více souvisejících Astro aplikací. Počáteční nastavení může vyžadovat více času, ale dlouhodobé výhody v podobě sdílení kódu, konzistence a snadnější údržby za to stojí.

Při rozrůstání projektu lze strukturu dále rozšiřovat o další aplikace nebo sdílené balíčky podle potřeby.
