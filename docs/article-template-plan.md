# План: Шаблон страницы статьи (MVP)

## Контекст

Блог о картах тахографа на Astro 6.1.1 имеет минимальный шаблон: Header → Hero → Дата → H1 → Текст → Footer (720px, одна колонка). Нет CTA, оглавления, хлебных крошек, FAQ-аккордеона, Schema.org-разметки.

**Компания:** АвтоОка (ПК «АВТООКА»)
**Фирменный цвет:** `#E8772E` (оранжевый, с лого skzicard.ru). Текущий `--accent: #2337ff` → заменить
**CTA-цель:** https://skzicard.ru/
**Хостинг:** сначала локально (`npm run preview`), деплой на GitHub Pages позже
**Шрифт:** Atkinson (оставить текущий)

**Дополнительно обнаружены проблемы:**
- `lang="en"` вместо `lang="ru"` в BlogPost
- `consts.ts`: `SITE_TITLE = 'AI Digest'`, описание про AI/ML
- Header/Footer: ссылки на Mastodon/Twitter/GitHub Astro (неактуальны)
- `FormattedDate`: locale `en-us`
- `sample-digest-post.md` — статья про Claude 4.6 Opus → удалить
- `astro.config.mjs`: `site: 'https://example.com'` → TODO перед деплоем
- `about.astro`: контент про «AI Digest» → обновить минимально
- `--accent: #2337ff` (синий) → `#E8772E` (оранжевый, фирменный)

---

## 1. Исправление существующих проблем

### 1.1 `src/consts.ts`
```ts
export const SITE_TITLE = 'АвтоОка — Карты тахографа';
export const SITE_DESCRIPTION = 'Экспертный блог о картах тахографа: оформление, замена, продление, законодательство и ответы на вопросы водителей и перевозчиков.';

export const CATEGORIES: Record<string, string> = {
  'poluchit-kartu': 'Получить карту',
  'zamenit-kartu': 'Заменить карту',
  'reshit-problemu': 'Решить проблему',
  'zakon-i-proverki': 'Закон и проверки',
  'spravochnik': 'Справочник',
};
```

### 1.2 `src/components/Header.astro`
- Ссылки: Главная (`/`), Блог (`/blog`), О компании (`/about`) — на русском
- Убрать ссылки на Mastodon/Twitter/GitHub Astro
- Соцсети убрать полностью

### 1.3 `src/components/Footer.astro`
- Копирайт: `© {year} АвтоОка. Все права защищены.`
- Убрать ссылки на Astro-соцсети
- Добавить: `8 (804) 333-90-55` (телефон с главной skzicard.ru)

### 1.4 `src/components/FormattedDate.astro`
- `locale: 'en-us'` → `locale: 'ru-RU'`

### 1.5 `src/styles/global.css`
- `--accent: #2337ff` → `--accent: #E8772E`
- `--accent-dark: #000d8a` → `--accent-dark: #C45F1A` (затемнённый оранжевый)

### 1.6 `src/pages/about.astro`
Обновить контент минимально, взяв за основу https://skzicard.ru/kompaniya-avtooko/:
- Название: АвтоОка
- С 2012 года на рынке
- Выдано более 400 000 карт водителей
- 500+ партнёров по всей России
- Услуги: карты СКЗИ, ЕСТР, предприятия
- Лицензия ФСБ Л051-00105-21/00283526/Н
- Телефон: 8 (804) 333-90-55
- Убрать упоминания AI Digest, Claude Code, «курс»

### 1.7 `astro.config.mjs`
- `site: 'https://example.com'` → оставить с TODO-комментарием: `// TODO: заменить на реальный домен перед деплоем`

### 1.8 Удалить `src/content/blog/sample-digest-post.md`

---

## 2. Новые компоненты

### 2.1 `src/components/Breadcrumbs.astro`
- Props: `items: Array<{ label: string; href?: string }>`
- `<nav aria-label="Breadcrumb"><ol>` с разделителем `›`
- Последний элемент — `<strong>` без ссылки
- Встроенный `<script type="application/ld+json">` с BreadcrumbList

### 2.2 `src/components/ArticleHeader.astro`
- Props: `title, pubDate, updatedDate?, readingTime, heroImage?, category?`
- Бейдж категории (один стиль для всех, фон `var(--accent)`) — если `category` передан и есть в `CATEGORIES`
- H1
- Строка мета: дата (FormattedDate) + время чтения + «Обновлено: ...» (если updatedDate)
- HeroImage через `<Image>` из `astro:assets` (если передано)

### 2.3 `src/components/TableOfContents.astro`
- Props: `headings: Array<{ depth: number; slug: string; text: string }>`
- **Фильтровать:** только H2 и H3 (`depth >= 2 && depth <= 3`), исключить H1
- Рендерит `<nav aria-label="Содержание"><ul>...</ul></nav>`
- Вложенный `<ul>` для H3 внутри родительского H2
- Рендерится **один раз** внутри `<details open>` обёртки в layout

