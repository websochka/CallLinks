# Примеры интеграций CallLinks API

В этом файле собраны практические примеры подключения CallLinks API к сайтам, CRM, WordPress, backend-сервисам и таблицам.

Во всех примерах используйте свои значения:

```text
YOUR_DEVICE_TOKEN
+380501234567
```

## HTML-ссылка

Самый простой вариант — обычная ссылка:

```html
<a href="https://calllinks.sochka.com/push/send.php?number=%2B380501234567&deviceToken=YOUR_DEVICE_TOKEN">
  Позвонить клиенту
</a>
```

Минус такого подхода: `TOKEN` виден в HTML.

## JavaScript в браузере

```html
<button class="calllinks-call" data-phone="+380501234567">
  Позвонить
</button>

<script>
  const CALLLINKS_TOKEN = "YOUR_DEVICE_TOKEN";

  async function sendToPhone(phone) {
    const url = new URL("https://calllinks.sochka.com/push/send.php");
    url.searchParams.set("number", phone);
    url.searchParams.set("deviceToken", CALLLINKS_TOKEN);

    const response = await fetch(url.toString());

    if (!response.ok) {
      throw new Error("Не удалось отправить номер");
    }

    return response.text();
  }

  document.querySelectorAll(".calllinks-call").forEach((button) => {
    button.addEventListener("click", async () => {
      button.disabled = true;

      try {
        const creditsLeft = await sendToPhone(button.dataset.phone);
        button.textContent = `Отправлено, осталось кредитов: ${creditsLeft}`;
      } catch (error) {
        button.textContent = "Ошибка отправки";
      } finally {
        setTimeout(() => {
          button.disabled = false;
          button.textContent = "Позвонить";
        }, 3000);
      }
    });
  });
</script>
```

## Node.js backend

```js
const express = require("express");

const app = express();
const token = process.env.CALLLINKS_DEVICE_TOKEN;

app.use(express.json());

app.post("/api/call", async (req, res) => {
  const { phone } = req.body;

  if (!phone) {
    return res.status(400).json({ error: "phone is required" });
  }

  const url = new URL("https://calllinks.sochka.com/push/send.php");
  url.searchParams.set("number", phone);
  url.searchParams.set("deviceToken", token);

  const response = await fetch(url.toString());
  const creditsLeft = await response.text();

  if (!response.ok) {
    return res.status(502).json({ error: "CallLinks request failed" });
  }

  res.json({ ok: true, creditsLeft });
});

app.listen(3000);
```

## PHP

```php
<?php

$token = getenv('CALLLINKS_DEVICE_TOKEN');
$phone = $_POST['phone'] ?? '';

if ($phone === '') {
    http_response_code(400);
    echo 'phone is required';
    exit;
}

$query = http_build_query([
    'number' => $phone,
    'deviceToken' => $token,
]);

$url = 'https://calllinks.sochka.com/push/send.php?' . $query;
$creditsLeft = file_get_contents($url);

echo $creditsLeft;
```

## Python

```python
import os
import requests

token = os.environ["CALLLINKS_DEVICE_TOKEN"]
phone = "+380501234567"

response = requests.get(
    "https://calllinks.sochka.com/push/send.php",
    params={
        "number": phone,
        "deviceToken": token,
    },
    timeout=10,
)

response.raise_for_status()

credits_left = response.text
print(f"Номер отправлен. Осталось кредитов: {credits_left}")
```

## WordPress shortcode

Пример шорткода `[calllinks phone="+380501234567"]Позвонить[/calllinks]`.

```php
<?php

add_shortcode('calllinks', function ($atts, $content = 'Позвонить') {
    $atts = shortcode_atts([
        'phone' => '',
    ], $atts);

    $token = defined('CALLLINKS_DEVICE_TOKEN') ? CALLLINKS_DEVICE_TOKEN : '';

    if ($atts['phone'] === '' || $token === '') {
        return '';
    }

    $url = add_query_arg([
        'number' => $atts['phone'],
        'deviceToken' => $token,
    ], 'https://calllinks.sochka.com/push/send.php');

    return sprintf(
        '<a class="calllinks-button" href="%s">%s</a>',
        esc_url($url),
        esc_html($content)
    );
});
```

В `wp-config.php` можно хранить токен так:

```php
define('CALLLINKS_DEVICE_TOKEN', 'YOUR_DEVICE_TOKEN');
```

## Google Sheets Apps Script

Пример отправки номера из активной ячейки:

```js
function sendSelectedPhoneToCallLinks() {
  const token = PropertiesService
    .getScriptProperties()
    .getProperty("CALLLINKS_DEVICE_TOKEN");

  const sheet = SpreadsheetApp.getActiveSheet();
  const phone = sheet.getActiveCell().getValue();

  const params = {
    number: phone,
    deviceToken: token,
  };

  const query = Object.keys(params)
    .map((key) => `${encodeURIComponent(key)}=${encodeURIComponent(params[key])}`)
    .join("&");

  const response = UrlFetchApp.fetch(
    `https://calllinks.sochka.com/push/send.php?${query}`
  );

  SpreadsheetApp.getUi().alert(
    `Номер отправлен. Осталось кредитов: ${response.getContentText()}`
  );
}
```

## CRM или внутренняя панель

Рекомендуемая схема:

```text
CRM frontend -> ваш backend -> CallLinks API -> смартфон менеджера
```

Почему так лучше:

- токен хранится на сервере;
- можно логировать факт нажатия без раскрытия токена;
- можно проверять права пользователя;
- можно выбирать токен конкретного менеджера;
- можно централизованно обрабатывать ошибки.

Пример бизнес-логики:

1. Менеджер нажимает кнопку `Позвонить`.
2. Frontend отправляет `customerId` на backend.
3. Backend получает телефон клиента из базы.
4. Backend выбирает `deviceToken` менеджера.
5. Backend вызывает `send.php`.
6. Frontend показывает результат.

## Проверка баланса перед звонком

```js
async function getCallLinksBalance(token) {
  const url = new URL("https://calllinks.sochka.com/push/balans.php");
  url.searchParams.set("deviceToken", token);

  const response = await fetch(url.toString());
  return response.text();
}
```

Если баланс низкий, можно показать ссылку на пополнение:

```text
https://calllinks.sochka.com/push/pay.php?deviceToken=YOUR_DEVICE_TOKEN
```

## Чеклист для интеграции

- Получить `TOKEN` в мобильном приложении CallLinks.
- Решить, где хранить `TOKEN`: backend, переменная окружения, защищенные настройки CRM.
- Выбрать сценарий: frontend-кнопка, backend endpoint, WordPress shortcode, Apps Script или webhook.
- Всегда URL-кодировать номер телефона.
- Обработать успешный ответ и показать остаток кредитов.
- Добавить обработку ошибок HTTP-запроса.
- Не публиковать реальный `TOKEN` в GitHub.
