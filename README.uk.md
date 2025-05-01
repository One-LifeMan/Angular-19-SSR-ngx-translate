# Angular + SSR + Ngx-translate <!-- omit in toc -->

Цей проєкт було створено за допомогою [Angular CLI](https://github.com/angular/angular-cli) версії 19.2.5.

🌐 Доступні мови:

- 🇺🇦 [Українська](README.uk.md)
- 🇺🇸 [English](README.md)

## Вступ <!-- omit in toc -->

Цей демонстраційний проект надає відправну точку для створення Angular-додатку з рендерингом на стороні сервера (SSR) та інтернаціоналізацією (i18n) за допомогою ngx-translate.

Його основна мета — дослідити, чи правильно індексується сайт пошуковими системами при використанні SSR разом з ngx-translate.

Це перша частина. Другу частину, яка зосереджена на SEO-оптимізації, можна знайти тут: [Angular + SSR + Ngx-translate + SEO](https://github.com/One-LifeMan/Angular-19-SSR-ngx-translate-SEO?tab=readme-ov-file#angular--ssr--ngx-translate--seo-)

**Ключові технології:**

- **Angular:** Потужний JavaScript-фреймворк для створення односторінкових додатків.
- **Серверний рендеринг (SSR):** Метод, який рендерить застосунок на сервері, покращуючи SEO, час початкового завантаження та доступність.
- **ngx-translate:** Бібліотека інтернаціоналізації для Angular, яка спрощує процес перекладу вашого застосунку кількома мовами.
- **Зміст:** <!-- omit in toc -->

## 1. Створити нову робочу область

```bash
ng new angular-ssr-ngx-translate --package-manager=pnpm
```

- Which stylesheet format would you like to use?
  ✔️ Sass (SCSS)
- Do you want to enable Server-Side Rendering (SSR) and Static Site Generation (SSG/Prerendering)?
  ✔️ Yes
- Would you like to use the Server Routing and App Engine APIs (Developer Preview) for this server application?
  ✔️ No

## 2. Додати лінтери та форматувальник коду (за бажанням)

### 2.1. Встановити залежності

```bash
pnpm add -D eslint @eslint/js angular-eslint typescript-eslint eslint-config-prettier eslint-plugin-prettier prettier prettier-plugin-organize-attributes @trivago/prettier-plugin-sort-imports stylelint stylelint-config-standard-scss stylelint-scss stylelint-prettier stylelint-order
```

### 2.2. Створити файли конфігурації

- .prettierignore
- .prettierrc
- .stylelintrc.json
- eslint.config.js

### 2.3. Налаштувати settings.json

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

### 2.4. Відформатувати проект

```bash
pnpx prettier --write .
```

## 3. Створити початкову структуру

### 3.1. Згенерувати компоненти та середовища

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

### 3.2. Налаштувати середовища

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

### 3.3. Додати конфігурацію середовища "watch"

**Обґрунтування:**
Під час запуску фронтенду та бекенду в режимі розробки за допомогою скриптів `watch` та `serve:ssr:angular-ssr-ngx-translate` може виникнути невідповідність між `environment.appUrl` (використовується фронтендом, зазвичай "http://localhost:4200/") та адресою сервера (зазвичай "http://localhost:4000/"). Ця невідповідність може спричинити проблеми із запитами на стороні сервера.

Щоб вирішити цю проблему, ми створюємо окрему конфігурацію середовища спеціально для скрипта `watch`. Ця конфігурація встановлює `environment.appUrl` на "http://localhost:4000/", забезпечуючи узгодженість із сервером розробки.

Хоча можуть існувати й інші рішення, цей підхід забезпечує простий спосіб керування цією невідповідністю під час розробки.

#### 3.3.1. Створити environment.watch.ts

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

#### 3.3.2. Редагувати angular.json

Додайте конфігурацію "watch" до projects.angular-ssr-ngx-translate.architect.build.configurations.
Це ті ж самі налаштування, що й для "development", за винятком того, що "src/environments/environment.ts" змінено на "src/environments/environment.watch.ts" замість "src/environments/environment.development.ts".

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

#### 3.3.3. Редагувати package.json

Замінити "development" на "watch" у скрипті з однойменною назвою "watch"

```json
...
"watch": "ng build --watch --configuration watch",
...
```

### 3.4. Налаштувати шляхи

**Створення експортних файлів**

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

### 3.5. Внести зміни до компонентів

**src\app\app.component.html**

```html
<router-outlet></router-outlet>
```

**src\app\layouts\client-layout\client-layout.component.html**

Чесно кажучи можна було обійтись і без client-layout.component, але на початку я думав ще створити admin-layout.component, а потім плани змінились :)

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

### 3.6. Додати стилі (необов'язково)

#### 3.6.1. Встановити @csstools/normalize.css

`@csstools/normalize.css` — це сучасна версія [Normalize.css](https://necolas.github.io/normalize.css/), упакована за допомогою npm, яка стандартизує стилі браузера, забезпечуючи єдиний вигляд HTML елементів у різних браузерах.

```bash
pnpm add @csstools/normalize.css
```

#### 3.6.2. Створити файли стилів (необов'язково)

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

## 4. Додати переклад

@ngx-translate/core:

- Це основний пакет для інтернаціоналізації (i18n) в Angular.
- Він забезпечує інфраструктуру для керування перекладами у вашому застосунку.
- Він дозволяє завантажувати, зберігати та відображати переклади різними мовами.

@ngx-translate/http-loader:

- Це допоміжний пакет для @ngx-translate/core.
- Він дозволяє завантажувати файли перекладів із сервера через HTTP-запити.
- Це корисно для динамічного завантаження перекладів на основі вибраної мови.

@gilsdav/ngx-translate-router:

- Цей пакет дозволяє перекладати URL-адреси маршрутів у вашому застосунку Angular.
- Це корисно для створення багатомовних URL-адрес, що покращують SEO та зручність використання.
- Перехоплює ініціалізацію маршрутизатора та перекладає кожен шлях маршруту.

@gilsdav/ngx-translate-router-http-loader:

- Це допоміжний пакет для @gilsdav/ngx-translate-router.
- Він дозволяє завантажувати файли перекладу маршрутів із сервера через HTTP-запити.
- Використовується для завантаження перекладів для @gilsdav/ngx-translate-router.
-

### 4.1. Встановити залежності

```bash
pnpm add @ngx-translate/core @ngx-translate/http-loader @gilsdav/ngx-translate-router @gilsdav/ngx-translate-router-http-loader
```

### 4.2. Створити файли перекладів

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

### 4.3. Налаштувати локалізації

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

### 4.4. Налаштувати завантажувачі

Нам знадобляться Loaders Factory — функції, які завантажуватимуть файли перекладу та локалізації. Тож давайте одразу ж їх створимо.

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

Переконайтеся, що:

- імпортовано Location, оскільки Angular не імпортує його автоматично
- withDisabledInitialNavigation() всередині provideRouter
- додано provideHttpClient(withFetch())

### 4.5. Реалізувати переклад в компонентах

**src\app\ui\client\menu\menu.component.ts**

Імпортуйте TranslatePipe, LocalizeRouterPipe у компоненті

```ts
  imports: [RouterLink, RouterLinkActive, TranslatePipe, LocalizeRouterPipe],
```

**src\app\ui\client\menu\menu.component.html**

Додайте pipe translate — для контенту, який потрібно перекласти та localize — до routerLink, щоб додати мовний код до шляху

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

Повторіть це з іншими компонентами

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

### 4.6. Оновити перемикач мов

Реалізуємо метод setCurrentUrl.

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

### 4.7. Критична конфігурація: Вимкнення попереднього рендерингу

**Важливо:** Переконайтеся що у `angular.json` для `prerender` встановлено значення `false`. Встановлення значення `true` призведе до помилок збірки, оскільки HTTP-запити до файлів перекладу (наприклад, `en.json`) не вдасться, оскільки сервер розробки не працює під час процесу збірки.

## 5. Подяки

Я хотів би подякувати таким людям:

- [Olivier Combe](https://github.com/ocombe) та [Andreas Loew](https://github.com/CodeAndWeb) за створення ngx-translate.
- [David Gilson](https://github.com/gilsdav) за створення ngx-translate-router.
- [Robert Isaac](https://robert-isaac.medium.com/) за його статтю ["Best Practices for Angular Internationalization with SSR"](https://robert-isaac.medium.com/best-practices-for-angular-internationalization-with-ssr-384a98ee672a).

---

У наступній частині буде додано SEO, ще кілька сторінок з динамічними даними щоб побачити як вдастся локалізувати їх (сторінки) та дані які прийдуть з сервера.

[Angular + SSR + Ngx-translate + SEO](https://github.com/One-LifeMan/Angular-19-SSR-ngx-translate-SEO?tab=readme-ov-file#angular--ssr--ngx-translate--seo-)
