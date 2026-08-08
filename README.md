# Akfa Nukus — окна, двери и витражи

Одностраничный сайт-визитка. Один файл `index.html` — без сборки, без зависимостей, без CDN.
Достаточно открыть файл в браузере или залить папку на любой хостинг.

## Состав

| Файл | Что это |
|---|---|
| `index.html` | вся страница: разметка, CSS, переводы и логика формы |
| `1.jpg` … `6.jpg` | фотографии работ |

## Что уже есть

- Адаптивная вёрстка (проверено на 375 px и десктопе).
- Переключатель языков **RU / UZ** — выбор сохраняется в `localStorage`,
  при первом визите берётся из языка браузера.
- Форма заказа: маска телефона `+998 90 123 45 67`, проверка полей,
  honeypot против ботов, блокировка кнопки на время отправки.
- Контакты: [+998 91 371 14 20](tel:+998913711420), Telegram [@emirlann1](https://t.me/emirlann1).

## Настройка формы заказа

В конце `index.html` есть две константы:

```js
const ENDPOINT = 'ВСТАВЬТЕ_АДРЕС_ОБРАБОТЧИКА';
const CHAT_ID  = '7713592368';
```

**Не вставляйте в `ENDPOINT` адрес Telegram с токеном бота.** Исходный код страницы
видит любой посетитель — токен уведут, и бот будет рассылать спам от вашего имени.

Вместо этого поднимите обработчик, который хранит токен у себя. Бесплатный вариант —
Cloudflare Worker (5 минут, карта не нужна):

1. [dash.cloudflare.com](https://dash.cloudflare.com) → **Workers & Pages** → **Create Worker**.
2. Вставьте код:

```js
export default {
  async fetch(req, env) {
    const cors = { 'Access-Control-Allow-Origin': '*', 'Access-Control-Allow-Headers': 'Content-Type' };
    if (req.method === 'OPTIONS') return new Response(null, { headers: cors });
    if (req.method !== 'POST')    return new Response('nope', { status: 405, headers: cors });

    const { text } = await req.json();
    if (!text || text.length > 2000) return Response.json({ ok: false }, { status: 400, headers: cors });

    const r = await fetch(`https://api.telegram.org/bot${env.BOT_TOKEN}/sendMessage`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ chat_id: env.CHAT_ID, text, disable_web_page_preview: true })
    });
    return Response.json(await r.json(), { headers: cors });
  }
};
```

3. **Settings → Variables** → добавьте секреты `BOT_TOKEN` и `CHAT_ID`.
4. Скопируйте адрес воркера (`https://…workers.dev`) в `ENDPOINT`.

## Публикация через GitHub Pages

**Settings → Pages** → Source: `Deploy from a branch` → ветка `main`, папка `/ (root)`.
Через минуту сайт будет доступен по адресу `https://azamat2014.github.io/akfa-nukus/`.

## Правки

- **Цвета** — переменные в начале `<style>`: `--brand`, `--accent`, `--bg`.
- **Тексты и переводы** — объект `I18N` в `<script>`, ключи `ru` и `uz`.
  Третий язык добавляется копией блока `uz:` и кнопкой `<button data-lang="…">` в шапке.
- **Фото** — замените файлы `1.jpg`…`6.jpg`, имена оставьте прежними.
