# CallLinks API

CallLinks — сервис Click-to-Call для передачи телефонных номеров из браузера или внешней системы на смартфон пользователя. Сервис не совершает звонок из браузера и не является VoIP-провайдером: он отправляет номер на Android или iPhone, открывает стандартный номеронабиратель смартфона и подставляет номер. Пользователь подтверждает вызов на телефоне.

Официальный сайт: https://calllinks.sochka.com/

Политика конфиденциальности: https://calllinks.sochka.com/privacy/

## Зачем нужен этот репозиторий

Этот репозиторий подготовлен как русскоязычная документация по CallLinks с акцентом на API и интеграции. Он поможет подключить передачу номера на телефон из CRM, интернет-магазина, внутренней панели, сайта, таблицы, WordPress, SaaS-сервиса или любого другого инструмента, который умеет выполнять HTTP-запросы или открывать URL.

## Быстрый старт

1. Установите мобильное приложение CallLinks на смартфон.
2. Откройте приложение и получите `TOKEN` устройства.
3. Сохраните `TOKEN` в настройках вашей системы или интеграции.
4. Передайте номер телефона через API:

```text
https://calllinks.sochka.com/push/send.php?number={phone}&deviceToken={token}
```

Пример:

```text
https://calllinks.sochka.com/push/send.php?number=%2B380501234567&deviceToken=YOUR_DEVICE_TOKEN
```

После запроса номер будет отправлен на смартфон, связанный с указанным `TOKEN`.

## Основные API-методы

### Отправить номер на смартфон

```text
GET https://calllinks.sochka.com/push/send.php?number={phone}&deviceToken={token}
```

Параметры:

- `number` — номер телефона в любом текстовом формате. Сервис нормализует номер перед передачей на смартфон.
- `deviceToken` — секретный `TOKEN`, сгенерированный мобильным приложением CallLinks.

Ответ:

- Число оставшихся кредитов.

### Проверить баланс кредитов

```text
GET https://calllinks.sochka.com/push/balans.php?deviceToken={token}
```

Ответ:

- Число оставшихся кредитов.

### Открыть страницу пополнения кредитов

```text
GET https://calllinks.sochka.com/push/pay.php?deviceToken={token}
```

Метод открывает страницу покупки или пополнения кредитов для указанного `TOKEN`.

## Пример интеграции в сайт

```html
<button id="call-client" data-phone="+380501234567">
  Позвонить клиенту
</button>

<script>
  const token = "YOUR_DEVICE_TOKEN";

  document.getElementById("call-client").addEventListener("click", async (event) => {
    const phone = event.currentTarget.dataset.phone;
    const url = new URL("https://calllinks.sochka.com/push/send.php");

    url.searchParams.set("number", phone);
    url.searchParams.set("deviceToken", token);

    const response = await fetch(url.toString());
    const creditsLeft = await response.text();

    console.log(`Осталось кредитов: ${creditsLeft}`);
  });
</script>
```

## Где использовать API

- CRM и панели менеджеров.
- Админки интернет-магазинов.
- WordPress-сайты.
- Внутренние корпоративные панели.
- Google Sheets и похожие таблицы через скрипты.
- Helpdesk и support-системы.
- SaaS-продукты с карточками клиентов.
- Call center dashboards.
- Любые системы, где рядом с клиентом хранится номер телефона.

## Документация

- [Подробная документация API](docs/API.md)
- [Примеры интеграций](docs/INTEGRATIONS.md)
- [Файл для LLM-ассистентов](llms.txt)

## Приложения и расширения

Android:

https://play.google.com/store/apps/details?id=com.sochka.calllink&hl=ru

iPhone:

https://apps.apple.com/us/app/calllinks-click-to-call/id6760187978

Google Chrome:

https://chromewebstore.google.com/detail/calllinks-click-to-call/cmgnijjcpabhmjielfmmldngfmogbdgj?hl=ru

Mozilla Firefox:

https://addons.mozilla.org/ru/firefox/addon/calllinks-click-to-call/

Microsoft Edge:

https://microsoftedge.microsoft.com/addons/detail/calllinks-clicktocall/hnbajdgpalcmgoadklojgaphdepldimh?hl=ru

## Безопасность

- Не публикуйте реальный `TOKEN` в публичном коде, GitHub-репозиториях, frontend-бандлах и скриншотах.
- Для серверных интеграций храните `TOKEN` в переменных окружения или в защищенных настройках проекта.
- Если интеграция работает только на стороне браузера, используйте отдельный `TOKEN` и учитывайте риск его раскрытия пользователям.
- Не передавайте `TOKEN` третьим лицам.

## Поддерживаемые языки сайта

- Украинский: https://calllinks.sochka.com/uk
- Русский: https://calllinks.sochka.com/ru
- Английский: https://calllinks.sochka.com/en
- Немецкий: https://calllinks.sochka.com/de
- Французский: https://calllinks.sochka.com/fr
- Испанский: https://calllinks.sochka.com/es

## Короткое описание

CallLinks — Click-to-Call API и набор приложений для передачи телефонного номера из браузера, CRM, сайта или внутренней системы на смартфон пользователя.
