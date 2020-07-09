---
title: API P2P Invoices 1.0.0

search: true

metatitle: API P2P Invoices  1.0.0

metadescription: API P2P Invoices opens a way to operate with invoices from your service or application. Invoice is the unique request for the payment. By default, the user may pay the invoice with any accessible means. API supports issuing and cancelling invoices, making refunds and checking operation status.

language_tabs:
  - shell
  - javascript: Node.js SDK
  - php: PHP SDK
  - java: Java SDK
  - ppp: Popup
  - csharp: .Net SDK

services:
 - <a href='#'>Swagger</a>  |  <a href='#'>Qiwi Demo</a>

toc_footers:
 - <a href='/en/'>Home page</a>
 - <a href='https://tele.click/qiwi_api_help_bot'>Feedback</a>

---

# API Basics {#intro}

###### Last update: 2020-06-04 | [Edit on GitHub](https://github.com/QIWI-API/p2p-payments-docs/blob/master/p2p-payments_en.html.md)

P2P Invoices API opens a way to operations with invoices from your service or application. Invoice is the unique request for the payment. The user may pay the invoice with any accessible means. API supports  issuing and cancelling invoices, making refunds and checking operation status.

**To use API,  you need public and secret keys. Keys are available after registration on [p2p.qiwi.com](https://p2p.qiwi.com).**

## Invoicing Operations Flow {#steps}


<div class="mermaid">
sequenceDiagram
participant user as User
participant rec as Recipient
participant p2p as QIWI P2P API
opt Basic scenario
user->>rec:Doing order
activate user
activate rec
opt Using API
rec->>p2p:Invoice issue
p2p->>rec:Link to Payment form
end
rec->>user:Redirect to Payment form<br>Invoice issue via the form<br>oplata.qiwi.com
deactivate rec
user->>p2p:Opening the Payment form<br>Choose payment method / Paying for invoice
p2p->>p2p:Invoice payment
opt Redirect to success page
p2p->>user:Redirect to success_url page
end
deactivate user
end
opt Additional
opt Notifications (callback)
activate p2p
activate rec
p2p->>rec:Invoice status notification
rec->>p2p:Server response (OK)
deactivate rec
deactivate p2p
end
opt Invoice status check
activate p2p
activate rec
rec->>p2p:Check invoice status
p2p->>rec:Invoice data
deactivate rec
deactivate p2p
end
end
</div>

<!-- ![Operation Flow](/images/bill_payments_en.png) -->

* User submits an order on the merchant’s website.

* Merchant redirects the user to [Payment Form](#http) link. It automatically issues an invoice for the order. Or you may [issue an invoice by API](#create) and [redirects to the Payment Form](#option).

* The user chooses the most convenient way to pay for the invoice on the Payment Form. By default, the optimal payment method is showed first.

* The merchant's service receives [notification](#notification) once the invoice is successfully paid by the user. You need to configure notifications on your [Personal Page](https://p2p.qiwi.com). Notifications contain authorization parameters which merchant needs to verify on its server.

* If needed, via the API merchant can:
  * [request current status](#invoice-status) of the invoice,
  * [cancel invoice](#cancel) (if the user has not initiated payment yet).

* When the invoice payment is confirmed, merchant delivers ordered services/goods.


## Authorization {#auth}

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

API requests are authorized by the API secret key (`SECRET_KEY`). Put this parameter to <i>Authorization</i> header as "Bearer SECRET_KEY".

Public key (`PUBLIC_KEY`) is used when issuing invoices via [the Payment Form](#http).

**Keys are available after registration on [p2p.qiwi.com](https://p2p.qiwi.com).**

<aside class="warning">
Do not share secret key to third parties!
</aside>

# Invoice Issue on Payment Form {#http}

<aside class="notice">
When opening Payment Form in Webview on Android, you should enable <code>settings.setDomStorageEnabled(true)</code>
</aside>

It is the simplest way of integration. On opening Payment Form, client receives an invoice at the same time. The invoice data sends in URL explicitly. Client gets a Payment Form web page with multiple payment means. When using  this method, one cannot be sure that all invoices are issued by the merchant. [API invoice creation](#create) mitigates this risk.

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

$publicKey = '2tbp1WQvsgQeziGY9vTLe9vDZNg7tmCymb4Lh6STQokqKrpCC6qrUUKEDZAJ7mvFnzr1yTebUiQaBLDnebLMMxL8nc6FF5zf******';
$params = [
  'publicKey' => $publicKey,
  'amount' => 200,
  'billId' => '893794793973'
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
String successUrl = "https://merchant.com/payment/success?billId=893794793973";
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
var successUrl = "https://merchant.com/payment/success?billId=893794793973";

var paymentUrl = client.createPaymentForm(new PaymentInfo(key, amount, billId, successUrl));
~~~


<h3 class="request method">REDIRECT → </h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://oplata.qiwi.com/create</span></h3></li>
</ul>

~~~javascript
const publicKey = 'Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******';

const params = {
    publicKey,
    amount: 42.24,
    billId: '893794793973',
    successUrl: 'http://test.ru/',
    email: 'm@ya.ru'
};

const link = qiwiApi.createPaymentForm(params);
~~~

~~~shell
curl https://oplata.qiwi.com/create?publicKey=Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******&amount=100&billId=893794793973&successUrl=http%3A%2F%2Ftest.ru%3F&email=m@ya.ru
~~~

<aside class="notice">
When opening Payment Form in Webview on Android, you should enable <code>settings.setDomStorageEnabled(true)</code>
</aside>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Invoice data are put in Payment Form URL.</span></li>
</ul>

Parameter|Description|Type|Required
---------|--------|---|---------|---|----
publicKey | Merchant public key received in p2p.qiwi|String|+
billId|Unique invoice identifier in merchant's system|URL-encoded, String(200)|-
amount| Amount of the invoice rounded down on two decimals | Number(6.2)|-
phone | Phone number of the client to which the invoice is issuing (international format) | URL-encoded string|-
email | E-mail of the client where the invoice payment link will be sent | URL-encoded string|-
account | Client identifier in merchant's system | URL-encoded string |-
comment | Invoice commentary|URL-encoded, String(255)|-
customFields[]|Additional invoice data|URL-encoded, String(255)|-
lifetime | Expiration date of the pay form link (invoice payment's due date). If the invoice is not paid after that date, the invoice assigns `EXPIRED` final status and it becomes void.<br> **Important! Invoice will be automatically expired when 45 days is passed after the invoicing date**|URL-encoded string<br>`YYYY-MM-DDThhmm`|-
successUrl|The URL to which the client will be redirected in case of successful payment from its QIWI Wallet balance. When payment is by any other means, redirection is not performed. URL must be within merchant's site.|URL-encoded string|-


# API Operations {#API}

## 1. Invoice Issue by API {#create}

It is the reliable method for integration. Parameters are sent by means of server2server requests with authorization. Method allows you to issue an invoice, successful response contains `payUrl` link to redirect client on Payment Form.

**[Additional features](#option)**

<aside class="notice">
For testing purposes, you can always create and pay bills for 1 ruble.
</aside>

<h3 class="request method">Request → PUT</h3>

~~~javascript
const billId = '893794793973';

const fields = {
    amount: 1.00,
    currency: 'RUB',
    comment: 'Hello world',
    expirationDateTime: '2018-03-02T08:44:07'
};

qiwiRestApi.createBill( billId, fields ).then( data => {
    //do smth with data
});
~~~

~~~shell
curl https://api.qiwi.com/partner/bill/v1/bills/893794793973 \
-X PUT \
-H 'Accept: application/json' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjIwNDIsImFwaV91c2VyX2lkIjo1NjYwMzk3Miwic2VjcmV0IjoiQjIwODlDNkI5Q0NDNTdCNDQzNGHJK43JFJDK595FJFJMjlCRkFFRDM5OE***********************'
-d '{ \
   "amount": { \
     "currency": "RUB", \
     "value": 100.00 \
   }, \
   "comment": "Text comment", \
   "expirationDateTime": "2018-04-13T14:30:00+03:00", \
   "customer": {}, \
   "customFields": {} \
   }'
~~~

~~~php
<?php

$billId = '893794793973';
$fields = [
  'amount' => 1.00,
  'currency' => 'RUB',
  'comment' => 'test',
  'expirationDateTime' => '2018-03-02T08:44:07',
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
var billInfo = new CreateBillInfo
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
};
var response = client.createBill(billInfo);
~~~


<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/partner/bill/v1/bills/{billid}</span></h3>
        <ul>
             <li><strong>billId</strong> - unique invoice identifier generated by the merchant.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer <a href="#auth">SECRET_KEY</a></li>
             <li>Accept: application/json</li>
             <li>Content-Type: application/json</li>
        </ul>
    </li>
</ul>

Parameter|Description|Type|Required
---------|--------|---|---------|---|----
billId|Unique invoice identifier in merchant's system|String(200)|+
amount|Object|Data of the invoice amount|+
amount.currency| Invoice amount currency code. Only `RUB` | Alpha-3 ISO 4217 code |+
amount.value| Amount of the invoice rounded down to two decimals | Number(6.2)|+
expirationDateTime |  Invoice due date. Time should be specified with time zone. |string<br>`YYYY-MM-DDThhmm+\-hh:mm`|+
customer|Object | Customer data of the invoice subject|-
customer.phone | Phone number of the client to which the invoice is issuing (international format) |string|-
customer.email | E-mail of the client where the invoice payment link will be sent |string|-
customer.account | Client identifier in merchant's system |string |-
comment | Invoice commentary|String(255)|-
customFields[]|Additional invoice data|String(255)|-


<h3 class="request">Response ←</h3>

>Successful response body example

~~~json
  {
    "siteId": "23044",
    "billId": "893794793973",
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
    "expirationDateTime": "2018-04-13T14:30:00",
    "payUrl": "https://oplata.qiwi.com/form/?invoice_uid=d875277b-6f0f-445d-8a83-f62c7c07be77"
  }
~~~

>Error response body

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

Field|Type|Description
--------|---|--------
billId|String|Unique invoice identifier in the merchant's system
siteId|String|Merchant's site identifier in p2p.qiwi
amount|Object|Data of the invoice amount
amount.value|String|The invoice amount. The number is rounded down to two decimals
amount.currency|String|Currency identifier of the invoice amount (Alpha-3 ISO 4217 code)
status|Object|Data of the invoice current status
status.value|String | String representation of the status. [Possible statuses](#status)
status.changedDateTime|String|Status refresh date. Date format:<br>`YYYY-MM-DDThh:mm:ss±hh`
customer|Object | Customer data of the invoice subject
customer.phone|String | The customer’s phone (if specified in the invoice)
customer.email|String|The customer's e-mail  (if specified in the invoice)
customer.account| String|The customer's identifier in the merchant's system (if specified in the invoice)
customFields|Object|Additional invoice data provided by the merchant
comment|String|Comment to the invoice
creationDateTime|String|System date of the invoice creation. Date format:<br>`YYYY-MM-DDThh:mm:ss±hh`
payUrl|String|Pay form link
expirationDateTime|String|Expiration date of the pay form link (invoice payment's due date). Date format:<br>`YYYY-MM-DDThh:mm:ss±hh`


## 2. Checking the Invoice Status {#invoice-status}

Use this method to get current invoice payment status. We recommend using it after receiving the payment notification.

<h3 class="request method">Request → GET</h3>

~~~javascript
const billId = '893794793973';

qiwiApi.getBillInfo(billId).then( data => {
    //do smth ith data
});
~~~

~~~shell
curl https://api.qiwi.com/partner/bill/v1/bills/893794793973 \
-X GET \
-H 'Accept: application/json' \
-H 'Authorization: Bearer eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjIwNDIsImFwaV91c2VyX2lkIjo1NjYwMzk3Miwic2VjcmV0IjoiQjIwODlDNkI5Q0NDNTdCNDQzNGHJK43JFJDK595FJFJMjlCRkFFRDM5OE***********************'
~~~

~~~php
<?php

$billId = '893794793973';

/** @var \Qiwi\Api\BillPayments $billPayments */
$response = $billPayments->getBillInfo($billId);

print_r($response);

?>
~~~

~~~java
String billId = "fcb40a23-6733-4cf3-bacf-8e425fd1fc71";
 BillResponse response = client.getBillInfo(billId);
~~~

~~~csharp
var billId = "fcb40a23-6733-4cf3-bacf-8e425fd1fc71";

var response = client.getBillInfo(billId);
~~~


<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/partner/bill/v1/bills/{billid}</span></h3>
        <ul>
             <li><strong>billId</strong> - unique invoice identifier generated by the merchant.</li>
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

<h3 class="request">Response ←</h3>

>Successful response body example

~~~json
 {
    "siteId": "23044",
    "billId": "893794793973",
    "amount": {
      "value": 2.42,
      "currency": "RUB"
    },
    "status": {
      "value": "WAITING",
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

>Error response body example

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

Field|Type|Description
--------|---|--------
billId|String|Unique invoice identifier in the merchant's system
siteId|String|Merchant's site identifier in p2p.qiwi
amount|Object|The invoice amount data
amount.value|Number|The invoice amount. The number is rounded down to two decimals
amount.currency|String|Currency identifier of the invoice amount (Alpha-3 ISO 4217 code)
status|Object|Invoice status data
status.value|String|Current [invoice status](#status)
status.changedDateTime|String|Status refresh date
customFields|Object|Additional invoice data provided by the merchant
customer |Object | Customer data of the invoice subject
customer.phone|String | The customer’s phone (if specified in the invoice)
customer.email|String|The customer's e-mail  (if specified in the invoice)
customer.account|String|The customer's identifier in the merchant's system (if specified in the invoice)
comment|String|Comment to the invoice
creationDateTime|String|System date of the invoice creation. Date format:<br>`YYYY-MM-DDThh:mm:ss`
payUrl|String|Pay form link
expirationDateTime|String|Expiration date of the pay form link (invoice payment's due date). Date format:<br>`YYYY-MM-DDThh:mm:ss`

## 3. Cancelling the Invoice {#cancel}

Use this method to cancel unpaid invoice.

<h3 class="request method">Request → POST</h3>

~~~javascript
const bill_id = '893794793973';

qiwiApi.cancelBill(billId).then( data => {
    //do smth with data
});
~~~

~~~shell
curl https://api.qiwi.com/partner/bill/v1/bills/893794793973/reject \
-X POST \
-H 'Accept: application/json' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjIwNDIsImFwaV91c2VyX2lkIjo1NjYwMzk3Miwic2VjcmV0IjoiQjIwODlDNkI5Q0NDNTdCNDQzNGHJK43JFJDK595FJFJMjlCRkFFRDM5OE***********************'
~~~

~~~php
<?php

$billId = '893794793973';

/** @var \Qiwi\Api\BillPayments $billPayments */
$response = $billPayments->cancelBill($billId);

print_r($response);

?>
~~~

~~~java
String billId = "fcb40a23-6733-4cf3-bacf-8e425fd1fc71";
 BillResponse response = client.cancelBill(billId);
~~~

~~~csharp
var billId = "fcb40a23-6733-4cf3-bacf-8e425fd1fc71";

var response = client.cancelBill(billId);
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://api.qiwi.com/partner/bill/v1/bills/{billId}/reject</span></h3>
        <ul>
        <strong>Parameters:</strong>
             <li><strong>billId</strong> - unique invoice identifier generated by the merchant.</li>
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

<h3 class="request">Response ←</h3>

>Successful response body example

~~~json
{
    "siteId": "23044",
    "billId": "893794793973",
    "amount": {
      "value": 2.42,
      "currency": "RUB"
    },
    "status": {
      "value": "REJECTED",
      "datetime": "2018-02-28T11:43:23"
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
    "creationDateTime": "2018-02-28T11:43:23",
    "expirationDateTime": "2018-04-14T11:43:23",
    "payUrl": "https://oplata.qiwi.com/form/?invoice_uid=6848dd49-e260-4343-b258-62199cffe8c1"
}
~~~

>Error response body example

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

Field|Type|Description
--------|---|--------
billId|String|Unique invoice identifier in the merchant's system
siteId|String|Merchant's site identifier in p2p.qiwi
amount|Object|The invoice amount data
amount.value|Number|The invoice amount. The number is rounded down to two decimals
amount.currency|String|Currency identifier of the invoice amount (Alpha-3 ISO 4217 code)
status|Object|Invoice status data
status.value|String|Current [invoice status](#status)
status.changedDateTime|String|Status refresh date
customFields|Object|Additional invoice data provided by the merchant
customer |Object | Customer data of the invoice subject
customer.phone|String | The customer’s phone (if specified in the invoice)
customer.email|String|The customer's e-mail  (if specified in the invoice)
customer.account|String|The customer's identifier in the merchant's system (if specified in the invoice)
comment|String|Comment to the invoice
creationDateTime|String|System date of the invoice creation. Date format:<br>`YYYY-MM-DDThh:mm:ss`
payUrl|String|Pay form link
expirationDateTime|String|Expiration date of the pay form link (invoice payment's due date). Date format:<br>`YYYY-MM-DDThh:mm:ss`



### Invoice Payment Statuses {#status}

Status|Description|Final
------|--------|---------
WAITING | Invoice issued awaiting for payment| -
PAID|Invoice paid|+
REJECTED|Invoice rejected by customer|+
EXPIRED	|Invoice expired. Invoice not paid|+

# Invoice Payment Notifications {#notification}

<aside class="notice">
Notifications handler service on your side is not required for the integration. You can implement <a href="#invoice-status">invoice status polling</a> instead.
</aside>

<aside class="warning">
Callback is sent by HTTPS protocol on 443 port only.
Certificate should be issued by any trusted center of certification (e.g. Comodo, Verisign, Thawte etc)
</aside>

<h3 class="request method">Request ← POST</h3>

 >Notification example

~~~http
POST /qiwi-notify.php HTTP/1.1
Accept: application/json
Content-type: application/json
X-Api-Signature-SHA256: J4WNfNZd***V5mv2w=
Host: server.ru

{ "bill":
  {
     "siteId":"23044",
     "billId":"1519892138404fhr7i272a2",
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
     "expirationDateTime":"2018-04-01T11:15:39"
   },
  "version":"1"
}
~~~

Notification is an incoming POST-request (callback). The request's body contains JSON-serialized invoice data encoded by UTF-8.

Notifications handler server URL is specified on <a href="https://p2p.qiwi.com/">p2p.qiwi.com</a> in **API** section.

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>X-Api-Signature-SHA256: XXX</li>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
        </ul>
    </li>
</ul>

## Authorization on Merchant's Server {#notifications_auth}

Upon receiving inbound notification you need to verify it by digital signature from notification HTTP header `X-Api-Signature-SHA256`. Signature is verified with HMAC algorithm integrity check with SHA256-hash function.

Signature verification algorithm is as follows:

1. Prepare a string of the following notification's parameters separated by `|`:

    `invoice_parameters = {amount.currency}|{amount.value}|{billId}|{siteId}|{status.value}`

    where `{*}` is the value of the parameter. All values should be treated as strings.

2. Apply HMAC-SHA256 function:

    `hash = HMAС(SHA256, invoice_parameters, secret_key)`

    where:

    * `secret_key` – function key;
    * `invoice_parameters` – string from step 1.

3. Compare `X-Api-Signature-SHA256` header's value with the result of step 2.

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
        creationDateTime: '2018-03-01T11:15:39+03',
        expirationDateTime: '2018-04-15T11:15:39+03'
    },
    version: '1'
};

const merchantSecret = 'test-merchant-secret-for-signature-check';

qiwiApi.checkNotificationSignature(
    validSignatureFromNotificationServer, notificationData, merchantSecret
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
$merchantSecret = 'test-merchant-secret-for-signature-check';

/** @var \Qiwi\Api\BillPayments $billPayments */
$billPayments->checkNotificationSignature(
  $validSignatureFromNotificationServer, $notificationData, $merchantSecret
); // true

?>
~~~

~~~java
String merchantSecret = "test-merchant-secret-for-signature-check";
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
 BillPaymentsUtils.checkNotificationSignature(validSignature, notification, merchantSecret); //true
~~~


String and key of the signature are encoded in UTF-8.

<ul class="nestedList params">
    <li><h3>Data</h3><span>Invoice data are in the notification's body.</span>
    </li>
</ul>


Field|Description|Type
---------|--------|---
bill|Invoice data|Object
bill.billId|Invoice identifier in the merchant's system|String(200)
bill.siteId|Merchant's site identifier in p2p.qiwi |String
bill.amount|The invoice amount data|Object
amount.value|The invoice amount. The number is rounded down to two decimals|Number(6.2)
amount.currency|Currency identifier of the invoice amount (Alpha-3 ISO 4217 code)|String(3)
bill.status|Invoice status data|Object
status.value|Current [invoice status](#status)|String
status.changedDateTime|Status refresh date. Date format:<br>`YYYY-MM-DDThh:mm:ssZ`|String
bill.customFields|Additional invoice data provided by the merchant|Object
bill.customer | Customer data of the invoice subject  (if specified in the invoice)|Object
customer.phone | The customer’s phone (if specified in the invoice)|String
customer.email|The customer's e-mail  (if specified in the invoice)|String
customer.account|The customer's identifier in the merchant's system (if specified in the invoice)| String
bill.comment|Comment to the invoice|String(255)
bill.creationDateTime|System date of the invoice creation. Date format:<br>`YYYY-MM-DDThh:mm:ssZ`|String
bill.payUrl|Pay form link|String
bill.expirationDateTime|Expiration date of the pay form link (invoice payment's due date). Date format:<br>`YYYY-MM-DDThh:mm:ssZ`|String
version | Notification service version | String

<h3 class="request">Response → </h3>


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

After receiving incoming notification request, you should verify its signature and returns the JSON-response. The processing result code should be returned in response.

<aside class="notice">
Any response with HTTP status code other than 200 (OK) will be treated as a temporary merchant's service error. QIWI server repeats the notification request with increasing period within the next 24 hours.
</aside>


# Personalization {#custom}

Personalization allows you to create a payment form with your style, customizable logo, background and color of the buttons.  

You can create styles in your account on [p2p.qiwi.com](https://p2p.qiwi.com). It is possible to create several styles.

When setting up, you create a code linked to the style (for example, `codeStyle`). To use style on the Payment Form, you must pass the variable: `"themeCode": "codeStyle"` with respective code of the style in the `customFields` parameter of the [invoice request](#create) or  [opening Payment Form URL](#http).

 >Invoice Issue on Payment Form

~~~shell
curl https://oplata.qiwi.com/create?publicKey=Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******&amount=100&billId=893794793973&successUrl=http%3A%2F%2Ftest.ru%3F&customFields%5BthemeCode%5D=codeStyle
~~~

 >Invoice Issue by API

~~~shell
curl https://api.qiwi.com/partner/bill/v1/bills/893794793973 \
-X PUT \
-H 'Accept: application/json' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer eyJ2ZXJzaW9uIjoicmVzdF92MyIsImRhdGEiOnsibWVyY2hhbnRfaWQiOjIwNDIsImFwaV91c2VyX2lkIjo1NjYwMzk3Miwic2VjcmV0IjoiQjIwODlDNkI5Q0NDNTdCNDQzNGHJK43JFJDK595FJFJMjlCRkFFRDM5OE***********************' \
-d '{ \
   "amount": {  \
     "currency": "RUB",  \
     "value": 100.00 \
   }, \
   "comment": "Text comment", \
   "expirationDateTime": "2018-04-13T14:30:00+03:00", \
   "customer": {}, \
   "customFields": {"themeCode":"codeStyle"} \
 }'
~~~

![Customer form](/images/Custom.png)

# Checkout Popup {#popup}

<button id="pop" class="button-popup" onclick="testPopup();">Demo popup</button>


[Download QIWI Checkout Popup ](https://github.com/QIWI-API/qiwi-invoicing-popup)

Installation:
<script src='https://oplata.qiwi.com/popup/v1.js'></script>


The library has two methods: create a new invoice and open an existing one.

##  Create new invoice {#createpopup}

Call function  `QiwiCheckout.createInvoice`.

| Parameter | Description | Type | Required |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|--------------|
| publicKey | Merchant public key received in p2p.qiwi | String | + |
| amount | Amount of the invoice rounded down on two decimals | Number(6.2) | + |
| phone | Phone number of the client to which the invoice is issuing (international format) | String | - |
| email | E-mail of the client where the invoice payment link will be sent | String | - |
| account | Client identifier in merchant’s system | String | - |
| comment | Invoice commentary | String(255) | - |
| customFields | Additional invoice data | Object | - |
| lifetime | Expiration date of the pay form link (invoice payment’s due date). If the invoice is not paid after that date, the invoice assigns `EXPIRED` final status and it becomes void.<br>**Important!** Invoice will be automatically expired when 45 days is passed after the invoicing date| URL-encoded string `YYYY-MM-DDThhmm` | - |

>Create new invoice

~~~ppp
params = {
    publicKey: '5nAq6abtyCz4tcDj89e5w7Y5i524LAFmzrsN6bQTQ3c******',
    amount: 1.23,

    phone: '79123456789',
    email: 'test@test.com',
    account: 'acc789',
    comment: 'Оплата',
    customFields: {
        data: 'data'
    },
    lifetime: '2019-04-04T1540'
}

QiwiCheckout.createInvoice(params)
    .then(data => {
        // ...
    })
    .catch(error => {
        // ...
    })
~~~


##  Open an existing invoice {#openpopup}

Call function  `QiwiCheckout.openInvoice`.

| Parameter | Description | Type | Required |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|--------------|
| payUrl | Pay form link| String | + |

>Open an existing one

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



# Payment Form options {#option}

<aside class="notice">
When opening Payment Form in Webview on Android, you should enable <code>settings.setDomStorageEnabled(true)</code>
</aside>


> Invoice URL example

~~~shell
curl https://oplata.qiwi.com/form?invoiceUid=606a5f75-4f8e-4ce2-b400-967179502275&allowedPaySources=card
~~~

You can add parameters to URL from `payUrl` field in response to the [invoice request](#create).

| Parameter | Description | Type |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|
| paySource |Pre-selected payment method for the client on Payment Form. Possible values: <br>`qw` - QIWI Wallet<br>`card` - card payment <br>`mobile` - mobile account payment <br>`sovest` - Sovest card payment <br> When specified method is inaccessible, the page automatically selects recommended method for the user.| String |
| allowedPaySources |Allow only these payment methods for the client on Payment Form. Possible values: <br>`qw` - QIWI Wallet<br>`card` - card payment <br>`mobile` - mobile account payment <br>`sovest` - Sovest card payment <br> | comma separated string |
| successUrl | The URL to which the client will be redirected in case of successful payment from its QIWI Wallet balance. When payment is by any other means, redirection is not performed. URL must belong to the merchant. | URL-encoded string |
| lifetime | Expiration date of the pay form link (invoice payment’s due date). If the invoice is not paid after that date, the invoice assigns EXPIRED final status and it becomes void. Important! Invoice will be automatically expired when 45 days is passed after the invoicing date| String<br>`YYYY-MM-DDThhmm` |

# SDK and CMS

## SDK and Libraries {#sdk}

* [NODE JS SDK](https://github.com/QIWI-API/bill-payments-node-js-sdk) - Node JS package of ready-to-use solutions for server2server integration development.
* [PHP SDK](https://github.com/QIWI-API/bill-payments-php-sdk) -  PHP package of ready-to-use solutions for server2server integration development.
* [Java SDK](https://github.com/QIWI-API/bill-payments-java-sdk) - Java package of ready-to-use solutions for server2server integration development.
* [.Net SDK ](https://github.com/QIWI-API/bill-payments-dotnet-sdk) - C# .net package of ready-to-use solutions for server2server integration development.

## CMS Solutions {#cms}

* [Wordpress](https://wordpress.org/plugins/woo-qiwi-payment-gateway/) -  plugin for Woocommerce for work with orders
* [Online Leyka](https://wordpress.org/plugins/leyka/) -  Wordpress plagin for charity
* [1С-Bitrix](http://marketplace.1c-bitrix.ru/solutions/qiwikassa.checkout/) - plugin for work with orders
* [Opencart](https://www.opencart.com/index.php?route=marketplace/extension/info&member_token=nH5fDsH3A5OkPF4zOe82hS0ypOhIqSEr&extension_id=36833) - plugin for work with orders
* [PrestaShop](https://github.com/QIWI-API/prestashop-payment-qiwi/releases) - plugin for work with orders
