---
title: API P2P-счетов 1.0.0

search: true

metatitle: API P2P-счетов 1.0.0



metadescription: API P2P-счетов открывает доступ к операциям с выставляемыми счетами. Счет - универсальная заявка на перевод. По умолчанию пользователю доступно несколько способов перевода. В API поддерживаются операции выставления и отмены счетов, а также проверки статуса выполнения операций.

language_tabs:
  - shell
  - php: PHP SDK
  - javascript: Node.js SDK
  - java: Java SDK
  - csharp: .Net SDK
  - ppp: Popup

services:
 - <a href='#'>Swagger</a>  |  <a href='#'>Qiwi Demo</a>


toc_footers:
 - <a href='/'>На главную</a>
 - <a href='https://tele.click/qiwi_api_help_bot'>Обратная связь</a>

---

# Основные сведения {#intro}

###### Последнее обновление: 2020-06-04 | [Редактировать на GitHub](https://github.com/QIWI-API/p2p-payments-docs/blob/master/p2p-payments_ru.html.md)

API P2P-счетов открывает доступ к операциям с выставляемыми счетами. Счет - универсальная заявка на перевод. По умолчанию пользователю доступно несколько способов перевода.
В API поддерживаются операции выставления и отмены счетов, а также проверки статуса выполнения операций.

