# Angular + SSR + Ngx-translate <!-- omit in toc -->

This project was generated using [Angular CLI](https://github.com/angular/angular-cli) version 19.2.5.

🌐 Available languages:

- 🇺🇦 [Українська](README.uk.md)
- 🇺🇸 [English](README.md)

## Introduction <!-- omit in toc -->

This demonstration project provides a starting point for building an Angular application with server-side rendering (SSR) and internationalization (i18n) using ngx-translate.

Its primary goal is to investigate whether a site is properly indexed by search engines when using SSR in conjunction with ngx-translate.

This is part one. Part two, which focuses on SEO optimization, can be found here: [Angular + SSR + Ngx-translate + SEO](https://github.com/One-LifeMan/Angular-19-SSR-ngx-translate-SEO?tab=readme-ov-file#angular--ssr--ngx-translate--seo-)

**Key Technologies:**

- **Angular:** A powerful JavaScript framework for building single-page applications.
- **Server-Side Rendering (SSR):** A technique that renders the application on the server, improving SEO, initial load time, and accessibility.
- **ngx-translate:** An internationalization library for Angular that simplifies the process of translating your application into multiple languages.

**Table of contents:** <!-- omit in toc -->

- [1. Create a new workspace](#1-create-a-new-workspace)
- [2. Add linters and code formatter (optionally)](#2-add-linters-and-code-formatter-optionally)
  - [2.1. Installing dependencies](#21-installing-dependencies)
  - [2.2. Create configuration files](#22-create-configuration-files)
  - [2.3. Configure settings.json](#23-configure-settingsjson)
  - [2.4. Formatting the Project](#24-formatting-the-project)
- [3. Create an initial project structure](#3-create-an-initial-project-structure)
  - [3.1. Generate components \& environments](#31-generate-components--environments)
  - [3.2. Configure environments](#32-configure-environments)
  - [3.3. Add a "watch" Environment Configuration](#33-add-a-watch-environment-configuration)
    - [3.3.1. Create environment.watch.ts](#331-create-environmentwatchts)
    - [3.3.2. Edit angular.json](#332-edit-angularjson)
    - [3.3.3. Edit package.json](#333-edit-packagejson)
  - [3.4. Configure paths](#34-configure-paths)
  - [3.5. Make changes to components](#35-make-changes-to-components)
  - [3.6. Implementing styles (optional)](#36-implementing-styles-optional)
    - [3.6.1. Install @csstools/normalize.css](#361-install-csstoolsnormalizecss)
    - [3.6.2. Create style files (optional)](#362-create-style-files-optional)
- [4. Add translation](#4-add-translation)
  - [4.1. Installing Dependencies](#41-installing-dependencies)
  - [4.2. Creating Translation Files](#42-creating-translation-files)
  - [4.3. Configuring Locales](#43-configuring-locales)
  - [4.4. Setting up Loaders](#44-setting-up-loaders)
  - [4.5. Implementing Translation in Components](#45-implementing-translation-in-components)
  - [4.6. Updating the Language Switcher](#46-updating-the-language-switcher)
  - [4.7. Critical Configuration: Disabling Prerendering](#47-critical-configuration-disabling-prerendering)
- [5. Acknowledgements](#5-acknowledgements)

## 1. Create a new workspace

```bash
ng new angular-ssr-ngx-translate --package-manager=pnpm
```

- Which stylesheet format would you like to use?
  ✔️ Sass (SCSS)
- Do you want to enable Server-Side Rendering (SSR) and Static Site Generation (SSG/Prerendering)?
  ✔️ Yes
- Would you like to use the Server Routing and App Engine APIs (Developer Preview) for this server application?
  ✔️ No

## 2. Add linters and code formatter (optionally)

### 2.1. Installing dependencies

```bash
pnpm add -D eslint @eslint/js angular-eslint typescript-eslint eslint-config-prettier eslint-plugin-prettier prettier prettier-plugin-organize-attributes @trivago/prettier-plugin-sort-imports stylelint stylelint-config-standard-scss stylelint-scss stylelint-prettier stylelint-order
```

### 2.2. Create configuration files

- .prettierignore
- .prettierrc
- .stylelintrc.json
- eslint.config.js

### 2.3. Configure settings.json

```json
{
  // === FORMATTER & LINTING ===
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "prettier.requireConfig": true,
  "prettier.configPath": ".prettierrc",
  // -------------------------------------
  "eslint.enable": true,
  "eslint.format.enable": true,
  "eslint.validate": ["javascript", "typescript", "html"],
  "eslint.useFlatConfig": true,
  "eslint.workingDirectories": ["../src"],
  "[javascript]": {
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": "explicit"
    }
  },
  "[typescript]": {
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": "explicit"
    }
  },
  // -------------------------------------
  "stylelint.enable": true,
  "stylelint.validate": ["css", "scss"],
  "[scss]": {
    "editor.codeActionsOnSave": {
      "source.fixAll.stylelint": "explicit"
    }
  },
  "[css]": {
    "editor.codeActionsOnSave": {
      "source.fixAll.stylelint": "explicit"
    }
  }
}
```

### 2.4. Formatting the Project

```bash
pnpx prettier --write .
```

## 3. Create an initial project structure

### 3.1. Generate components & environments

```bash
ng g c layouts/client-layout
ng g c pages/client/home
ng g c pages/client/about
ng g c pages/not-found-page
ng g c ui/client/header
ng g c ui/client/footer
ng g c ui/client/menu
ng g c ui/client/language-switcher
ng generate environments
```

### 3.2. Configure environments

**src\environments\environment.ts**

```ts
export const environment = {
  production: true,
  languages: ["en", "uk"],
  appUrl: "/",
};
```

**src\environments\environment.development.ts**

```ts
export const environment = {
  production: true,
  languages: ["en", "uk"],
  appUrl: "http://localhost:4200/",
};
```

### 3.3. Add a "watch" Environment Configuration

**Rationale:**
When running both the frontend and backend in development mode using the `watch` and `serve:ssr:angular-ssr-ngx-translate` scripts, a mismatch can occur between `environment.appUrl` (used by the frontend, typically "http://localhost:4200/") and the server's address (typically "http://localhost:4000/"). This mismatch can cause issues with server-side requests.

To address this, we create a separate environment configuration specifically for the `watch` script. This configuration sets `environment.appUrl` to "http://localhost:4000/", ensuring consistency with the development server.

While other solutions may exist, this approach provides a simple way to manage this inconsistency during development.

#### 3.3.1. Create environment.watch.ts

```bash
touch src/environments/environment.watch.ts
```

```ts
export const environment = {
  production: true,
  languages: ["en", "uk"],
  appUrl: "http://localhost:4000/",
};
```

#### 3.3.2. Edit angular.json

Add the "watch" configuration to projects.angular-ssr-ngx-translate.architect.build.configurations.
These are the same settings as for "development", except that "src/environments/environment.ts" is changed to "src/environments/environment.watch.ts" instead of "src/environments/environment.development.ts".

```json
{
  ...
  "projects": {
    "angular-ssr-ngx-translate": {
      ...
      "architect": {
        "build": {
          ...
          "configurations": {
            ...
            "watch": {
              "optimization": false,
              "extractLicenses": false,
              "sourceMap": true,
              "fileReplacements": [
                {
                  "replace": "src/environments/environment.ts",
                  "with": "src/environments/environment.watch.ts"
                }
              ]
            }
          }
          ...
        }
        ...
      }
    }
  }
}
```

#### 3.3.3. Edit package.json

replace "development" with "watch" in the script of the same name "watch"

```json
...
"watch": "ng build --watch --configuration watch",
...
```

### 3.4. Configure paths

**Create export files**

```bash
touch src/app/layouts/index.ts
touch src/app/pages/index.ts
```

**src\app\layouts\index.ts**

```ts
export * from "./client-layout/client-layout.component";
```

**src\app\pages\index.ts**

```ts
export * from "./client/home/home.component";
export * from "./client/about/about.component";
export * from "./not-found-page/not-found-page.component";
```

**tsconfig.json**

```json
{
  ...
  "compilerOptions": {
    ...
    "baseUrl": "./",
    "paths": {
      "@app/layouts": ["src/app/layouts/index.ts"],
      "@app/pages": ["src/app/pages/index.ts"],
    }
  }
  ...
}
```

**app.routes.ts**

```ts
import { Routes } from "@angular/router";

const loadClientLayout = () => import("@app/layouts").then(c => c.ClientLayoutComponent);

const loadHome = () => import("@app/pages").then(c => c.HomeComponent);
const loadAbout = () => import("@app/pages").then(c => c.AboutComponent);

const loadNotFoundPage = () => import("@app/pages").then(c => c.NotFoundPageComponent);

export const routes: Routes = [
  {
    path: "",
    loadComponent: loadClientLayout,
    children: [
      { path: "", loadComponent: loadHome },
      { path: "about", loadComponent: loadAbout },
      { path: "404", loadComponent: loadNotFoundPage },
    ],
  },
  { path: "**", redirectTo: "/404" },
];
```

### 3.5. Make changes to components

**src\app\app.component.html**

```html
<router-outlet></router-outlet>
```

**src\app\layouts\client-layout\client-layout.component.html**

Honestly, I could have done without client-layout.component, but at first I was thinking of creating admin-layout.component, and then my plans changed :)

```html
<app-header></app-header>
<router-outlet></router-outlet>
<app-footer></app-footer>
```

**src\app\layouts\client-layout\client-layout.component.ts**

```ts
...
imports: [HeaderComponent, FooterComponent, RouterOutlet],
...
```

**src\app\ui\client\footer\footer.component.html**

```html
<footer class="footer">
  <div class="container">
    <div class="footer__inner">
      <p>footer works!</p>
    </div>
  </div>
</footer>
```

**src\app\ui\client\header\header.component.html**

```html
<header class="header">
  <app-menu></app-menu>
  <app-language-switcher></app-language-switcher>
</header>
```

**src\app\ui\client\header\header.component.ts**

```ts
...
  imports: [MenuComponent, LanguageSwitcherComponent],
...
```

**src\app\ui\client\menu\menu.component.html**

```html
<nav class="nav">
  <ul class="nav__list">
    <li class="nav__item">
      <a
        class="nav__link"
        [routerLinkActiveOptions]="{ exact: true }"
        routerLink="/"
        routerLinkActive="active"
        >Home</a
      >
    </li>
    <li class="nav__item">
      <a
        class="nav__link"
        [routerLinkActiveOptions]="{ exact: true }"
        routerLink="/about"
        routerLinkActive="active"
        >About</a
      >
    </li>
  </ul>
</nav>
```

**src\app\ui\client\menu\menu.component.ts**

```ts
...
  imports: [RouterLink, RouterLinkActive],
...
```

**src\app\ui\client\language-switcher\language-switcher.component.html**

```html
<ul class="lang-switcher">
  @for (lang of languages; track lang) {
  <li class="lang-switcher__item">
    <a
      class="lang-switcher__link"
      [routerLinkActiveOptions]="{ exact: true }"
      routerLink="/{{ lang }}/{{ currentUrl }}"
      routerLinkActive="active"
    >
      {{ lang }}
    </a>
  </li>
  } @empty {
  <li>List of supported languages not found</li>
  }
</ul>
```

**src\app\ui\client\language-switcher\language-switcher.component.ts**

```ts
import { environment } from "../../../../environments/environment";
import { filter } from "../../../shared/rxjs";
import { Component, inject } from "@angular/core";
import { NavigationEnd, Router, RouterLink, RouterLinkActive } from "@angular/router";

@Component({
  selector: "app-language-switcher",
  imports: [RouterLink, RouterLinkActive],
  templateUrl: "./language-switcher.component.html",
  styleUrl: "./language-switcher.component.scss",
})
export class LanguageSwitcherComponent {
  private readonly router = inject(Router);
  languages = environment.languages;
  protected currentUrl = "";

  constructor() {
    this.router.events
      .pipe(filter(event => event instanceof NavigationEnd))
      .subscribe(() => this.setCurrentUrl());
  }

  private setCurrentUrl = () => {
    // Later here will be the language switching logic
  };
}
```

### 3.6. Implementing styles (optional)

#### 3.6.1. Install @csstools/normalize.css

`@csstools/normalize.css` is a modern npm-packaged version of [Normalize.css](https://necolas.github.io/normalize.css/), which standardizes browser styles, ensuring a unified appearance of HTML elements across browsers.

```bash
pnpm add @csstools/normalize.css
```

#### 3.6.2. Create style files (optional)

```bash
mkdir src/app/styles
touch src/app/styles/_variables.scss
touch src/app/styles/_theme.scss
```

**src/app/styles/variables.scss**

```scss
:root {
  --gap-small: 4px;
  --gap-medium: 8px;
}
```

**src/app/styles/theme.scss**

```scss
:root {
  --primary-color: #ccc;
  --on-primary-color: #000;
  --secondary-color: #ffffe0;
  --on-secondary-color: #000;
  --accent-color: #ffa500;
}

@media (prefers-color-scheme: dark) {
  :root {
    --primary-color: #191919;
    --on-primary-color: #ccc;
    --secondary-color: #131313;
    --on-secondary-color: #ccc;
    --accent-color: #ffa500;
  }
}
```

**src\styles.scss**

```scss
/* You can add global styles to this file, and also import other style files */

@use "@csstools/normalize.css";
@use "./app/styles/theme";
@use "./app/styles/variables";

*,
*::before,
*::after {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}

body {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  color: var(--on-primary-color);
  background-color: var(--primary-color);
}

ul,
ol {
  list-style: none;
}

a {
  color: inherit;
  text-decoration: none;
}

@media (hover: hover) {
  a:hover {
    color: var(--accent-color);
  }
}

.container {
  max-width: 1220px;
  padding: 0 10px;
}
```

**src\app\layouts\client-layout\client-layout.component.scss**

```scss
:host {
  display: flex;
  flex-direction: column;
  min-height: 100dvh;
}

.container {
  flex: 1 1 0;
}
```

**src\app\ui\client\footer\footer.component.scss**

```scss
.footer {
  color: var(--primary-color);
  background-color: var(--on-primary-color);

  &__inner {
  }
}
```

**src\app\ui\client\header\header.component.scss**

```scss
.header {
  color: var(--on-secondary-color);
  background-color: var(--secondary-color);

  &__inner {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    align-items: center;
    justify-content: space-between;
  }
}
```

**src\app\ui\client\menu\menu.component.scss**

```scss
.nav {
  &__list {
    display: flex;
    flex-wrap: wrap;
    gap: var(--gap-small);
    align-items: center;
  }

  &__item {
  }

  &__link {
    display: flex;
    padding-top: var(--gap-small);
    padding-right: var(--gap-medium);
    padding-bottom: var(--gap-small);
    padding-left: var(--gap-medium);

    &.active {
      color: var(--accent-color);
    }
  }
}

@media (hover: hover) {
  .nav {
    &__link {
      &:hover {
        color: var(--primary-color);
        background-color: var(--on-primary-color);
      }
    }
  }
}
```

**src\app\ui\client\language-switcher\language-switcher.component.scss**

```scss
.lang-switcher {
  display: flex;
  flex-wrap: wrap;
  gap: var(--gap-small);

  &__item {
  }

  &__link {
    padding: var(--gap-small);

    &.active {
      color: var(--accent-color);
    }
  }
}

@media (hover: hover) {
  .lang-switcher {
    &__item {
    }

    &__link {
      &:hover {
        color: var(--primary-color);
        background-color: var(--on-primary-color);
      }
    }
  }
}
```

## 4. Add translation

@ngx-translate/core:

- This is the core package for internationalization (i18n) in Angular.
- It provides the infrastructure for managing translations in your application.
- It allows you to load, store, and display translations in different languages.

@ngx-translate/http-loader:

- This is a helper package for @ngx-translate/core.
- It allows you to load translation files from the server via HTTP requests.
- This is useful for dynamically loading translations based on the selected language.

@gilsdav/ngx-translate-router:

- This package allows you to translate route URLs in your Angular application.
- This is useful for creating multilingual URLs that improve SEO and usability.
- Intercepts Router initialization and translates each route path.

@gilsdav/ngx-translate-router-http-loader:

- This is a helper package for @gilsdav/ngx-translate-router.
- It allows you to download route translation files from the server via HTTP requests.
- Used to download translations for @gilsdav/ngx-translate-router.

### 4.1. Installing Dependencies

```bash
pnpm add @ngx-translate/core @ngx-translate/http-loader @gilsdav/ngx-translate-router @gilsdav/ngx-translate-router-http-loader
```

### 4.2. Creating Translation Files

```bash
mkdir public/i18n
touch public/i18n/en.json
touch public/i18n/uk.json
```

**public\i18n\en.json**

```json
{
  "NAV": {
    "HOME": "Home",
    "ABOUT": "About"
  },
  "CONTENT": {
    "HOME": "home works!",
    "ABOUT": "about works!",
    "404": "Page not found"
  }
}
```

**public\i18n\uk.json**

```json
{
  "NAV": {
    "HOME": "Головна",
    "ABOUT": "Про нас"
  },
  "CONTENT": {
    "HOME": "home працює!",
    "ABOUT": "about працює!",
    "404": "Сторінку не знайдено"
  }
}
```

### 4.3. Configuring Locales

```bash
touch public/locales.json
```

**public\locales.json**

```json
{
  "locales": ["en", "uk"],
  "prefix": "ROUTES."
}
```

### 4.4. Setting up Loaders

We will need Loaders Factory - functions that will load translation and locale files. So let's create them right away.

```bash
mkdir src/app/core
touch src/app/core/translate.config.ts
```

**src\app\core\translate.config.ts**

```ts
import { Location } from "@angular/common";
import { HttpClient } from "@angular/common/http";
import { LocalizeRouterSettings } from "@gilsdav/ngx-translate-router";
import { LocalizeRouterHttpLoader } from "@gilsdav/ngx-translate-router-http-loader";
import { TranslateService } from "@ngx-translate/core";
import { TranslateHttpLoader } from "@ngx-translate/http-loader";
import { environment } from "src/environments/environment";

export const httpLoaderFactory: (http: HttpClient) => TranslateHttpLoader = (http: HttpClient) =>
  new TranslateHttpLoader(http, `${environment.appUrl}i18n/`, ".json");

export const localizeLoaderFactory = (
  translate: TranslateService,
  location: Location,
  settings: LocalizeRouterSettings,
  http: HttpClient,
) => {
  return new LocalizeRouterHttpLoader(
    translate,
    location,
    settings,
    http,
    `${environment.appUrl}locales.json`,
  );
};
```

**src\app\app.config.ts**

```ts
import { routes } from "./app.routes";
import { httpLoaderFactory, localizeLoaderFactory } from "./core/translate.config";
import { Location } from "@angular/common";
import { HttpClient, provideHttpClient, withFetch } from "@angular/common/http";
import { ApplicationConfig, provideZoneChangeDetection } from "@angular/core";
import { provideClientHydration, withEventReplay } from "@angular/platform-browser";
import { provideRouter, withDisabledInitialNavigation } from "@angular/router";
import {
  LocalizeParser,
  LocalizeRouterSettings,
  withLocalizeRouter,
} from "@gilsdav/ngx-translate-router";
import { provideTranslateService, TranslateLoader, TranslateService } from "@ngx-translate/core";

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(
      routes,
      withDisabledInitialNavigation(),
      withLocalizeRouter(routes, {
        parser: {
          provide: LocalizeParser,
          useFactory: localizeLoaderFactory,
          deps: [TranslateService, Location, LocalizeRouterSettings, HttpClient],
        },
        initialNavigation: true,
      }),
    ),
    provideClientHydration(withEventReplay()),
    provideHttpClient(withFetch()),
    provideTranslateService({
      defaultLanguage: "en",
      loader: {
        provide: TranslateLoader,
        useFactory: httpLoaderFactory,
        deps: [HttpClient],
      },
    }),
  ],
};
```

Make sure that:

- imported Location because Angular doesn't import it automatically
- withDisabledInitialNavigation() inside provideRouter
- added provideHttpClient(withFetch())

### 4.5. Implementing Translation in Components

**src\app\ui\client\menu\menu.component.ts**

Import TranslatePipe, LocalizeRouterPipe in component

```ts
  imports: [RouterLink, RouterLinkActive, TranslatePipe, LocalizeRouterPipe],
```

**src\app\ui\client\menu\menu.component.html**

Add pipe translate — for the content to be translated and localize — to routerLink to add the language code to the path

```html
<nav class="nav">
  <ul class="nav__list">
    <li class="nav__item">
      <a
        class="nav__link"
        [routerLink]="['/' | localize]"
        [routerLinkActiveOptions]="{ exact: true }"
        routerLinkActive="active"
      >
        {{ "NAV.HOME" | translate }}
      </a>
    </li>
    <li class="nav__item">
      <a
        class="nav__link"
        [routerLink]="['about' | localize]"
        [routerLinkActiveOptions]="{ exact: true }"
        routerLinkActive="active"
      >
        {{ "NAV.ABOUT" | translate }}
      </a>
    </li>
  </ul>
</nav>
```

Repeat this with the other components

**src\app\pages\client\home\home.component.html**

```html
<p>{{ "CONTENT.HOME" | translate }}</p>
```

**src\app\pages\client\about\about.component.html**

```html
<p>{{ "CONTENT.ABOUT" | translate }}</p>
```

**src\app\pages\not-found-page\not-found-page.component.html**

```html
<p>{{ "CONTENT.404" | translate }}</p>
```

### 4.6. Updating the Language Switcher

We implement the setCurrentUrl method.

**src\app\ui\client\language-switcher\language-switcher.component.ts**

```ts
import { environment } from "../../../../environments/environment";
import { filter } from "../../../shared/rxjs";
import { Component, inject } from "@angular/core";
import { NavigationEnd, Router, RouterLink, RouterLinkActive } from "@angular/router";
import { LocalizeRouterService } from "@gilsdav/ngx-translate-router";

@Component({
  selector: "app-language-switcher",
  imports: [RouterLink, RouterLinkActive],
  templateUrl: "./language-switcher.component.html",
  styleUrl: "./language-switcher.component.scss",
})
export class LanguageSwitcherComponent {
  private readonly router = inject(Router);
  private readonly localizeRouterService = inject(LocalizeRouterService); // <---
  languages = environment.languages;
  protected currentUrl = "";

  constructor() {
    this.setCurrentUrl(); // <---

    this.router.events
      .pipe(filter(event => event instanceof NavigationEnd))
      .subscribe(() => this.setCurrentUrl());
  }

  private setCurrentUrl = () => {
    this.currentUrl = this.router.url
      .replace("/" + this.localizeRouterService.parser.currentLang, "")
      .split("?")[0]; // <---
  };
}
```

### 4.7. Critical Configuration: Disabling Prerendering

**Important:** In `angular.json`, ensure that `prerender` is set to `false`. Setting it to `true` will cause build errors because HTTP requests to translation files (e.g., `en.json`) will fail, as the development server is not running during the build process.

## 5. Acknowledgements

I would like to thank the following people:

- [Olivier Combe](https://github.com/ocombe) and [Andreas Loew](https://github.com/CodeAndWeb) for creating ngx-translate.
- [David Gilson](https://github.com/gilsdav) for creating ngx-translate-router.
- [Robert Isaac](https://robert-isaac.medium.com/) for his article ["Best Practices for Angular Internationalization with SSR"](https://robert-isaac.medium.com/best-practices-for-angular-internationalization-with-ssr-384a98ee672a).

---

In the next part, SEO will be added, a few more pages with dynamic data to see how they (the pages) and the data that will come from the server will be localized.

[Angular + SSR + Ngx-translate + SEO](https://github.com/One-LifeMan/Angular-19-SSR-ngx-translate-SEO?tab=readme-ov-file#angular--ssr--ngx-translate--seo-)