### 2.4 `src/components/FaqAccordion.astro`
- Props: `items: Array<{ question: string; answer: string }>`
- **Условный рендер:** `items` пустой/undefined → не рендерить ничего
- `<details>/<summary>` для каждого вопроса (нативный HTML, без JS)
- Встроенный `<script type="application/ld+json">` с FAQPage (только если items > 0)

### 2.5 `src/components/CtaBlock.astro`
- Props: `text?, buttonText?, buttonUrl?` (с дефолтами)
- Дефолты: `text: 'Закажите карту тахографа'`, `buttonText: 'Оформить карту на skzicard.ru'`, `buttonUrl: 'https://skzicard.ru/'`
- **Рендерится на каждой статье** (дефолтные значения всегда применяются)
- Кнопка: `target="_blank" rel="noopener noreferrer"`

### 2.6 `src/components/RelatedArticles.astro`
- Props: `currentSlug, currentTags: string[], allPosts`
- Алгоритм: исключить текущую → count пересечений тегов → sort desc → топ-3
- 0 совпадений → не рендерить блок
- Карточки: название, дата (FormattedDate), описание (обрезка 150 символов)

### 2.7 `src/components/VideoBlock.astro`
- Props: `url?: string, title?: string`
- **Условный рендер:** `url` не передан → не рендерить ничего
- Только YouTube: `watch?v=ID`, `youtu.be/ID` → `youtube.com/embed/ID`
- Адаптивный контейнер 16:9, `loading="lazy"` на iframe
- Плейсхолдер URL для существующей статьи: `'https://www.youtube.com/watch?v=PLACEHOLDER'` (заменить позже)

### 2.8 JSON-LD Article — инлайн в BlogPost.astro
- В `<head>`: `<script type="application/ld+json">` с Article schema
- Поля: name, description, datePublished, dateModified, url
- author: `@type: Organization`, name: `АвтоОка`

---

## 3. Утилита

### `src/utils/readingTime.ts`
```ts
export function computeReadingTime(text: string): string {
  if (!text) return '1 мин чтения';
  const words = text.split(/\s+/).filter(Boolean).length;
  const minutes = Math.max(1, Math.ceil(words / 200));
  return `${minutes} мин чтения`;
}
```
- Вызывается в `[...slug].astro` с `post.body ?? ''`
- Fallback: пустой текст → `'1 мин чтения'`

---

## 4. Изменение схемы контента

**Файл:** `src/content.config.ts`

Все новые поля — optional/default, **не ломают существующие статьи**:

```ts
// Добавить к существующим полям:
category: z.string().default('spravochnik'),
faq: z.array(z.object({
  question: z.string(),
  answer: z.string(),
})).optional(),
video: z.string().url().optional(),
cta: z.object({
  text: z.string().optional(),
  buttonText: z.string().optional(),
  buttonUrl: z.string().optional(),
}).optional(),
```

---

## 5. Реструктуризация BlogPost.astro

**Файл:** `src/layouts/BlogPost.astro`

### Изменения:
1. `lang="en"` → `lang="ru"`
2. Props: `{ post, headings, readingTime, allPosts }`
3. **Удалить** старый `.hero-image` блок (перенесён в ArticleHeader)
4. **Удалить** `.title`, `.date`, `.last-updated-on` стили (перенесены в ArticleHeader)
5. Двухколоночный grid + все новые компоненты

### Структура:
```
<html lang="ru">
  <head>
    <BaseHead />
    <script type="application/ld+json">   ← Article JSON-LD
  </head>
  <body>
    <Header />
    <main>
      <Breadcrumbs items={[
        { label: 'Главная', href: '/' },
        { label: 'Блог', href: '/blog' },
        { label: post.data.title }
      ]} />
      <div class="article-layout">
        <div class="article-body">
          <ArticleHeader ... />
          <div class="prose">
            <slot />
          </div>
          <VideoBlock url={post.data.video} />
          <FaqAccordion items={post.data.faq} />
          <CtaBlock ... />
        </div>
        <aside class="toc-sidebar">
          <details open class="toc-details">
            <summary class="toc-summary">Содержание</summary>
            <TableOfContents headings={headings} />
          </details>
        </aside>
      </div>
      <RelatedArticles ... />
    </main>
    <Footer />
  </body>
</html>
```

### TOC: один рендер, адаптивность через CSS

TOC рендерится **один раз** внутри `<details open>`:
- **Десктоп (>960px):** `<details>` всегда раскрыт, summary стилизован как заголовок, треугольник скрыт
- **Мобильный (<=960px):** `<details>` работает как аккордеон, начинается раскрытым (`open`), пользователь может свернуть

---

## 6. Обновление `[...slug].astro`

**Файл:** `src/pages/blog/[...slug].astro`

```astro
---
import { getCollection, render } from 'astro:content';
import type { CollectionEntry } from 'astro:content';
import BlogPost from '../../layouts/BlogPost.astro';
import { computeReadingTime } from '../../utils/readingTime';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post, allPosts: posts },
  }));
}

const { post, allPosts } = Astro.props;
const { Content, headings } = await render(post);
const readingTime = computeReadingTime(post.body ?? '');
---

<BlogPost post={post} headings={headings} readingTime={readingTime} allPosts={allPosts}>
  <Content />
</BlogPost>
```

