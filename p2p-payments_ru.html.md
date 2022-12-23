---
title: API P2P-счетов 1.0.0

search: true

metatitle: API P2P-счетов 1.0.0

category: p2p

metadescription: API P2P-счетов открывает физическим лицам доступ к операциям с выставляемыми счетами. Счет - универсальная заявка на перевод денежных средств. По умолчанию пользователю доступно несколько способов перевода. В API поддерживаются выставление и отмена счетов, а также проверка статуса выполнения перевода.

language_tabs:
  - shell
  - php: PHP SDK
  - javascript: Node.js SDK
  - java: Java SDK
  - csharp: .Net SDK
  - ppp: Popup

toc_footers:
 - <a href='/'>На главную</a>
 - <a href='https://t.me/qiwi_api_help_bot'>Обратная связь</a>
 - <a href='https://github.com/QIWI-API/p2p-payments-docs/'>Эта страница на Github</a>

---

# P2P-счета

###### Последнее обновление: 06-06-2022

## Условия использования {#terms}

Чтобы подключить на свой сайт сервис приема p2p-переводов для физических лиц, вам необходим QIWI Кошелек со статусом идентификации **Основной** или **Профессиональный**. Если у вашего кошелька статус **Анонимный**, пройдите идентификацию удобным для вас способом:

* Для получения статуса **Основной** достаточно [указать паспортные данные](https://qiwi.com/settings/identification/form).
* Для получения статуса **Профессиональный** необходимо [пройти очную идентификацию](https://qiwi.com/settings/identification/full-ru).

Рекомендуем сразу получить статус **Профессиональный**. Такой статус имеет повышенные лимиты на остаток на балансе, сумму платежей и переводов в месяц, максимальную сумму одной операции. [Подробнее про лимиты](https://qiwi.com/settings/identification/).

Рекомендуем ознакомиться с [частыми вопросами](https://qiwi.com/support/products/p2p/faq) по нашему сервису, а также с информацией о том, [как избежать блокировки кошелька](https://qiwi.com/support/products/p2p/kak_izbezhat_blokirovki_koshelka).

## Активация доступа к сервису {#p2p-activate}

1. Авторизуйтесь на [p2p.qiwi.com](https://p2p.qiwi.com)
2. Убедитесь, что вам доступно выставление счетов – в [форме **Выставить счет**](https://qiwi.com/p2p-admin/transfers/invoice) заполните поле _Сумма_ и нажмите кнопку. Ниже должна появиться ссылка на счет и кнопка **Скопировать ссылку**
![invoice-test](/images/p2p-api/invoice-create-test.png)

**Поздравляем! Вы можете приступить к интеграции.**

## Как работать с сервисом {#how-to-start}

1. Создайте публичный и секретный ключи. Подробнее см. в разделе [Методы авторизации](#auth).
2. Реализуйте взаимодействие через [API](#create) или через [вызов формы оплаты счета](#http-invoice). Вы можете воспользоваться [SDK](#sdk) или [готовыми решениями для CMS](#cms).
3. Для получения [уведомлений](#notification) после перевода по счету [активируйте их отправку](#notification-server).
4. Начните принимать платежи с банковских карт или c QIWI Кошельков.

### Сценарий платежа {#payment-scenario}

<div class="mermaid">
sequenceDiagram
participant user as Пользователь
participant rec as Получатель
participant p2p as QIWI P2P API
opt Основной сценарий
user->>rec:Оформление заказа
activate user
activate rec
alt Использование API
rec->>p2p:Выставление счета
p2p->>rec:Ссылка на платежную форму
rec->>user:Перенаправление на платежную форму<br>oplata.qiwi.com
end
alt Использование платежной формы
rec->>user:Выставление счета через форму<br>oplata.qiwi.com
end
deactivate rec
user->>p2p:Выбор способа оплаты и подтверждение платежа
p2p->>p2p:Оплата счета
opt Возврат на страницу партнера
p2p->>user:Возврат на страницу партнера
end
deactivate user
end
opt Дополнительный сценарий
opt Уведомления (callback)
activate p2p
activate rec
p2p->>rec:Уведомление об успешной оплате
rec->>p2p:Уведомление принято
deactivate rec
deactivate p2p
end
opt Проверка статуса
activate p2p
activate rec
rec->>p2p:Проверка статуса счета
p2p->>rec:Информация о счете
deactivate rec
deactivate p2p
end
end
</div>

1. Пользователь формирует счет на вашей стороне.

2. Вы перенаправляете пользователя на [платежную форму](#http-invoice) для выставления и оплаты счета в сервисе. Или [выставляете счет по API](#create) и также [перенаправляете](#payurl) пользователя на созданную платежную форму (ссылка на форму придет в ответе).

3. Пользователь выбирает способ перевода и подтверждает перевод. По умолчанию на форме отображается оптимальный для пользователя способ перевода.

4. После перевода по счету вы получите [уведомление](#notification), если [активировали отправку уведомлений](#notification-server). Уведомления о переводе по счету содержат цифровую подпись, которую необходимо [проверять на вашем сервере](#notification-auth).

Также через API вы можете:

* [запросить текущий статус перевода по счету](#invoice-status),
* [отменить неиспользуемый счет](#cancel).

## SDK и библиотеки {#sdk}

* [NODE JS SDK](https://github.com/QIWI-API/bill-payments-node-js-sdk) — Готовое решение для разработки server2server интеграции c помощью Node.js.
* [PHP SDK](https://github.com/QIWI-API/bill-payments-php-sdk) — Готовое решение для разработки server2server интеграции c помощью PHP.
* [Java SDK](https://github.com/QIWI-API/bill-payments-java-sdk) — Готовое решение для разработки server2server интеграции c помощью Java.
* [.Net SDK](https://github.com/QIWI-API/bill-payments-dotnet-sdk) — Готовое решение для разработки server2server интеграции c помощью C# .NET.

**С руководством по работе с SDK можно ознакомиться [здесь](/ru/p2p-sdk-guide/).**

## Решения для CMS {#cms}

<!--* [Wordpress](https://wordpress.org/plugins/woo-qiwi-payment-gateway/) — расширение под Woocommerce для работы с заказами.-->
* [Онлайн Лейка](https://wordpress.org/plugins/leyka/) — Wordpress расширение для благотворительности.
* [1С-Битрикс](http://marketplace.1c-bitrix.ru/solutions/qiwikassa.checkout/) — решение для работы с заказами.
* [Opencart](https://www.opencart.com/index.php?route=marketplace/extension/info&member_token=nH5fDsH3A5OkPF4zOe82hS0ypOhIqSEr&extension_id=36833) — решение для работы с заказами.
* [PrestaShop](https://github.com/QIWI-API/prestashop-payment-qiwi/releases) — решение для работы с заказами.

## Методы авторизации в сервисе {#auth}

~~~javascript
const QiwiBillPaymentsAPI = require('@qiwi/bill-payments-node-js-sdk');

const SECRET_KEY = 'eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjUyNjgxMiwiYXBpX3VzZXJfaWQiOjcxNjI2MTk3LCJzZWNyZXQiOiJmZjBiZmJiM2UxYzc0MjY3YjIyZDIzOGYzMDBkNDhlYjhiNTnONPININONPN090MTg5Z**********************';

const qiwiApi = new QiwiBillPaymentsAPI(SECRET_KEY);
~~~

~~~shell
--header "Authorization: Bearer MjMyNDQxMjM6NDUzRmRnZDQ0M*******"
~~~

~~~php
<?php

const SECRET_KEY = 'eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjUyNjgxMiwiYXBpX3VzZXJfaWQiOjcxNjI2MTk3LCJzZWNyZXQiOiJmZjBiZmJiM2UxYzc0MjY3YjIyZDIzOGYzMDBkNDhlYjhiNTnONPININONPN090MTg5Z**********************';

/** @var \Qiwi\Api\BillPayments $billPayments */
$billPayments = new Qiwi\Api\BillPayments(SECRET_KEY);

?>
~~~

~~~java
String secretKey = "eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjUyNjgxMiwiYXBpX3VzZXJfaWQiOjcxNjI2MTk3LCJzZWNyZXQiOiJmZjBiZmJiM2UxYzc0MjY3YjIyZDIzOGYzMDBkNDhlYjhiNTnONPININONPN090MTg5Z**********************";
 BillPaymentClient client = BillPaymentClientFactory.createDefault(secretKey);
~~~

~~~csharp
var secretKey = "eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjUyNjgxMiwiYXBpX3VzZXJfaWQiOjcxNjI2MTk3LCJzZWNyZXQiOiJmZjBiZmJiM2UxYzc0MjY3YjIyZDIzOGYzMDBkNDhlYjhiNTnONPININONPN090MTg5Z**********************";

var client = BillPaymentClientFactory.createDefault(secretKey);
~~~

Для авторизации запросов вам понадобятся ключи:

* Секретный ключ `<SECRET_KEY>` — для авторизации запросов к [API P2P-счетов](#create) по [стандарту OAuth 2.0](http://tools.ietf.org/html/rfc6750). Ключ указывается в заголовке HTTP-запроса `Authorization: Bearer <SECRET_KEY>`.
* Публичный ключ `<PUBLIC_KEY>` — для авторизации при [выставлении счетов через форму](#http-invoice).

Чтобы выпустить пару ключей `<SECRET_KEY>` и `<PUBLIC_KEY>`:

1. Авторизуйтесь в личном кабинете <https://p2p.qiwi.com/>.
2. Перейдите на вкладку **API** и нажмите кнопку **Создать пару ключей и настроить**. Если вы создаете пару ключей впервые, то нажмите кнопку **Настроить**.

   ![p2p API Settings](/images/p2p-api/api-settings.png)
3. Укажите название для пары ключей и нажмите кнопку **Создать**.

   <img src="/images/p2p-api/create_p2p_keys_without_notifications.png" width="384">
4. Сохраните секретный ключ в безопасном месте — в дальнейшем он не будет отображаться в интерфейсе. Публичный ключ вы всегда можете скопировать из Личного кабинета.

   <img src="/images/p2p-api/key-pair.png" width="384">
5. Нажмите кнопку **Дальше**. Пара ключей будет активирована.

<aside class="warning">
Не передавайте секретный ключ третьим лицам!

При компрометации ключей необходимо перевыпустить их.
</aside>

Вы можете использовать секретный ключ также для автоматизации платежных операций по QIWI Кошельку:

* [выполнить перевод на кошелек](/ru/qiwi-wallet-personal/#p2p);
* [выполнить перевод на карту](/ru/qiwi-wallet-personal/#cards).

Подробнее см. в [документации API QIWI Кошелька](/ru/qiwi-wallet-personal/#payments).

## Выставление счета через форму {#http-invoice}

<aside class="warning">
Этим способом доступно выставление счетов только в рублях. Для выставления счетов в тенге используйте <a href="#create">API</a>, <a href="#sdk">SDK</a> или <a href="#cms">решения для CMS</a>.
</aside>

<aside class="notice">
При открытии формы в webview на android обязательно включать <code>settings.setDomStorageEnabled(true)</code>
</aside>

Простой способ для интеграции. При переходе на форму клиенту автоматически выставляется счет. Параметры счета передаются в открытом виде в ссылке. Далее клиенту отображается форма с выбором способа перевода.

При использовании этого способа нельзя гарантировать, что все счета выставлены вами, в отличие от [выставления счета по API](#create).

<h3 class="request method">GET → </h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://oplata.qiwi.com/create</span></h3></li>
</ul>

~~~javascript
const publicKey = 'Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******';

const params = {
    publicKey,
    amount: 42.24,
    billId: 'cc961e8d-d4d6-4f02-b737-2297e51fb48e',
    successUrl: 'http://example.com/success/',
    email: 'mail@example.com'
};

const link = qiwiApi.createPaymentForm(params);
~~~

~~~shell
curl https://oplata.qiwi.com/create?publicKey=Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******&amount=100&successUrl=http%3A%2F%2Fexample.com%3Fsuccess%3F&email=mail@example.com&customFields[paySourcesFilter]=qw,card&lifetime=2020-12-01T0509
~~~

~~~php
<?php

$publicKey = '2tbp1WQvsgQeziGY9vTLe9vDZNg7tmCymb4Lh6STQokqKrpCC6qrUUKEDZAJ7mvFnzr1yTebUiQaBLDnebLMMxL8nc6FF5zf******';
$params = [
  'publicKey' => $publicKey,
  'amount' => 200,
  'billId' => 'cc961e8d-d4d6-4f02-b737-2297e51fb48e'
];

/** @var \Qiwi\Api\BillPayments $billPayments */
$link = $billPayments->createPaymentForm($params);

echo $link;

?>
~~~

~~~java
String publicKey = "2tbp1WQvsgQeziGY9vTLe9vDZNg7tmCymb4Lh6STQokqKrpCC6qrUUKEDZAJ7mvFnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypdXCbQJqHEJW5RJmKfj8nvgc";
 MoneyAmount amount = new MoneyAmount(
        BigDecimal.valueOf(499.90),
        Currency.getInstance("RUB")
);
String billId = UUID.randomUUID().toString();
String successUrl = "https://example.com/payment/success?billId=cc961e8d-d4d6-4f02-b737-2297e51fb48e";
 String paymentUrl = client.createPaymentForm(new PaymentInfo(key, amount, billId, successUrl));
~~~

~~~csharp
var publicKey = "2tbp1WQvsgQeziGY9vTLe9vDZNg7tmCymb4Lh6STQokqKrpCC6qrUUKEDZAJ7mvFnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypdXCbQJqHEJW5RJmKfj8nvgc";

var amount = new MoneyAmount
{
    ValueDecimal = 499.9m,
    CurrencyEnum = CurrencyEnum.Rub
};
var billId = Guid.NewGuid().ToString();
var successUrl = "https://example.com/payment/success?billId=cc961e8d-d4d6-4f02-b737-2297e51fb48e";

var paymentUrl = client.createPaymentForm(new PaymentInfo(key, amount, billId, successUrl));
~~~

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>В ссылке на веб-форму указываются параметры счета.</span></li>
</ul>

Параметр|Описание|Тип
---------|--------|-------
publicKey | **Обязательный параметр**. Ваш [публичный ключ для формы переводов](#auth), полученный на сайте p2p.qiwi.com|String
billId| Идентификатор выставляемого счета в вашей системе. Он должен быть уникальным и генерироваться на вашей стороне любым способом. Идентификатором может быть любая уникальная последовательность букв или цифр. Также разрешено использование символа подчеркивания (`_`) и дефиса (`-`).|String(200)
amount| Сумма, на которую выставляется счет, округленная в меньшую сторону до 2 десятичных знаков | Number(6.2)
phone | Номер телефона пользователя (в международном формате) |URL-Encoded String
email | E-mail пользователя | URL-Encoded String
account | Идентификатор пользователя в вашей системе | URL-Encoded String
comment | Комментарий к счету|URL-Encoded String(255)
customFields[]|Дополнительные данные счета|URL-Encoded String(255)
customFields[paySourcesFilter]|При открытии формы будут отображаться только указанные способы перевода, если они доступны. Возможные значения (можно указать все, через `,`): <br>`qw` - QIWI Кошелек,<br>`card` - банковская карта,<br>`mobile` - баланс телефона (телефон получателя будет виден отправителю). |URL-Encoded String
customFields[themeCode]|[Код персонализации](#custom) вашей формы |String(255)
lifetime | Дата, до которой счет будет доступен для перевода. Если перевод по счету не будет произведен до этой даты, ему присваивается финальный статус `EXPIRED` и последующий перевод станет невозможен.<br> **Внимание! По истечении 45 суток от даты выставления счет автоматически будет переведен в финальный статус** |URL-Encoded String<br>`ГГГГ-ММ-ДДTччмм`
successUrl|URL для переадресации на ваш сайт в случае успешного перевода|URL-Encoded String

## API P2P-счетов. Выставление счета {#create}

<aside class="warning">
По оплаченным счетам возврат денежных средств не предусмотрен.
</aside>

Доступно выставление счетов в рублях и тенге.

Надежный способ для интеграции. Параметры передаются server2server с использованием авторизации.

При успешном выполнении запроса в ответе вернется параметр `payUrl` — ссылка для перенаправления пользователя на форму оплаты. К ней вы можете добавить дополнительные параметры. Подробнее см. в разделе [Форма для оплаты счета](#payurl).

<aside class="success">
Рекомендуем воспользоваться <a href="/ru/p2p-sdk-guide/">SDK</a> для интеграции.
</aside>

Также существует более простой способ выставления счета — [непосредственно через вызов платежной формы](#http-invoice).

<aside class="notice">
Для тестирования и отладки сервиса рекомендуем выставлять и оплачивать счета суммой 1 рубль.
</aside>

**Ознакомьтесь со статьями [Кошелек заблокирован. Что делать?](https://qiwi.com/support/security/subject21/razblokirovat-koshelek) и [Как избежать блокировки кошелька](https://qiwi.com/support/products/p2p/kak_izbezhat_blokirovki_koshelka).**

<h3 class="request method">Запрос → PUT</h3>

~~~javascript
const billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

const fields = {
    amount: 1.00,
    currency: 'RUB',
    comment: 'Hello world',
    expirationDateTime: '2018-03-02T08:44:07+03:00'
};

qiwiApi.createBill( billId, fields ).then( data => {
    //do with data
});
~~~

~~~shell
curl --location --request PUT 'https://api.qiwi.com/partner/bill/v1/bills/cc961e8d-d4d6-4f02-b737-2297e51fb48e' \
--header 'content-type: application/json' \
--header 'accept: application/json' \
--header 'Authorization: Bearer eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjIwNDIsImFwaV91c2VyX2lkIjo1NjYwMzk3Miwic2VjcmV0IjoiQjIwODlDNkI5Q0NDNTdCNDQzNGHJK43JFJDK595FJFJMjlCRkFFRDM5OE***********************' \
--data-raw '{  
   "amount": {   
     "currency": "RUB",   
     "value": "1.00" 
   },  
   "comment": "Text comment",  
   "expirationDateTime": "2025-12-10T09:02:00+03:00",  
   "customer": {
     "phone": "78710009999",
     "email": "test@example.com",
     "account": "454678"
   }, 
   "customFields" : {
     "paySourcesFilter":"qw",
     "themeCode": "Yvan-YKaSh",
     "yourParam1": "64728940",
     "yourParam2": "order 678"
   }
 }'
~~~

~~~php
<?php

$billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';
$fields = [
  'amount' => 1.00,
  'currency' => 'RUB',
  'comment' => 'test',
  'expirationDateTime' => '2018-03-02T08:44:07+03:00',
  'email' => 'mail@example.org',
  'account' => 'client4563'
];

/** @var \Qiwi\Api\BillPayments $billPayments */
$response = $billPayments->createBill($billId, $fields);

print_r($response);

?>
~~~

~~~java
CreateBillInfo billInfo = new CreateBillInfo(
                UUID.randomUUID().toString(),
                new MoneyAmount(
                        BigDecimal.valueOf(199.90),
                        Currency.getInstance("RUB")
                ),
                "comment",
                ZonedDateTime.now().plusDays(45),
                new Customer(
                        "mail@example.org",
                        UUID.randomUUID().toString(),
                        "79123456789"
                ),
                "http://example.com/success"
        );
BillResponse response = client.createBill(billInfo);
~~~

~~~csharp
client.CreateBill(
    info: new CreateBillInfo
    {
        BillId = Guid.NewGuid().ToString(),
        Amount = new MoneyAmount
        {
            ValueDecimal = 199.9m,
            CurrencyEnum = CurrencyEnum.Rub
        },
        Comment = "comment",
        ExpirationDateTime = DateTime.Now.AddDays(45),
        Customer = new Customer
        {
            Email = "mail@example.org",
            Account = Guid.NewGuid().ToString(),
            Phone = "79123456789"
        },
        SuccessUrl = new Uri("http://example.com/success"),
        CustomFields: new CustomFields
        {
            ThemeCode = "кодСтиля"
        }
    }
);
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/partner/bill/v1/bills/{billid}</span></h3>
        <ul>
             <li><strong>billId</strong> - сгенерированный на вашей стороне любым способом идентификатор счета. Идентификатором может быть любая уникальная последовательность букв или цифр. Также разрешено использование символа подчеркивания (_) и дефиса (-).</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer <a href="#auth">SECRET_KEY</a></li>
             <li>Content-Type: application/json</li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

Параметр тела запроса|Описание|Тип
---------|--------|------------
amount|**Обязательный параметр**. Информация о сумме счета|Object
amount.<br>value| **Обязательный параметр**. Сумма, на которую выставляется счет, округленная в меньшую сторону до 2 десятичных знаков | Number(6.2)
amount.<br>currency| **Обязательный параметр**. Валюта суммы счета. Возможные значения:<br>`RUB` - рубли,<br>`KZT` - тенге.| Alpha-3 ISO 4217 код
expirationDateTime | **Обязательный параметр**. Дата, до которой счет будет доступен для оплаты. Если перевод не будет совершен до этой даты, ему присваивается финальный статус `EXPIRED` и последующий перевод станет невозможен. | `ГГГГ-ММ-ДДTчч:мм:сс±чч:мм`
customer|Идентификаторы пользователя|Object
customer.<br>phone | Номер телефона пользователя (в международном формате) |String
customer.<br>email | E-mail пользователя |String
customer.<br>account | Идентификатор пользователя в вашей системе|String
comment | Комментарий к счету|String(255)
customFields | Дополнительная информация о счете. Вы можете здесь передавать свои дополнительные поля с данными, например, `SteamId`|Object
customFields.<br>paySourcesFilter | Строка с разделителями-запятыми. При [открытии формы](#payurl) будут отображаться только указанные способы перевода (один или несколько), если они доступны. Возможные значения:<br>`qw` - QIWI Кошелек,<br>`card` - банковская карта,<br>`mobile` - баланс телефона (телефон получателя будет виден отправителю). |String
customFields.<br>themeCode | [Код персонализации](#custom) вашей формы |String(255)

<h3 class="request">Ответ ←</h3>

>Пример тела ответа

~~~json
{
    "siteId": "9hh4jb-00",
    "billId": "cc961e8d-d4d6-4f02-b737-2297e51fb48e",
    "amount": {
        "currency": "RUB",
        "value": "1.00"
    },
    "status": {
        "value": "WAITING",
        "changedDateTime": "2021-01-18T14:22:56.672+03:00"
    },
    "customer": {
        "phone": "78710009999",
        "email": "test@example.com",
        "account": "454678"
    },
    "customFields": {
        "paySourcesFilter": "qw",
        "themeCode": "Yvan-YKaSh",
        "yourParam1": "64728940",
        "yourParam2": "order 678"
    },
    "comment": "Text comment",
    "creationDateTime": "2021-01-18T14:22:56.672+03:00",
    "expirationDateTime": "2025-12-10T09:02:00+03:00",
    "payUrl": "https://oplata.qiwi.com/form/?invoice_uid=aa0fa2bb-5452-47ca-9190-cd9c1a73718f"
}
~~~

>Пример тела ответа при ошибке

~~~json
{
    "serviceName": "invoicing-api",
    "errorCode": "http.message.conversion.failed",
    "description": "Bad request",
    "userMessage": "Bad request",
    "dateTime": "2021-01-18T14:29:51.984+03:00",
    "traceId": "8fa9cfe10c7f83d1"
}

~~~

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Content-Type: application/json</li>
        </ul>
    </li>
</ul>

Поле|Тип|Описание
--------|---|--------
siteId|String|Ваш идентификатор в сервисе приема платежей для физических лиц p2p.qiwi.com
billId|String|Уникальный идентификатор счета в вашей системе, указанный при выставлении
amount|Object|Информация о сумме счета
amount.<br>value|Number|Сумма счета, округленная до 2 знаков после запятой в меньшую сторону
amount.<br>currency	|String|Валюта суммы счета (Alpha-3 ISO 4217 код)
status|Object|Информация о статусе счета
status.<br>value|String|Текущий [статус счета](#status)
status.<br>changedDateTime|String|Дата обновления статуса. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс±чч:мм`
customFields|Object|Объект строковых дополнительных параметров. Возможные элементы: `paySourcesFilter`, `themeCode`
customer|Object|Идентификаторы пользователя. Возможные элементы: `email`, `phone`, `account`
comment|String|Комментарий к счету
creationDateTime|String| Системная дата создания счета. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс±чч:мм`
expirationDateTime|String|Срок действия созданной формы для перевода. Формат даты: `ГГГГ-ММ-ДДTчч:мм:сс±чч:мм`
payUrl|String|[Ссылка](#payurl) на созданную форму. Перенаправьте пользователя по этой ссылке для оплаты счета или используйте [библиотеку Popup](#popup), чтобы открыть форму во всплывающем окне.

## API P2P-счетов. Проверка статуса перевода по счету {#invoice-status}

Рекомендуется использовать этот метод после получения [уведомления о переводе](#notification).

<h3 class="request method">Запрос → GET</h3>

~~~javascript
const billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

qiwiApi.getBillInfo(billId).then( data => {
    //do smth with data
});
~~~

~~~shell
curl --location --request GET 'https://api.qiwi.com/partner/bill/v1/bills/cc961e8d-d4d6-4f02-b737-2297e51fb48e' \
--header 'accept: application/json' \
--header 'Authorization: Bearer eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjIwNDIsImFwaV91c2VyX2lkIjo1NjYwMzk3Miwic2VjcmV0IjoiQjIwODlDNkI5Q0NDNTdCNDQzNGHJK43JFJDK595FJFJMjlCRkFFRDM5OE***********************'
~~~

~~~php
<?php

$billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

/** @var \Qiwi\Api\BillPayments $billPayments */
$response = $billPayments->getBillInfo($billId);

print_r($response);

?>
~~~

~~~java
String billId = "cc961e8d-d4d6-4f02-b737-2297e51fb48e";
 BillResponse response = client.getBillInfo(billId);
~~~

~~~csharp
var billId = "cc961e8d-d4d6-4f02-b737-2297e51fb48e";

var response = client.getBillInfo(billId);
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/partner/bill/v1/bills/{billid}</span></h3>
        <ul>
             <li><strong>billId</strong> — уникальный идентификатор счета в вашей системе.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer <a href="#auth">SECRET_KEY</a></li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

>Пример тела ответа

~~~json
{
    "siteId": "9hh4jb-00",
    "billId": "cc961e8d-d4d6-4f02-b737-2297e51fb48e",
    "amount": {
        "currency": "RUB",
        "value": "1.00"
    },
    "status": {
        "value": "WAITING",
        "changedDateTime": "2021-01-18T14:22:56.672+03:00"
    },
    "customer": {
        "email": "test@example.com",
        "phone": "78710009999",
        "account": "454678"
    },
    "customFields": {
        "paySourcesFilter": "qw",
        "themeCode": "Yvan-YKaSh",
        "yourParam1": "64728940",
        "yourParam2": "order 678"
    },
    "comment": "Text comment",
    "creationDateTime": "2021-01-18T14:22:56.672+03:00",
    "expirationDateTime": "2025-12-10T09:02:00+03:00",
    "payUrl": "https://oplata.qiwi.com/form/?invoice_uid=aa0fa2bb-5452-47ca-9190-cd9c1a73718f"
}
~~~

>Пример тела ответа при ошибке

~~~json
{
    "serviceName": "invoicing-api",
    "errorCode": "api.invoice.not.found",
    "description": "Invoice not found",
    "userMessage": "Invoice not found",
    "dateTime": "2021-01-18T14:34:40.865+03:00",
    "traceId": "b3d41cafa0c6d088"
}
~~~

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Content-Type: application/json</li>
        </ul>
    </li>
</ul>

Поле|Тип|Описание
--------|---|--------
billId|String|Уникальный идентификатор счета в вашей системе, указанный при выставлении
siteId|String|Ваш идентификатор в сервисе приема платежей для физических лиц p2p.qiwi.com
amount|Object|Информация о сумме счета
amount.<br>value|Number|Сумма счета, округленная до 2 знаков после запятой в меньшую сторону.
amount.<br>currency|String|Идентификатор валюты суммы счета (Alpha-3 ISO 4217 код)
status|Object|Информация о статусе счета
status.<br>value  |String|Текущий [статус счета](#status)
status.<br>changedDateTime|String|Дата обновления статуса
customFields|Object|Объект строковых дополнительных параметров, переданных вами
customer|Object|Идентификаторы пользователя
customer.<br>phone  |String |Номер телефона (если был указан при выставлении счета)
customer.<br>email |String|E-mail пользователя (если был указан при выставлении счета)
customer.<br>account |String| Идентификатор пользователя в вашей системе (если был указан при выставлении счета)
comment|String|Комментарий к счету
creationDateTime|String| Системная дата создания счета. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс`
payUrl|String|[Ссылка для переадресации](#payurl) пользователя на созданную форму
expirationDateTime|String|Срок действия созданной формы для перевода. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс±чч:мм`

## API P2P-счетов. Отмена неоплаченного счета {#cancel}

Вы можете отменить счет, по которому не был выполнен перевод.

<h3 class="request method">Запрос → POST</h3>

~~~javascript
const bill_id = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

qiwiApi.cancelBill(billId).then( data => {
    //do with data
});
~~~

~~~shell
curl --location --request POST 'https://api.qiwi.com/partner/bill/v1/bills/cc961e8d-d4d6-4f02-b737-2297e51fb48e/reject' \
--header 'content-type: application/json' \
--header 'accept: application/json' \
--header 'Authorization: Bearer eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjIwNDIsImFwaV91c2VyX2lkIjo1NjYwMzk3Miwic2VjcmV0IjoiQjIwODlDNkI5Q0NDNTdCNDQzNGHJK43JFJDK595FJFJMjlCRkFFRDM5OE***********************' \
--data-raw ''
~~~

~~~php
<?php

$billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

/** @var \Qiwi\Api\BillPayments $billPayments */
$response = $billPayments->cancelBill($billId);

print_r($response);

?>
~~~

~~~java
String billId = "cc961e8d-d4d6-4f02-b737-2297e51fb48e";
 BillResponse response = client.cancelBill(billId);
~~~

~~~csharp
var billId = "cc961e8d-d4d6-4f02-b737-2297e51fb48e";

var response = client.cancelBill(billId);
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/partner/bill/v1/bills/{billId}/reject</span></h3>
        <ul>
        <strong>Параметры:</strong>
             <li><strong>billId</strong> — уникальный идентификатор счета в вашей системе.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer <a href="#auth">SECRET_KEY</a></li>
             <li>Content-Type: application/json</li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

>Пример тела ответа

~~~json
{
    "siteId": "9hh4jb-00",
    "billId": "cc961e8d-d4d6-4f02-b737-2297e51fb48e",
    "amount": {
        "currency": "RUB",
        "value": "1.00"
    },
    "status": {
        "value": "REJECTED",
        "changedDateTime": "2021-01-18T14:36:17.65+03:00"
    },
    "customer": {
        "email": "test@example.com",
        "phone": "78710009999",
        "account": "454678"
    },
    "customFields": {
        "paySourcesFilter": "qw",
        "themeCode": "Yvan-YKaSh",
        "yourParam1": "64728940",
        "yourParam2": "order 678"
    },
    "comment": "Text comment",
    "creationDateTime": "2021-01-18T14:22:56.672+03:00",
    "expirationDateTime": "2025-12-10T09:02:00+03:00",
    "payUrl": "https://oplata.qiwi.com/form/?invoice_uid=aa0fa2bb-5452-47ca-9190-cd9c1a73718f"
}
~~~

>Пример тела ответа при ошибке

~~~json
{
    "serviceName": "invoicing-api",
    "errorCode": "api.invoice.not.found",
    "description": "Invoice not found",
    "userMessage": "Invoice not found",
    "dateTime": "2021-01-18T14:39:54.265+03:00",
    "traceId": "bc6bb6e7c5cf5beb"
}
~~~

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Content-Type: application/json</li>
        </ul>
    </li>
</ul>

Поле|Тип|Описание
--------|---|--------
billId|String|Уникальный идентификатор счета в вашей системе, указанный при выставлении
siteId|String|Ваш идентификатор в сервисе приема платежей для физических лиц p2p.qiwi.com
amount|Object|Информация о сумме счета
amount.<br>value|Number|Сумма счета, округленная до 2 знаков после запятой в меньшую сторону
amount.<br>currency	|String|Идентификатор валюты суммы счета (Alpha-3 ISO 4217 код)
status|Object|Информация о статусе счета
status.<br>value  |String|Текущий [статус счета](#status)
status.<br>changedDateTime|String|Дата обновления статуса. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс±чч:мм`
customFields|Object|Объект строковых дополнительных параметров. Возможные элементы: `paySourcesFilter`, `themeCode`
customer|Object|Идентификаторы пользователя. Возможные элементы: `email`, `phone`, `account`
comment|String|Комментарий к счету
creationDateTime|String| Системная дата создания счета. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс`
payUrl|String|Ссылка на созданную платежную форму
expirationDateTime|String|Срок действия созданной формы для перевода. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс±чч:мм`

## API P2P-счетов. Статусы оплаты счетов {#status}

Статус|Описание|Финальный
------|--------|---------
WAITING | Счет выставлен, ожидает оплаты| -
PAID|Счет оплачен|+
REJECTED|Счет отклонен|+
EXPIRED|Время жизни счета истекло. Счет не оплачен|+

## Уведомления о переводе по счету {#notification}

<aside class="notice">
Сервис не является обязательным для интеграции, вы можете реализовать более простой вариант с <a href="#invoice-status">опросом статуса счета</a>.
</aside>

Перед началом работы с сервисом уведомлений прочитайте [условия по интеграции API уведомлений](https://qiwi.com/support/products/p2p/usloviya_integratsii_api_uvedomleniy).

Пулы IP-адресов, с которых сервисы QIWI отправляют уведомления:

* 79.142.16.0/20
* 195.189.100.0/22
* 91.232.230.0/23
* 91.213.51.0/24

Если ваш сервер обработки уведомлений работает за брандмауэром, необходимо добавить эти IP-адреса в список разрешенных адресов входящих TCP-пакетов.

<aside class="warning">
Уведомление о переводе (callback) отправляется только по протоколу HTTPS и только на 443 порт.

Сертификат должен быть выдан доверенным центром сертификации (например, Comodo, Verisign, Thawte и т.п.).
</aside>

Уведомление представляет собой входящий HTTP POST-запрос.

<h3 class="request method">Запрос ← POST</h3>

 >Пример уведомления

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: application/json
Content-type: application/json
X-Api-Signature-SHA256: J4WNfNZd***V5mv2w=
Host: example.com

{
  "bill": {
    "siteId": "9hh4jb-00",
    "billId": "cc961e8d-d4d6-4f02-b737-2297e51fb48e",
    "amount": {
      "value": "1.00",
      "currency": "RUB"
    },
    "status": {
      "value": "PAID",
      "changedDateTime": "2021-01-18T15:25:18+03"
    },
    "customer": {
      "phone": "78710009999",
      "email": "test@example.com",
      "account": "454678"
    },
    "customFields": {
      "paySourcesFilter": "qw",
      "themeCode": "Yvan-YKaSh",
      "yourParam1": "64728940",
      "yourParam2": "order 678"
    },
    "comment": "Text comment",
    "creationDateTime": "2021-01-18T15:24:53+03",
    "expirationDateTime": "2025-12-10T09:02:00+03"
  },
  "version": "1"
}
~~~

Тело запроса содержит JSON-сериализованные данные счета (кодировка UTF-8).

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>X-Api-Signature-SHA256: XXX</li>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
        </ul>
    </li>
</ul>

<aside class="notice">
Заголовки по стандарту регистронезависимы и ваш клиент может их изменить — смотрите документацию вашего клиента.
</aside>

<aside class="notice">
Если в ответе код состояния HTTP отличается от 200 (OK), это интерпретируется как временная ошибка в вашей системе. Сервер QIWI будет повторять запрос с нарастающим интервалом в течение суток.
</aside>

<aside class="warning">
Уведомление может быть отправлено несколько раз даже в случае успешного ответа от вашего сервиса. Учитывайте это при разработке бизнес-логики на вашей стороне.
</aside>

## Регистрация сервера уведомлений {#notification-server}

Адрес сервера для уведомлений настраивается в [Личном кабинете P2P](https://p2p.qiwi.com/). При этом выпускается новая [пара ключей для авторизации](#auth).

<aside class="warning">
Вы можете выполнить настройку сервера уведомлений только для одной пары ключей.
</aside>

1. Авторизуйтесь в [Личном кабинете P2P](https://p2p.qiwi.com/).
2. Перейдите на вкладку **API** и нажмите кнопку **Создать пару ключей и настроить**. Если вы настраиваете адрес сервера вместе с первичным созданием пары ключей впервые, то нажмите кнопку **Настроить**.

   ![p2p API Settings](/images/p2p-api/api-settings.png)
3. Укажите название для новой пары ключей.

   <img src="/images/p2p-api/create_p2p_keys_without_notifications.png" width="384">
4. Выделите поле **Использовать эту пару ключей для серверных уведомлений об изменении статусов счетов**.

   <img src="/images/p2p-api/create_p2p_keys_with_notifications.png" width="384">
5. В поле **URL сервера для уведомлений** укажите адрес вашего сервера для обработки уведомлений об оплате. **Внимание! Сервер должен быть доступен из интернета.**
6. Нажмите кнопку **Создать**.

   <img src="/images/p2p-api/key-pair.png" width="384">
7. Замените в настройках ваших приложений ключи для сервиса QIWI P2P на вновь созданные.

## Проверка подлинности уведомлений {#notification-auth}

После получения входящего уведомления необходимо проверить его подлинность. Для этого используется механизм цифровой подписи. Подпись уведомления отправляется в HTTP–заголовке `X-Api-Signature-SHA256`. Для формирования подписи используется механизм проверки целостности HMAC с хеш-функцией SHA256.

Алгоритм проверки подписи:

1. Объединить значения следующих параметров уведомления в одну строку с разделителем `|`:

   `invoice_parameters = {amount.currency}|{amount.value}|{billId}|{siteId}|{status.value}`

   где `{*}` – значение параметра. Все значения при проверке подписи должны трактоваться как строки.

2. Вычислить HMAC-хеш c алгоритмом хеширования SHA256:

   `hash = HMAС(SHA256, invoice_parameters, <SECRET_KEY>)`

   Где:

   * `<SECRET_KEY>` – [секретный ключ](#auth), при помощи которого был выставлен счёт;
   * `invoice_parameters` – строка из п.1.

3. Сравнить значение заголовка `X-Api-Signature-SHA256` с результатом из п.2.

~~~javascript
const validSignatureFromNotificationServer =
      '07e0ebb10916d97760c196034105d010607a6c6b7d72bfa1c3451448ac484a3b';

const notificationData = {
    bill: {
        siteId: 'test',
        billId: 'test_bill',
        amount: { value: 1, currency: 'RUB' },
        status: { value: 'PAID', datetime: '2018-03-01T11:16:12+03' },
        customer: {},
        customFields: {},
        creationDateTime: '2018-03-01T11:15:39+03:',
        expirationDateTime: '2018-04-15T11:15:39+03'
    },
    version: '1'
};

const secretKey = 'eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjUyNjgxMiwiYXBpX3VzZXJfaWQiOjcxNjI2MTk3LCJzZWNyZXQiOiJmZjBiZmJiM2UxYzc0MjY3YjIyZDIzOGYzMDBkNDhlYjhiNTnONPININONPN090MTg5Z**********************';

qiwiApi.checkNotificationSignature(
    validSignatureFromNotificationServer, notificationData, secretKey
); // true
~~~

~~~php
<?php

$validSignatureFromNotificationServer = '07e0ebb10916d97760c196034105d010607a6c6b7d72bfa1c3451448ac484a3b';
$notificationData = [
  'bill' => [
    'siteId' => 'test',
    'billId' => 'test_bill',
    'amount' => ['value' => 1, 'currency' => 'RUB'],
    'status' => ['value' => 'PAID']
  ],
  'version' => '3'
];
$secretKey = 'eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjUyNjgxMiwiYXBpX3VzZXJfaWQiOjcxNjI2MTk3LCJzZWNyZXQiOiJmZjBiZmJiM2UxYzc0MjY3YjIyZDIzOGYzMDBkNDhlYjhiNTnONPININONPN090MTg5Z**********************';

/** @var \Qiwi\Api\BillPayments $billPayments */
$billPayments->checkNotificationSignature(
  $validSignatureFromNotificationServer, $notificationData, $secretKey
); // true

?>
~~~

~~~java
String secretKey = "eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjUyNjgxMiwiYXBpX3VzZXJfaWQiOjcxNjI2MTk3LCJzZWNyZXQiOiJmZjBiZmJiM2UxYzc0MjY3YjIyZDIzOGYzMDBkNDhlYjhiNTnONPININONPN090MTg5Z**********************";
Notification notification = new Notification(
        new Bill(
                "test",
                "test_bill",
                new MoneyAmount(
                        BigDecimal.ONE,
                        Currency.getInstance("RUB")
                ),
                BillStatus.PAID
        ),
        "3"
);
String validSignature = "07e0ebb10916d97760c196034105d010607a6c6b7d72bfa1c3451448ac484a3b";
 BillPaymentsUtils.checkNotificationSignature(validSignature, notification, secretKey); //true
~~~

Строка и ключ подписи кодируются в UTF-8.

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>В уведомлении содержится информация о счете.</span>
    </li>
</ul>

Поле|Описание|Тип
---------|--------|---
bill|Информация о счете|Object
billId|Уникальный идентификатор счета в вашей системе, указанный при выставлении|String(200)
siteId|Ваш идентификатор в сервисе приема платежей для физических лиц p2p.qiwi.com| String
amount|Информация о сумме счета|Object
amount.<br>value | Сумма счета, округленная до двух десятичных знаков в меньшую сторону | Number(6.2)
amount.<br>currency | Идентификатор валюты суммы счета (Alpha-3 ISO 4217 код) | String(3)
status | Информация о статусе счета | Object
status.<br>value |[Строковое значение статуса](#status) | String
status.<br>changedDateTime|Дата обновления статуса. Формат даты<br>`ГГГГ-ММ-ДДTЧЧ:ММ:СС+Z`| String
customer | Информация о пользователе| Object
customer.<br>phone |Номер телефона (если был указан при выставлении счета) |String
customer.<br>email|E-mail пользователя (если был указан при выставлении счета)|String
customer.<br>account| Идентификатор пользователя в вашей системе (если был указан при выставлении счета)|String
creationDateTime | Дата создания счета. Формат даты<br>`ГГГГ-ММ-ДДTЧЧ:ММ:СС+Z`| String
expirationDateTime | Срок оплаты счета. Формат даты<br>`ГГГГ-ММ-ДДTЧЧ:ММ:СС+Z` |String
comment | Комментарий к счету | String(255)
customFields | Дополнительные данные счета (если были указаны при выставлении счета)| Object
version | Версия уведомлений | String

<h3 class="request">Ответ → </h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
 "error":"0"
}
~~~

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Content-type: application/json</li>
        </ul>
    </li>
</ul>

После того, как был получен входящий запрос-уведомление, необходимо [проверить подлинность цифровой подписи](#notification-auth) и отправить ответ.

## Форма для оплаты счета {#payurl}

<aside class="notice">
При открытии формы в webview на android обязательно включать <code>settings.setDomStorageEnabled(true)</code>
</aside>

При [выставлении счета через API](#create) в ответе присутствует поле `payUrl` со ссылкой на платежную форму. При перенаправлении по ссылке для оплаты счета к ней можно добавить параметры:

> Пример ссылки

~~~shell
https://oplata.qiwi.com/form?invoiceUid=a8437e7e-dc48-44f7-9bdb-4d46ca8ef2e4&paySource=qw&successUrl=google.com
~~~

|Параметр | Описание | Тип|
|--------|----------------|---------|
| paySource | При открытии формы сразу будет выбран указанный способ перевода. Возможные значения:<br>`qw` - QIWI Кошелек,<br>`card` - банковская карта,<br>`mobile` - баланс телефона.<br> Если способ перевода недоступен, выбирается рекомендуемый для пользователя способ| String |
| successUrl | URL для переадресации на ваш сайт в случае успешного перевода | URL-Encoded String |

Добавьте реферальные ссылки для платежей с сайта. Полная ссылка подтвердит реальность сайта и позволит [избежать проблем с блокировкой кошелька](https://qiwi.com/support/products/p2p/kak_izbezhat_blokirovki_koshelka). **Платежи, проходящие со страницы без заголовка запроса [Refer](https://developer.mozilla.org/ru/docs/Web/HTTP/Headers/Referer) будут приводить к блокировке кошелька. Подробнее см. в статье [Как передавать реферальные ссылки](https://qiwi.com/support/products/p2p/referalnie_ssylki).**

> Пример передачи реферальной ссылки

~~~php
<?php 
  header("Referrer-Policy: no-referrer-when-downgrade"); 
?>
~~~

Для того, чтобы в новых версиях браузеров передавался [полный referer](https://web.dev/referrer-best-practices/) при переходе, укажите в ответе сервера заголовок `Referrer-Policy` со значением `no-referrer-when-downgrade`. Это можно сделать для всего сайта или только для страницы со ссылкой на форму оплаты.

## Персонализация {#custom}

Вы можете настроить персонализированную форму оплаты: изменить свое имя на название магазина и настроить цвет фона и кнопок.

Пример персонализированной формы оплаты:

![Customer form](/images/Custom.png)

Чтобы настроить внешний вид формы оплаты:

1. Авторизуйтесь в [Личном кабинете P2P](https://p2p.qiwi.com/).
2. Перейдите в раздел [Форма приема переводов](https://qiwi.com/p2p-admin/transfers/link). Код вашего стиля для переменной `themeCode` (указывается в запросах API и вызовах SDK, см. далее) выделен жирным шрифтом в блоке серого цвета. Значение `themeCode` индивидуально для разных QIWI кошельков.

   ![alt_text](/images/p2p-api/sdk-image1.png "Form settings")
3. Нажмите кнопку **Настроить** и выполните настройку формы.
4. Нажмите кнопку **Сохранить**.

 >Пример передачи параметра при вызове платежной формы

~~~shell
curl https://oplata.qiwi.com/create?publicKey=Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******&amount=100&billId=cc961e8d-d4d6-4f02-b737-2297e51fb48e&successUrl=http%3A%2F%2Fexample.com%3Fsuccess%3F&customFields%5BthemeCode%5D=кодСтиля
~~~

 >Пример передачи параметра в запросе к API

~~~shell
curl --location --request PUT 'https://api.qiwi.com/partner/bill/v1/bills/cc961e8d-d4d6-4f02-b737-2297e51fb48e' \
--header 'content-type: application/json' \
--header 'accept: application/json' \
--header 'Authorization: Bearer eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjIwNDIsImFwaV91c2VyX2lkIjo1NjYwMzk3Miwic2VjcmV0IjoiQjIwODlDNkI5Q0NDNTdCNDQzNGHJK43JFJDK595FJFJMjlCRkFFRDM5OE***********************' \
--data-raw '{
   "amount": {
     "currency": "RUB",
     "value": 100.00
   },
   "comment": "Text comment",
   "expirationDateTime": "2025-04-13T14:30:00+03:00",
   "customer": {},
   "customFields": {"themeCode":"кодСтиля"}
 }'
~~~

~~~javascript
const billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

const fields = {
    amount: 1.00,
    currency: 'RUB',
    comment: 'Hello world',
    customFields: {themeCode: 'кодСтиля'},
    expirationDateTime: '2018-03-02T08:44:07+03:00'
};

qiwiApi.createBill( billId, fields ).then( data => {
    //do with data
});
~~~

~~~php
<?php

$billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';
$customFields = ['themeCode' => 'кодСтиля'];
$fields = [
  'amount' => 1.00,
  'currency' => 'RUB',
  'comment' => 'test',
  'expirationDateTime' => '2018-03-02T08:44:07+03:00',
  'email' => 'mail@example.org',
  'account' => 'client4563',
  'customFields' => $customFields
];

/** @var \Qiwi\Api\BillPayments $billPayments */
$response = $billPayments->createBill($billId, $fields);

print_r($response);

?>
~~~

~~~csharp
client.CreateBill(
    info: new CreateBillInfo
    {
        BillId = Guid.NewGuid().ToString(),
        Amount = new MoneyAmount
        {
            ValueDecimal = 199.9m,
            CurrencyEnum = CurrencyEnum.Rub
        },
        Comment = "comment",
        ExpirationDateTime = DateTime.Now.AddDays(45),
        Customer = new Customer
        {
            Email = "mail@example.org",
            Account = Guid.NewGuid().ToString(),
            Phone = "79123456789"
        },
        SuccessUrl = new Uri("http://example.com/success"),
        CustomFields: new CustomFields
        {
            ThemeCode = "кодСтиля"
        }
    }
);
~~~

Для применения стиля к платежной форме:

* В [запросе создания счета](#create) или при вызове метода создания счета в [нужном модуле SDK](#sdk) добавьте в тело запроса объект `customFields` c полем `themeCode` и укажите в нем код вашего стиля.

   Например, `"customFields": { "themeCode":"кодСтиля"}`.
* При [вызове платежной формы](#http-invoice) добавьте в запрос параметр `customFields` c полем `themeCode` и укажите в нем код вашего стиля.

   Например, `customFields[themeCode]=кодСтиля`.

## Библиотека Checkout Popup {#popup}

<button id="pop" class="button-popup" onclick="testPopup();">Пример работы popup</button>

Методы библиотеки позволяют открыть форму перевода как всплывающее окно (popup) поверх вашего сайта.

[Скачать библиотеку QIWI Checkout Popup](https://github.com/QIWI-API/qiwi-invoicing-popup)

Установка и подключение библиотеки:

`<script src='https://oplata.qiwi.com/popup/v1.js'></script>`

В библиотеке доступно два метода:

* [Открытие существующего счета](#openpopup).
* [Открытие](#openpopupcustom) вашей [формы приема переводов](https://qiwi.com/p2p-admin/transfers/link).

### Открытие существующего счета {#openpopup}

Метод  `QiwiCheckout.openInvoice`.

>Пример открытия уже созданного счета в popup

~~~ppp
params = {
    payUrl: 'https://oplata.qiwi.com/form?invoiceUid=06df838c-0f86-4be3-aced-a950c244b5b1'
}

QiwiCheckout.openInvoice(params)
    .then(data => {
        // ...
    })
    .catch(error => {
        // ...
    })
~~~

|Параметр | Описание | Тип |
|---------|----------|----------|
| payUrl | **Обязательный параметр**. URL счета | String |

### Открытие персонализированной формы {#openpopupcustom}

Метод  `QiwiCheckout.openPreorder`.

>Пример открытия персонализированной формы

~~~ppp
params = {
    widgetAlias: 'https://my.qiwi.com/form/User-ABCDE1234-56'
}

QiwiCheckout.openPreorder(params)
    .then(data => {
        // ...
    })
    .catch(error => {
        // ...
    })
~~~

|Параметр | Описание | Тип |
|---------|----------|----------|
| widgetAlias | **Обязательный параметр**. Ссылка на форму приема переводов | String |