**Для работы API потребуются публичный и секретный ключи. Ключи создаются в личном кабинете в разделе "API" (доступен после авторизации на сайте [p2p.qiwi.com](https://p2p.qiwi.com)).**

## Схема работы {#steps}

<!--![Operation Flow](/images/bill_payments.png)-->

<div class="mermaid">
sequenceDiagram
participant user as Пользователь
participant rec as Получатель
participant p2p as QIWI P2P API
opt Основной сценарий
user->>rec:Оформление заказа
activate user
activate rec
opt Использование API
rec->>p2p:Выставление счета
p2p->>rec:Ссылка на платежную форму
end
rec->>user:Перенаправление на платежную форму<br>Выставление счета через форму<br>oplata.qiwi.com
deactivate rec
user->>p2p:Открытие платежной формы<br>Выбор способа оплаты и подтверждение платежа
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


* Пользователь формирует счет на вашей стороне.

* Вы перенаправляете пользователя на [платежную форму](#http) для выставления счета. Или [выставляете счет по API](#create) и перенаправляете пользователя на созданную платежную форму.

* Пользователь выбирает способ перевода и подтверждает перевод. По умолчанию подбирается оптимальный для пользователя способ перевода.

* После перевода по счету вы получаете [уведомление](#notification) (предварительно настройте отправку уведомлений в [Личном кабинете](https://p2p.qiwi.com) при создании ключей). Уведомления о переводе по счету содержат параметры авторизации, которые необходимо проверять на Вашем сервере.

* Также есть возможность
  * [запросить текущий статус перевода по счету](#invoice-status),
  * [отменить неиспользуемый счет](#cancel).

* Для автоматизации операций по QIWI кошельку можете воспользоваться [API QIWI Кошелька](https://developer.qiwi.com/ru/qiwi-wallet-personal):
  * [выполнить перевод на кошелек](https://developer.qiwi.com/ru/qiwi-wallet-personal/#p2p),
  * [выполнить перевод на карту](https://developer.qiwi.com/ru/qiwi-wallet-personal/#cards),
  * другие возможности - подробнее в [документации](https://developer.qiwi.com/ru/qiwi-wallet-personal/#payments).


# Готовые решения

## SDK и библиотеки {#sdk}

* [NODE JS SDK](https://github.com/QIWI-API/bill-payments-node-js-sdk) - Готовое решение для разработки server2server интеграции c помощью Node.js.
* [PHP SDK](https://github.com/QIWI-API/bill-payments-php-sdk) - Готовое решение для разработки server2server интеграции c помощью PHP.
* [Java SDK](https://github.com/QIWI-API/bill-payments-java-sdk) - Готовое решение для разработки server2server интеграции c помощью Java.
* [.Net SDK](https://github.com/QIWI-API/bill-payments-dotnet-sdk) - Готовое решение для разработки server2server интеграции c помощью C# .Net.

## Решения для CMS {#cms}

* [Wordpress](https://wordpress.org/plugins/woo-qiwi-payment-gateway/) -  расширение под Woocommerce для работы с заказами
* [Онлайн Лейка](https://wordpress.org/plugins/leyka/) -  Wordpress расширение для благотворительности
* [1С-Битрикс](http://marketplace.1c-bitrix.ru/solutions/qiwikassa.checkout/) - решение для работы с заказами
* [Opencart](https://www.opencart.com/index.php?route=marketplace/extension/info&member_token=nH5fDsH3A5OkPF4zOe82hS0ypOhIqSEr&extension_id=36833) - решение для работы с заказами
* [PrestaShop](https://github.com/QIWI-API/prestashop-payment-qiwi/releases) - решение для работы с заказами

# Авторизация {#auth}

~~~javascript
const QiwiBillPaymentsAPI = require('bill-payments-node-js-sdk');

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

Ваши запросы авторизуются посредством секретного ключа API (`SECRET_KEY`). Параметр авторизации указывается в заголовке <i>Authorization</i>, значение которого формируется как "Bearer SECRET_KEY".

Публичный ключ (`PUBLIC_KEY`) используется для выставления счетов [через форму](#http).

**Ключи создаются в личном кабинете на вкладке API после авторизации на [p2p.qiwi.com](https://p2p.qiwi.com).**

<aside class="warning">
Важно! Не передавайте секретный ключ третьим лицам!.
</aside>


# Выставление счета через форму {#http}

<aside class="notice">
При открытии формы в webview на android обязательно включать <code>settings.setDomStorageEnabled(true)</code>
</aside>

Простой способ для интеграции. При открытии формы клиенту автоматически выставляется счет. Параметры счета передаются в открытом виде в ссылке. Далее клиенту отображается форма с выбором способа перевода.
При использовании этого способа нельзя гарантировать, что все счета выставлены вами, в отличие от [выставления по API](#create).


<h3 class="request method">REDIRECT → </h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://oplata.qiwi.com/create</span></h3></li>
</ul>

~~~javascript
const publicKey = 'Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******';

const params = {
    publicKey,
    amount: 42.24,
    billId: 'cc961e8d-d4d6-4f02-b737-2297e51fb48e',
    successUrl: 'http://test.ru/',
    email: 'm@ya.ru'
};

const link = qiwiApi.createPaymentForm(params);
~~~

~~~shell
curl https://oplata.qiwi.com/create?publicKey=Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******&amount=100&billId=cc961e8d-d4d6-4f02-b737-2297e51fb48e&successUrl=http%3A%2F%2Ftest.ru%3F&email=m@ya.ru
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
String successUrl = "https://merchant.com/payment/success?billId=cc961e8d-d4d6-4f02-b737-2297e51fb48e";
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
var successUrl = "https://merchant.com/payment/success?billId=cc961e8d-d4d6-4f02-b737-2297e51fb48e";

var paymentUrl = client.createPaymentForm(new PaymentInfo(key, amount, billId, successUrl));
~~~

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>В ссылке на веб-форму указываются параметры счета.</span></li>
</ul>

Параметр|Описание|Тип|Обяз.
---------|--------|---|---------
publicKey | Ваш ключ идентификации полученный в p2p.qiwi.com|String|+
billId|Уникальный идентификатор выставляемого счета в вашей системе|URL-закодированная строка String(200)|-
amount| Сумма, на которую выставляется счет, округленная в меньшую сторону до 2 десятичных знаков | Number(6.2)|-
phone | Номер телефона пользователя (в международном формате) | URL-закодированная строка|-
email | E-mail пользователя | URL-закодированная строка|-
account | Идентификатор пользователя в вашей системе | URL-закодированная строка |-
comment | Комментарий к счету|URL-закодированная строка String(255)|-
customFields[]|Дополнительные данные счета|URL-закодированная строка String(255)|-
lifetime | Дата, до которой счет будет доступен для перевода. Если перевод по счету не будет произведен до этой даты, ему присваивается финальный статус `EXPIRED` и последующий перевод станет невозможен.<br> **Внимание! По истечении 45 суток от даты выставления счет автоматически будет переведен в финальный статус** |URL-закодированная строка<br>`ГГГГ-ММ-ДДTччмм`|-
successUrl|URL для переадресации в случае успешного перевода с баланса QIWI Кошелька. При ином способе оплаты переадресация не выполняется. Ссылка должна вести на ваш сайт.|URL-закодированная строка|-

# Взаимодействие через API {#API}

## 1. Выставление счета {#create}

<aside class="warning">
Доступно выставление счетов только в рублях. По оплаченным счетам возврат денежных средств не предусмотрен.
</aside>

Надежный способ для интеграции. Параметры передаются server2server с использованием авторизации.
Метод позволяет выставить счет: при успешном выполнении запроса в ответе вернется параметр `payUrl` - ссылка для редиректа пользователя на форму.

**[Настройки формы и счета](#option)**



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
   "comment": "",  
   "expirationDateTime": "2020-07-10T09:02:00+03:00",  
   "customer": {},  
   "customFields" : {}   
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
  'email' => 'example@mail.org',
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
                        "example@mail.org",
                        UUID.randomUUID().toString(),
                        "79123456789"
                ),
                "http://merchant.ru/success"
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
            Email = "example@mail.org",
            Account = Guid.NewGuid().ToString(),
            Phone = "79123456789"
        },
        SuccessUrl = new Uri("http://merchant.ru/success")
    },
    customFields: new CustomFields
    {
        ThemeCode = "кодСтиля"
    }
);
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/partner/bill/v1/bills/{billid}</span></h3>
        <ul>
             <li><strong>billId</strong> - уникальный идентификатор счета в вашей системе.</li>
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

Параметр|Описание|Тип|Обяз.
---------|--------|---|---------
billId|Идентификатор счета в вашей системе, он должен быть уникальным для каждого счета и генерироваться на вашей стороне |string|+
amount|Данные о сумме счета|Object|+
amount.value| Сумма, на которую выставляется счет, округленная в меньшую сторону до 2 десятичных знаков | Number(6.2)|+
amount.currency| Валюта суммы счета. Доступна только Рубли (`RUB`)| Alpha-3 ISO 4217 код |+
expirationDateTime | Дата, до которой счет будет доступен для оплаты. Если перевод не будет совершен до этой даты, ему присваивается финальный статус `EXPIRED` и последующий перевод станет невозможен.<br> |URL-закодированная строка<br>`ГГГГ-ММ-ДДTчч:мм:сс+\-чч:мм`|+
customer|Идентификаторы пользователя|Object|-
customer.phone | Номер телефона пользователя (в международном формате) |string|-
customer.email | E-mail пользователя |string|-
customer.account | Идентификатор пользователя в вашей системе|string|-
comment | Комментарий к счету|String(255)|-
customFields[]|Дополнительные данные счета. Так же при помощи этого параметра вы можете настроить [персонализацию](#custom) вашей формы, передав переменную `themeCode`|String(255)|-



<h3 class="request">Ответ ←</h3>

>Пример тела ответа

~~~json
  {
    "siteId": "23044",
    "billId": "cc961e8d-d4d6-4f02-b737-2297e51fb48e",
    "amount": {
      "value": 100,
      "currency": "RUB"
    },
    "status": {
      "value": "WAITING",
      "changedDateTime": "2018-03-05T11:27:41+03:00"
    },
    "comment": "Text comment",
    "creationDateTime": "2018-03-05T11:27:41",
    "expirationDateTime": "2018-04-13T14:30:00+03:00",
    "payUrl": "https://oplata.qiwi.com/form/?invoice_uid=d875277b-6f0f-445d-8a83-f62c7c07be77"
  }
~~~

>Пример тела ответа при ошибке

~~~json
{
	"serviceName": "invoicing-api",
	"errorCode": "auth.unauthorized",
	"description": "Неверные аутентификационные данные",
	"userMessage": "",
	"datetime": "2018-04-09T18:31:42+03:00",
	"traceId" : "48485a395dfsdf34v124"
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
billId|String|Идентификатор счета в вашей системе, он должен быть уникальным для каждого счета и генерироваться на вашей стороне
siteId|String|Ваш идентификатор в системе p2p.qiwi
amount|Object|Данные о сумме счета
amount.value|Number|Сумма счета, округленная до 2 знаков после запятой в меньшую сторону
amount.currency	|String|Валюта суммы счета (Alpha-3 ISO 4217 код)
status|Object|Данные о статусе счета
status.value	|String|Текущий [статус счета](#status)
status.changedDateTime|String|Дата обновления статуса. Формат даты:<br>`YYYY-MM-DDThh:mm:ss±hh`
customFields|Object|Дополнительные поля
customer|Object|Идентификаторы пользователя. Возможные опции: `email`, `phone`, `account`
comment|String|Комментарий к счету
creationDateTime|String| Системная дата создания счета. Формат даты:<br>`YYYY-MM-DDThh:mm:ss`
payUrl|String|[Ссылка](#option) на созданную форму
expirationDateTime|String|Срок действия созданной формы для перевода. Формат даты:<br>`YYYY-MM-DDThh:mm:ss+\-hh:mm`


## 2. Проверка статуса перевода по счету {#invoice-status}


Метод позволяет проверить статус перевода по счету. Рекомендуется его использовать после получения уведомления о переводе.

<h3 class="request method">Запрос → GET</h3>

~~~javascript
const billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

qiwiApi.getBillInfo(billId).then( data => {
    //do smth with data
});
~~~

~~~shell
curl https://api.qiwi.com/partner/bill/v1/bills/cc961e8d-d4d6-4f02-b737-2297e51fb48e \
-X GET \
-H 'Accept: application/json' \
-H 'Authorization: Bearer eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjIwNDIsImFwaV91c2VyX2lkIjo1NjYwMzk3Miwic2VjcmV0IjoiQjIwODlDNkI5Q0NDNTdCNDQzNGHJK43JFJDK595FJFJMjlCRkFFRDM5OE***********************'
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
             <li><strong>billId</strong> - уникальный идентификатор счета в вашей системе.</li>
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
    "siteId": "23044",
    "billId": "cc961e8d-d4d6-4f02-b737-2297e51fb48e",
    "amount": {
      "value": 100,
      "currency": "RUB"
    },
    "status": {
      "value": "WAITING",
      "changedDateTime": "2018-03-05T11:27:41+03:00"
    },
    "comment": "Text comment",
    "creationDateTime": "2018-03-05T11:27:41",
    "expirationDateTime": "2018-04-13T14:30:00+03:00",
    "payUrl": "https://oplata.qiwi.com/form/?invoice_uid=d875277b-6f0f-445d-8a83-f62c7c07be77"
  }
~~~

>Пример тела ответа при ошибке

~~~json
{
	"serviceName": "invoicing",
	"errorCode": "auth.unauthorized",
	"description": "Неверные аутентификационные данные",
	"userMessage": "",
	"datetime": "2018-04-09T18:31:42+03:00",
	"traceId" : ""
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
billId|String|Уникальный идентификатор выставляемого счета в вашей системе
siteId|String|Ваш идентификатор в системе p2p.qiwi
amount|Object|Данные о сумме счета
amount.value|Number|Сумма счета, округленная до 2 знаков после запятой в меньшую сторону.
amount.currency	|String|Идентификатор валюты суммы счета (Alpha-3 ISO 4217 код)
status|Object|Данные о статусе счета
status.value  |String|Текущий [статус счета](#status)
status.changedDateTime|String|Дата обновления статуса
customFields|Object|Объект строковых дополнительных параметров, переданных вами
customer|Object|Идентификаторы пользователя
customer.phone  |String |Номер телефона (если был указан при выставлении счета)
customer.email |String|E-mail пользователя (если был указан при выставлении счета)
customer.account |String| Идентификатор пользователя в вашей системе (если был указан при выставлении счета)
comment|String|Комментарий к счету
creationDateTime|String| Системная дата создания счета. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс`
payUrl|String|[Ссылка для переадресации](#option) пользователя на созданную форму
expirationDateTime|String|Срок действия созданной формы для перевода. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс+\-чч:мм`


## 3. Отмена неоплаченного счета {#cancel}

Метод позволяет отменить счет, по которому не был выполнен перевод.

<h3 class="request method">Запрос → POST</h3>

~~~javascript
const bill_id = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

qiwiApi.cancelBill(billId).then( data => {
    //do with data
});
~~~

~~~shell
curl https://api.qiwi.com/partner/bill/v1/bills/cc961e8d-d4d6-4f02-b737-2297e51fb48e/reject \
-X POST \
-H 'Accept: application/json' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjIwNDIsImFwaV91c2VyX2lkIjo1NjYwMzk3Miwic2VjcmV0IjoiQjIwODlDNkI5Q0NDNTdCNDQzNGHJK43JFJDK595FJFJMjlCRkFFRDM5OE***********************'
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
             <li><strong>billId</strong> - уникальный идентификатор счета в вашей системе.</li>
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
    "siteId": "23044",
    "billId": "cc961e8d-d4d6-4f02-b737-2297e51fb48e",
    "amount": {
      "value": 2.42,
      "currency": "RUB"
    },
    "status": {
      "value": "REJECTED",
      "changedDateTime": "2018-02-28T11:43:23.386+03:00"
    },
    "customer": {
      "email": "test@qiwi.com",
      "phone": "79191234567",
      "account": "user_account"
    },
    "customFields": {
      "city": "Moscow"
    },
    "comment": "Text comment",
    "creationDateTime": "2018-02-28T11:43:23.612+03:00",
    "expirationDateTime": "2018-04-14T11:43:23+03:00",
    "payUrl": "https://oplata.qiwi.com/form/?invoice_uid=6848dd49-e260-4343-b258-62199cffe8c1"
  }

~~~

>Пример тела ответа при ошибке

~~~json
{
	"serviceName": "invoicing",
	"errorCode": "auth.unauthorized",
	"description": "Неверные аутентификационные данные",
	"userMessage": "",
	"datetime": "2018-04-09T18:31:42+03:00",
	"traceId" : ""
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
billId|String|Уникальный идентификатор выставляемого счета в вашей системе
siteId|String|Ваш идентификатор в p2p.qiwi
amount|Object|Данные о сумме счета
amount.value|Number|Сумма счета, округленная до 2 знаков после запятой в меньшую сторону
amount.currency	|String|Идентификатор валюты суммы счета (Alpha-3 ISO 4217 код)
status|Object|Данные о статусе счета
status.value  |String|Текущий [статус счета](#status)
status.changedDateTime|String|Дата обновления статуса. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс+\-чч:мм`
customFields|Object|Объект строковых дополнительных параметров, переданных партнером
customer|Object|Идентификаторы пользователя. Возможные опции: `email`, `phone`, `account`
comment|String|Комментарий к счету
creationDateTime|String| Системная дата создания счета. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс`
payUrl|String|Ссылка на созданную платежную форму
expirationDateTime|String|Срок действия созданной формы для перевода. Формат даты:<br>`ГГГГ-ММ-ДДTчч:мм:сс+\-чч:мм`

### Статусы оплаты счетов {#status}

Статус|Описание|Финальный
------|--------|---------
WAITING | Счет выставлен, ожидает оплаты| -
PAID|Счет оплачен|+
REJECTED|Счет отклонен|+
EXPIRED	|Время жизни счета истекло. Счет не оплачен|+


# Уведомления о переводе по счету {#notification}

<aside class="notice">
Данный сервис не является обязательным для интеграции, вы можете реализовать более простой вариант с <a href="#invoice-status">опросом статуса счета</a>.
</aside>

<aside class="warning">
Уведомление о переводе (callback) отправляется только по протоколу HTTPS и только на 443 порт.
Сертификат должен быть выдан доверенным центром сертификации (напр., Comodo, Verisign, Thawte и т.п.)
</aside>


<h3 class="request method">Запрос ← POST</h3>

 >Пример уведомления

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: application/json
Content-type: application/json
X-Api-Signature-SHA256: J4WNfNZd***V5mv2w=
Host: server.ru

{ "bill":
  {  
     "siteId":"23044",
     "billId":"cc961e8d-d4d6-4f02-b737-2297e51fb48e",
     "amount":{  
        "value":"100",
        "currency":"RUB"
     },
     "status":{  
        "value":"PAID",
        "datetime":"2018-03-01T11:16:12"
     },
     "customer":{},
     "customFields":{},
     "creationDateTime":"2018-03-01T11:15:39",
     "expirationDateTime":"2018-04-01T11:15:39+03:00"
   },
  "version":"1"
}
~~~

Уведомление представляет собой входящий POST-запрос.

Тело запроса содержит JSON-сериализованные данные счета (кодировка UTF-8).

Адрес сервера для уведомлений вы указываете на <a href="https://p2p.qiwi.com/">p2p.qiwi.com</a> при [генерации ключей](#auth).

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
Заголовки по стандарту регистронезависимы и ваш клиент может их изменить - смотрите документацию вашего клиента.
</aside>

<aside class="notice">
Если в ответе код состояния HTTP отличается от 200 (OK), это интерпретируется как временная ошибка в вашей системе. Сервер QIWI будет повторять запрос с нарастающим интервалом в течение суток.
</aside>

<aside class="warning">
Обратите внимание, что уведомление может быть отправлено несколько раз даже в случае успешного ответа от вашего сервиса. Вам необходимо учитывать это при разработке бизнес-логики на вашей стороне.
</aside>


## Авторизация уведомлений {#notifications_auth}

После получения входящего уведомления необходимо проверить его подлинность. Для этого используется механизм цифровой подписи. Подпись уведомления отправляется в HTTP заголовке `X-Api-Signature-SHA256`. Для формирования подписи используется механизм проверки целостности HMAC с хэш-функцией SHA256.

Алгоритм проверки подписи:

1. Объединить значения следующих параметров уведомления в одну строку с разделителем `|`:

   `invoice_parameters = {amount.currency}|{amount.value}|{billId}|{siteId}|{status.value}`

   где `{*}` – значение параметра. Все значения при проверке подписи должны трактоваться как строки.


2. Вычислить HMAC-хэш c алгоритмом хэширования SHA256:

   `hash = HMAС(SHA256, invoice_parameters, secret_key)`
   Где:

   * `secret_key` – ключ функции ;
   * `invoice_parameters` – строка из п.1;

3. Сравнить значение заголовка X-Api-Signature-SHA256 с результатом из п.2.

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
    <li><h3>Данные</h3><span>В уведомлении содержится информация о счете.</span>
    </li>
</ul>

Поле|Описание|Тип
---------|--------|---
bill|Данные о счете|Object
billId|Уникальный идентификатор выставляемого счета в вашей системе|String(200)
siteId|Ваш идентификатор в системе p2p.qiwi| String
amount|Данные о сумме счета|Object
amount.value | Сумма счета, округленная до двух десятичных знаков в меньшую сторону | Number(6.2)
amount.currency | Идентификатор валюты суммы счета (Alpha-3 ISO 4217 код) | String(3)
status | Данные о статусе счета | Object
status.value |Строковое значение статуса | String
status.changedDateTime|Дата обновления статуса. Формат даты<br>`ГГГГ-ММ-ДДTЧЧ:ММ:ССZ`| String
customer | Данные о пользователе| Object
customer.phone |Номер телефона (если был указан при выставлении счета) |String
customer.email|E-mail пользователя (если был указан при выставлении счета)|String
customer.account| Идентификатор пользователя в вашей системе (если был указан при выставлении счета)|String
creationDateTime | Дата создания счета. Формат даты<br>`ГГГГ-ММ-ДДTЧЧ:ММ:ССZ`| String
expirationDateTime | Срок оплаты счета. Формат даты<br>`ГГГГ-ММ-ДДTЧЧ:ММ:СС+ЧЧ:ММ\-Z` |String
comment | Комментарий к счету | String(255)
customFields | Дополнительные данные счета (если были указаны при выставлении счета).| Object
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

После того, как был получен входящий запрос-уведомление, необходимо проверить подлинность цифровой подписи и отправить ответ.



# Настройки формы и счета {#option}


<aside class="notice">
При открытии формы в webview на android обязательно включать <code>settings.setDomStorageEnabled(true)</code>
</aside>

При [выставлении счета](#create) через API в ответе приходит `payUrl` с ссылкой на форму. К ссылке можно добавить следующие параметры:

> Пример ссылки

~~~shell
curl https://oplata.qiwi.com/form?invoiceUid=606a5f75-4f8e-4ce2-b400-967179502275&allowedPaySources=card
~~~

|Параметр | Описание | Тип|
|--------|----------------|---------|
| paySource | При открытии формы сразу будет выбран указанный способ перевода. Возможные значения:<br>`qw` - QIWI Кошелек<br>`card` - банковская карта<br>`mobile` - платеж со счета мобильного телефона<br> Если способ перевода недоступен - выбирается рекомендуемый для  данного пользователя способ | String |
| allowedPaySources | При открытии формы будут отображаться только указанные способы перевода. (Если они доступны) Возможные значения:<br>`qw` - QIWI Кошелек<br>`card` - банковская карта<br>`mobile` - платеж со счета мобильного телефона<br> | Строка с разделителями-запятыми |
| successUrl | URL для переадресации в случае успешного перевода с баланса QIWI Кошелька. При ином способе перевода переадресация не выполняется. Ссылка должна вести на ваш сайт. | URL-закодированная строка |
| lifetime | Дата, до которой счет будет доступен для перевода. Если перевод не будет осуществлен до этой даты, ему присваивается финальный статус EXPIRED и последующая оплата станет невозможна.| Строка формата<br>`ГГГГ-ММ-ДДTччмм` |

# Персонализация {#custom}

Персонализация  позволяет адаптировать платежную форму под ваш стиль. Настраивается наименование, фон и цвет кнопок.

Создать стили можно в личном кабинете на [p2p.qiwi.com](https://qiwi.com/p2p-admin/transfers/link). Возможно создать несколько стилей.

При настройке задается код, привязанный к стилю (например, `кодСтиля`). Для использования стиля на платежной форме необходимо передать в параметре `customFields` [запроса создания счета](#create) или [вызова платежной формы](#http) переменную: `"themeCode":"кодСтиля"` с соответствующим кодом стиля. 

 >Пример передачи параметра при вызове платежной формы

~~~shell
curl https://oplata.qiwi.com/create?publicKey=Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******&amount=100&billId=cc961e8d-d4d6-4f02-b737-2297e51fb48e&successUrl=http%3A%2F%2Ftest.ru%3F&customFields%5BthemeCode%5D=кодСтиля
~~~

 >Пример передачи параметра в запросе к API

~~~shell
curl https://api.qiwi.com/partner/bill/v1/bills/cc961e8d-d4d6-4f02-b737-2297e51fb48e \
-X PUT \
-H 'Accept: application/json' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjIwNDIsImFwaV91c2VyX2lkIjo1NjYwMzk3Miwic2VjcmV0IjoiQjIwODlDNkI5Q0NDNTdCNDQzNGHJK43JFJDK595FJFJMjlCRkFFRDM5OE***********************'
-d '{ \
   "amount": {  \
     "currency": "RUB",  \
     "value": 100.00 \
   }, \
   "comment": "Text comment", \
   "expirationDateTime": "2018-04-13T14:30:00+03:00", \
   "customer": {}, \
   "customFields": {"themeCode":"кодСтиля"} \
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
  'email' => 'example@mail.org',
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
            Email = "example@mail.org",
            Account = Guid.NewGuid().ToString(),
            Phone = "79123456789"
        },
        SuccessUrl = new Uri("http://merchant.ru/success")
    },
    customFields: new CustomFields
    {
        ThemeCode = "кодСтиля"
    }
);
~~~

![Customer form](/images/Custom.png)

# Checkout Popup {#popup}

<button id="pop" class="button-popup" onclick="testPopup();">Пример работы popup</button>

Всплывающее окно (popup) позволяет открыть форму перевода поверх вашего сайта.
В библиотеке доступно два метода: создание нового счета и открытие существующего.

[Скачать библиотеку QIWI Checkout Popup](https://github.com/QIWI-API/qiwi-invoicing-popup)


Установка и подключение:<br>
`<script src='https://oplata.qiwi.com/popup/v1.js'></script>`

##  Выставление нового счета  {#createpopup}
Метод  `QiwiCheckout.createInvoice`


>Пример выставления счета через popup

~~~ppp
QiwiCheckout.createInvoice({
    publicKey: '5nAq6abtyCz4tcDj89e5w7Y5i524LAFmzrsN6bQTQ3c******',
    amount: 1.23,
    phone: '79123456789',
})
    .then(data => {
        //  data === {
        //    publicKey: '5nAq6abtyCz4tcDj89e5w7Y5i524LAFmzrsN6bQTQ3c******',
        //    amount: 1.23,
        //    phone: '79123456789',
        //  }
    })
    .catch(error => {
        //  error === {
        //      reason: "PAYMENT_FAILED"
        //  }
    })
~~~

|Параметр | Описание | Тип | Обязательное|
|------------|-----------|-------------|--------------|
| publicKey | Ваш ключ идентификации , полученный в p2p.qiwi | String | + |
| amount | Сумма, на которую выставляется счет, округленная в меньшую сторону до 2 десятичных знаков | Number(6.2) | + |
| phone | Номер телефона пользователя (в международном формате) | String | - |
| email | E-mail пользователя| String | - |
| account | Идентификатор пользователя в вашей системе| String | - |
| comment | Комментарий к счету | String(255) | - |
| customFields | Дополнительные данные счета | Object | - |
| lifetime | Дата, до которой счет будет доступен для перевода. Если перевод не будет осуществлен до этой даты, ему присваивается финальный статус EXPIRED и последующая оплата станет невозможна.| Строка в виде `ГГГГ-ММ-ДДTччмм` | - |

##  Открытие существующего счета {#openpopup}

Метод  `QiwiCheckout.openInvoice`

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

|Параметр | Описание | Тип | Обязательное|
|---------|----------|----------|---------|
| payUrl | URL инвойса | String | + |