---

## 7. CSS

**Файл:** `src/styles/global.css`

### Изменить в `:root`:
```css
--accent: #E8772E;           /* было #2337ff */
--accent-dark: #C45F1A;      /* было #000d8a */
--sidebar-width: 260px;       /* новое */
--content-max-width: 1100px;  /* новое */
```

### Новые стили:
```css
/* Article layout — двухколоночный grid */
.article-layout {
  display: grid;
  grid-template-columns: 1fr var(--sidebar-width);
  gap: 2rem;
  max-width: var(--content-max-width);
  margin: 0 auto;
  padding: 0 1em;
}

.toc-sidebar {
  position: sticky;
  top: 2rem;
  align-self: start;
  max-height: calc(100vh - 4rem);
  overflow-y: auto;
}

/* TOC details — десктоп всегда раскрыт, summary как заголовок */
.toc-details[open] > .toc-summary { list-style: none; }
.toc-details > .toc-summary::-webkit-details-marker { display: none; }
.toc-details > .toc-summary { font-weight: 700; font-size: 1.1em; cursor: default; }

/* Video responsive wrapper */
.video-wrapper {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 9;
}
.video-wrapper iframe {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  border: none;
  border-radius: 8px;
}

/* Mobile: одна колонка, TOC-аккордеон */
@media (max-width: 960px) {
  .article-layout {
    display: flex;
    flex-direction: column;
  }
  .toc-sidebar {
    position: static;
    order: -1;
    border-bottom: 1px solid rgb(var(--gray-light));
    padding-bottom: 1rem;
    margin-bottom: 1.5rem;
  }
  .toc-details > .toc-summary {
    cursor: pointer;
    /* показать треугольник-маркер на мобильном */
  }
}
```

### Не трогать:
- `main { width: 720px }` в global.css — BlogPost уже переопределяет через scoped `main { max-width: 100% }`
- Остальные страницы (blog listing, about) сохраняют 720px

---

## 8. Порядок реализации

### Фаза A: Фундамент (4 шага)
1. Обновить `src/consts.ts`
2. Обновить `src/content.config.ts`
3. Создать `src/utils/readingTime.ts`
4. Обновить `src/components/FormattedDate.astro`

### Фаза B: Существующие компоненты + очистка (5 шагов)
5. Обновить `src/components/Header.astro`
6. Обновить `src/components/Footer.astro`
7. Удалить `src/content/blog/sample-digest-post.md`
8. Обновить `src/pages/about.astro` (контент о компании)
9. Обновить `src/styles/global.css` (акцентный цвет, новые переменные)

### Фаза C: Новые компоненты (7 шагов, параллельно)
10. `src/components/Breadcrumbs.astro`
11. `src/components/ArticleHeader.astro`
12. `src/components/TableOfContents.astro`
13. `src/components/FaqAccordion.astro`
14. `src/components/VideoBlock.astro`
15. `src/components/CtaBlock.astro`
16. `src/components/RelatedArticles.astro`

### Фаза D: Сборка (3 шага)
17. Добавить CSS для article-layout в `src/styles/global.css`
18. Реструктурировать `src/layouts/BlogPost.astro`
19. Обновить `src/pages/blog/[...slug].astro`

### Фаза E: Контент (1 шаг)
20. Обновить frontmatter `kak-poluchit-kartu-takhografa.md` (добавить `category`)

### Фаза F: Проверка
21. `npm run build` → `npm run preview` → визуальная проверка

---

## 9. Проверка

1. `npm run build` — без ошибок
2. `npm run preview` — визуально:
   - Десктоп: двухколоночный grid, TOC sticky справа, раскрыт
   - Мобильный: одна колонка, TOC-аккордеон перед контентом, можно свернуть
   - Хлебные крошки: Главная › Блог › Название
   - Бейдж категории у H1
   - Дата на русском + время чтения
   - Оранжевый акцент вместо синего
   - CTA-кнопка → skzicard.ru
   - Footer: «© 2026 АвтоОка»
3. View source: JSON-LD (BreadcrumbList + Article)
4. FAQ — если нет faq в frontmatter, блок не рендерится
5. Video — если нет video, блок не рендерится
6. Похожие статьи — 0 совпадений → блок скрыт
7. `sample-digest-post.md` удалена
8. About page — контент про АвтоОку
9. Google Rich Results Test — валидация Schema.org

---

## 10. Что НЕ входит в MVP

- Деплой на GitHub Pages (после проверки локально)
- Поддержка Rutube / VK Видео
- Документ `article-writing-requirements.md`
- Блок автора
- Страницы категорий `/blog/[category]/`
- Обновление главной страницы `index.astro`
- 404-страница
- Аналитика/отслеживание CTA-кликов
- Категории и время чтения в карточках Blog listing
