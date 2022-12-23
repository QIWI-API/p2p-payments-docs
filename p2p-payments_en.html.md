---
title: API P2P Invoices 1.0.0

search: true

metatitle: API P2P Invoices  1.0.0

metadescription: API P2P Invoices opens a way for individuals to operate with invoices from their service or application. An invoice is the unique request for the money transfer. By default, the customer may pay the invoice with any accessible means. API supports issuing and cancelling invoices, and checking operation status.

category: p2p

language_tabs:
  - shell
  - php: PHP SDK
  - javascript: Node.js SDK
  - java: Java SDK
  - csharp: .Net SDK
  - ppp: Popup

toc_footers:
 - <a href='/en/'>Home page</a>
 - <a href='https://t.me/qiwi_api_help_bot'>Feedback</a>
 - <a href='https://github.com/QIWI-API/p2p-payments-docs/'>This page on Github</a>


---

# P2P Invoices

###### Last update: 2022-06-06

## Terms of Service {#terms}

To enable p2p-transfer service for individuals on your site, you need a QIWI wallet with **Basic** or **Professional** identification status. If your wallet has **Anonymous** status, pass identification with any means suitable for you:

* To get **Basic** status, you need to [enter your passport information](https://qiwi.com/settings/identification/form).
* To get **Professional** status, you need to [pass identification on-site](https://qiwi.com/settings/identification/full-ru).

We recommend to get **Professional** status, as it applies higher value of limits for remains on your balance, payments and transfers amount per month, and maximum amount of a single operation. See details of limits in the [documentation](https://qiwi.com/settings/identification/).

Take a look at [frequently asked questions](https://qiwi.com/support/products/p2p/faq) on the service, and get an understanding on how to avoid blocking your wallet from the [documentation](https://qiwi.com/support/products/p2p/kak_izbezhat_blokirovki_koshelka).

## Get access to the service {#p2p-activate}

1. Authorize on [p2p.qiwi.com](https://p2p.qiwi.com).
2. Make sure you have access to invoice creation – on the [invoicing form](https://qiwi.com/p2p-admin/transfers/invoice) fill in _Amount_ field and click **Create invoice** button. Below the link to the invoice and **Copy link** button would be displayed.

![invoice-test](/images/p2p-api/invoice-create-test.png)

**Congratulations! You can start service integration.**

## How to start working with the service {#how-to-start}

1. [Create public and secret keys](#auth).
2. Implement [API operations](#create) or [invoicing payment form call](#http-invoice). Use [SDK](#sdk) or various [CMS solutions](#cms).
3. To get notifications after invoice payment, implement its processing and [activate sending notifications](#notification-server).
4. Start accepting payments from cards and QIWI wallets.

## Invoicing Operations Flow {#payment-scenario}

<div class="mermaid">
sequenceDiagram
participant user as User
participant rec as Recipient
participant p2p as QIWI P2P API
opt Basic scenario
user->>rec:Doing order
activate user
activate rec
alt Using API
rec->>p2p:Invoice issue
p2p->>rec:Link to Payment form
rec->>user:Redirect to Payment form<br>oplata.qiwi.com
end
alt Using Payment form
rec->>user:Invoice issue on the form<br>oplata.qiwi.com
end
deactivate rec
user->>p2p:Choose payment method / Paying for invoice
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

1. User submits an order on the merchant’s website.

2. Merchant redirects the user to [Payment Form](#http) link. It automatically issues an invoice for the order. Or you may [issue an invoice by API](#create) and [redirect the user to the created Payment Form](#payUrl) (link to the form is placed in the API response).

3. The user chooses a way to pay for the invoice on the Payment Form and confirm payment. By default, the optimal payment method is showed.

4. If merchant [activates notification server](#notification-server), the merchant's service receives [notification](#notification) once the invoice is successfully paid by the user. Notifications are digitally signed, so you need to verify the signature on your [notification server](#notification-auth).

If required, via the API merchant can:

* [request current status](#invoice-status) of the invoice,
* [cancel invoice](#cancel) (if the user has not initiated payment yet).

## SDK and libraries {#sdk}

* [NODE JS SDK](https://github.com/QIWI-API/bill-payments-node-js-sdk) — Node JS package of ready-to-use solutions for server2server integration development.
* [PHP SDK](https://github.com/QIWI-API/bill-payments-php-sdk) — PHP package of ready-to-use solutions for server2server integration development.
* [Java SDK](https://github.com/QIWI-API/bill-payments-java-sdk) — Java package of ready-to-use solutions for server2server integration development.
* [.Net SDK](https://github.com/QIWI-API/bill-payments-dotnet-sdk) — C# .NET package of ready-to-use solutions for server2server integration development.

## CMS solutions {#cms}

* [Online Leyka](https://wordpress.org/plugins/leyka/) — Wordpress plugin for charity solutions
* [1С-Bitrix](http://marketplace.1c-bitrix.ru/solutions/qiwikassa.checkout/) — plugin for work with orders
* [Opencart](https://www.opencart.com/index.php?route=marketplace/extension/info&member_token=nH5fDsH3A5OkPF4zOe82hS0ypOhIqSEr&extension_id=36833) — plugin for work with orders
* [PrestaShop](https://github.com/QIWI-API/prestashop-payment-qiwi/releases) — plugin for work with orders

## Authorization methods {#auth}

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

To authorize API requests, you need the keys:

* Secret key `<SECRET_KEY>` — for request authorization in [P2P Invoicing API](#create) by [OAuth 2.0 specification](http://tools.ietf.org/html/rfc6750). Put the key into the HTTP-request header `Authorization: Bearer <SECRET_KEY>`.
* Public key `<PUBLIC_KEY>` — for authorization in [invoicing by the payment form call](#http-invoice).

To create a `<SECRET_KEY>` and `<PUBLIC_KEY>` pair of keys:

1. Login to the personal account on <https://p2p.qiwi.com/>.
2. Open **API** tab and click **Create a key pair and configure** button. When you create a key pair for the first time, click **Configure** button.

   ![p2p API Settings](/images/p2p-api/api-settings.png)
3. Enter a name for the key pair and click **Create** button.

   <img src="/images/p2p-api/create_p2p_keys_without_notifications.png" width="384">
4. Save the secret key in a safe place — it won't be displayed in your personal account interface. But you can always copy public key from your personal account.

   <img src="/images/p2p-api/key-pair.png" width="384">
5. Click on **Next** button. The key pair will be activated for use.

<aside class="warning">
Do not share secret key to third parties!

When a key is compromised, you need to re-issue them.
</aside>

You can use the secret key for QIWI Wallet payment operations:

* [make transfer to a wallet](/ru/qiwi-wallet-personal/#p2p);
* [make transfer to a card](/ru/qiwi-wallet-personal/#cards).

See details in the [documentation](/ru/qiwi-wallet-personal/#payments).

## Invoice Issue on Payment Form {#http}

<aside class="warning">
Only invoices in rubles can be created this way. For creating invoices in tenge use <a href="#create">API</a>, <a href="#sdk">SDK</a>, or <a href="#cms">CMS solutions</a>.
</aside>

<aside class="notice">
When opening Payment Form in Webview on Android, you should enable <code>settings.setDomStorageEnabled(true)</code>
</aside>

It is the simplest way of integration. On opening Payment Form, client receives an invoice at the same time. The invoice data sends in URL explicitly. Client gets a Payment Form web page with multiple payment means.

When using  this method, one cannot be sure that all invoices are issued by the merchant. [API invoice creation](#create) mitigates this risk.

~~~javascript
const publicKey = 'Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******';

const params = {
    publicKey,
    amount: 42.24,
    billId: 'cc961e8d-d4d6-4f02-b737-2297e51fb48e',
    successUrl: 'http://example.com/',
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
    successUrl: 'http://example.com/',
    email: 'mail@example.com'
};

const link = qiwiApi.createPaymentForm(params);
~~~

~~~shell
curl https://oplata.qiwi.com/create?publicKey=Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******&amount=100&successUrl=http%3A%2F%2Fexample.com%3Fsuccess%3F&email=mail@example.com&customFields[paySourcesFilter]=qw,card&lifetime=2020-12-01T0509
~~~

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Invoice data are put in Payment Form URL.</span></li>
</ul>

Parameter|Description|Type
---------|--------|---
publicKey | **Required**. Merchant's [public key for the payment form](#auth) obtained on p2p.qiwi.com|String
billId|Unique invoice identifier in merchant's system|URL-Encoded String(200)
amount| Amount of the invoice rounded down on two decimals | Number(6.2)
phone | Phone number of the client to which the invoice is issuing (international format) | URL-Encoded String
email | E-mail of the client where the invoice payment link will be sent | URL-Encoded String
account | Client identifier in merchant's system | URL-Encoded String
comment | Invoice commentary|URL-Encoded String(255)
customFields[]|Additional invoice data|URL-encoded String(255)
customFields[paySourcesFilter]|Allow only the specified payment methods for the client on Payment Form, if they are available to the merchant. Possible values: <br>`qw` - QIWI Wallet <br>`card` - card payment |URL-Encoded String(255)
customFields[themeCode]|[Personalization code](#custom) for Payments Form |String(255)
lifetime | Expiration date of the pay form link (invoice payment's due date). If the invoice is not paid after that date, the invoice assigns `EXPIRED` final status and it becomes void.<br> **Important! Invoice will be automatically expired when 45 days is passed after the invoicing date**|URL-Encoded String<br>`YYYY-MM-DDThhmm`
successUrl|The URL to which the client will be redirected in case of successful payment. URL must be within merchant's site.|URL-Encoded String

## P2P Invoices API. Creating an invoice {#create}

<aside class="warning">
Funds return is not supported for paid invoices.
</aside>

Only invoicing in ruble and tenge is supported.

API invoicing is the reliable method for integration. Parameters are sent by means of server2server requests with authorization.

Successful response contains `payUrl` URL to redirect client on Payment Form. See the [documentation](#payurl) for additional URL parameters supported.

<aside class="success">
Try <a href="/ru/p2p-sdk-guide/">SDK</a> for the integration.
</aside>

There is also more simple way to invoicing — with [direct calling payment form](#http-invoice).

<aside class="notice">
For testing your service, we recommend to create and pay invoices for 1 ruble.
</aside>

<h3 class="request method">Request → PUT</h3>

~~~javascript
const billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

const fields = {
    amount: 1.00,
    currency: 'RUB',
    comment: 'Hello world',
    expirationDateTime: '2018-03-02T08:44:07'
};

qiwiApi.createBill( billId, fields ).then( data => {
    //do smth with data
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
  'expirationDateTime' => '2018-03-02T08:44:07',
  'email' => 'mail@example.com',
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
                        "mail@example.com",
                        UUID.randomUUID().toString(),
                        "79123456789"
                ),
                "http://example.com/success"
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
        Email = "mail@example.com",
        Account = Guid.NewGuid().ToString(),
        Phone = "79123456789"
    },
    SuccessUrl = new Uri("http://example.com/success")
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

Body parameter|Description|Type
---------|--------|-----
billId|**Required**. Unique invoice identifier in merchant's system|String(200)
amount|**Required**. Invoice amount information|Object
amount.<br>currency| **Required**. Invoice amount currency code. `RUB` `KZT` | Alpha-3 ISO 4217 code
amount.<br>value| **Required**. Amount of the invoice rounded down to two decimals | Number(6.2)
expirationDateTime |  **Required**. Invoice due date. Time should be specified with time zone. If the date passed, the invoice is cancelled.|`YYYY-MM-DDThhmm+\-hh:mm`
customer | Customer data of the invoice subject|Object
customer.<br>phone | Phone number of the client to which the invoice is issuing (international format) |String
customer.<br>email | E-mail of the client where the invoice payment link will be sent |String
customer.<br>account | Client identifier in merchant's system |String
comment | Invoice commentary|String(255)
customFields|Additional invoice data|Object
customFields.<br>paySourcesFilter | Allow only these payment methods for the client on Payment Form. Possible values: <br>`qw` - QIWI Wallet <br>`card` - card payment |Comma separated string
customFields.<br>themeCode | [Personalization](#custom) for Payments Form |String(255)

<h3 class="request">Response ←</h3>

>Successful response body example

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

>Error response body

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

Field|Type|Description
--------|---|--------
billId|String|Unique invoice identifier in the merchant's system
siteId|String|Merchant's site identifier in p2p.qiwi
amount|Object|Information about the invoice amount
amount.<br>value|String|The invoice amount. The number is rounded down to two decimals
amount.<br>currency|String|Currency identifier of the invoice amount (Alpha-3 ISO 4217 code)
status|Object|Data of the invoice current status
status.<br>value|String | String representation of the status. [Possible statuses](#status)
status.<br>changedDateTime|String|Status refresh date. Date format:<br>`YYYY-MM-DDThh:mm:ss±hh`
customer|Object | Customer data of the invoice subject
customer.<br>phone|String | The customer’s phone (if specified in the invoice)
customer.<br>email|String|The customer's e-mail  (if specified in the invoice)
customer.<br>account| String|The customer's identifier in the merchant's system (if specified in the invoice)
customFields|Object|Additional invoice data provided by the merchant
comment|String|Comment to the invoice
creationDateTime|String|System date of the invoice creation. Date format:<br>`YYYY-MM-DDThh:mm:ss±hh`
payUrl|String|Pay form URL
expirationDateTime|String|Expiration date of the payment form URL (invoice payment's due date). [Redirect the customer](#payurl) to the URL link for invoice payment, or use [Popup JavaScript library](#popup), to open the form in the popup window. Date format:<br>`YYYY-MM-DDThh:mm:ss±hh`

## P2P Invoices API. Checking the Invoice Status {#invoice-status}

We recommend using the method after receiving the [payment notification](#notification).

<h3 class="request method">Request → GET</h3>

~~~javascript
const billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

qiwiApi.getBillInfo(billId).then( data => {
    //do smth ith data
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

>Error response body example

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

Field|Type|Description
--------|---|--------
billId|String|Unique invoice identifier in the merchant's system
siteId|String|Merchant's site identifier in p2p.qiwi
amount|Object|Information about the invoice amount
amount.<br>value|Number|The invoice amount. The number is rounded down to two decimals
amount.<br>currency|String|Currency identifier of the invoice amount (Alpha-3 ISO 4217 code)
status|Object|Invoice status data
status.<br>value|String|Current [invoice status](#status)
status.<br>changedDateTime|String|Status refresh date
customFields|Object|Additional invoice data provided by the merchant
customer |Object | Customer data of the invoice subject
customer.<br>phone|String | The customer’s phone (if specified in the invoice)
customer.<br>email|String|The customer's e-mail  (if specified in the invoice)
customer.<br>account|String|The customer's identifier in the merchant's system (if specified in the invoice)
comment|String|Comment to the invoice
creationDateTime|String|System date of the invoice creation. Date format:<br>`YYYY-MM-DDThh:mm:ss`
payUrl|String|Payment form URL for [customer redirect](#payurl)
expirationDateTime|String|Expiration date of the pay form link (invoice payment's due date). Date format:<br>`YYYY-MM-DDThh:mm:ss`

## P2P Invoices API. Cancelling the Invoice {#cancel}

Use this method to cancel unpaid invoice.

<h3 class="request method">Request → POST</h3>

~~~javascript
const bill_id = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

qiwiApi.cancelBill(billId).then( data => {
    //do smth with data
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
             <li>Content-Type: application/json</li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

>Successful response body example

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

>Error response body example

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

Field|Type|Description
--------|---|--------
billId|String|Unique invoice identifier in the merchant's system
siteId|String|Merchant's site identifier in p2p.qiwi
amount|Object|Information about the invoice amount
amount.<br>value|Number|The invoice amount. The number is rounded down to two decimals
amount.<br>currency|String|Currency identifier of the invoice amount (Alpha-3 ISO 4217 code)
status|Object|Invoice status data
status.<br>value|String|Current [invoice status](#status)
status.<br>changedDateTime|String|Status refresh date
customFields|Object|Additional invoice data provided by the merchant
customer |Object | Customer data of the invoice subject
customer.<br>phone|String | The customer’s phone (if specified in the invoice)
customer.<br>email|String|The customer's e-mail  (if specified in the invoice)
customer.<br>account|String|The customer's identifier in the merchant's system (if specified in the invoice)
comment|String|Comment to the invoice
creationDateTime|String|System date of the invoice creation. Date format:<br>`YYYY-MM-DDThh:mm:ss`
payUrl|String|Payment form URL
expirationDateTime|String|Expiration date of the pay form link (invoice payment's due date). Date format:<br>`YYYY-MM-DDThh:mm:ss`

## P2P Invoices API. Invoice Payment Statuses {#status}

Status|Description|Final
------|--------|---------
WAITING | Invoice issued awaiting for payment| -
PAID|Invoice paid|+
REJECTED|Invoice rejected by customer|+
EXPIRED|Invoice expired. Invoice not paid|+

## Invoice payment notifications {#notification}

<aside class="notice">
Notifications handler service on your side is not required for the integration. You can implement <a href="#invoice-status">invoice status polling</a> instead.
</aside>

Before working with the notification service, consider the [Notification API Integration Terms](https://qiwi.com/support/products/p2p/usloviya_integratsii_api_uvedomleniy).

Pools of IP-addresses from which QIWI service sends notifications:

* 79.142.16.0/20
* 195.189.100.0/22
* 91.232.230.0/23
* 91.213.51.0/24

If your web service works behinds the firewall, you need to add these IP-addresses to the list of allowed addresses for incoming TCP packets.

<aside class="warning">
Callback is sent by HTTPS protocol on 443 port only.

Certificate should be issued by any trusted center of certification (e.g. Comodo, Verisign, Thawte etc).
</aside>

Notification (callback) is an incoming HTTP POST-request.

<h3 class="request method">Request ← POST</h3>

>Notification example

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

The request's body contains JSON-serialized invoice data encoded by UTF-8 codepage.

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
Headers are case-insensitive according to the standard, and your client can change them — see the documentation of your client.
</aside>

<aside class="notice">
Any response with HTTP status code other than 200 (OK) will be treated as a temporary merchant's service error. QIWI server repeats the notification request with increasing period within the next 24 hours.
</aside>

<aside class="warning">
Notification might be sent more than once, even if your service responds with successful HTTP status code. Take it into consideration when developing application business logic on your side.
</aside>

## Merchant's server registration {#notification-server}

URL address of notification processing server is configured in the [personal account profile](https://p2p.qiwi.com/). A new [key pair](#auth) is issued simultaneously.

<aside class="warning">
You can configure the notification server for only one key pair.
</aside>

1. Login to your [personal account](https://p2p.qiwi.com/).
2. Open **API** tab and click **Create key pair and configure** button. When you configure the server address along with creating a key pair for the first time, click **Configure** button.

   ![p2p API Settings](/images/p2p-api/api-settings.png)
3. Enter name for the new key pair.

   <img src="/images/p2p-api/create_p2p_keys_without_notifications.png" width="384">
4. Select **Use this key pair for notifications about invoice status changes** field.

   <img src="/images/p2p-api/create_p2p_keys_with_notifications.png" width="384">
5. In **URL notification server** specify your notification service URL. **The service URL must be accessible from the Internet.**
6. Click **Create** button.

   <img src="/images/p2p-api/key-pair.png" width="384">
7. Change QIWI P2P keys in your application settings to the new one.

## How to verify notification authenticity {#notification-auth}

Upon receiving incoming notification you need to verify its digital signature. The notification signature is placed in `X-Api-Signature-SHA256` HTTP header. Signature mechanism uses HMAC algorithm integrity check with SHA256-hash function.

Signature verification algorithm is as follows:

1. Prepare a string of the following notification's parameters separated by `|`:

    `invoice_parameters = {amount.currency}|{amount.value}|{billId}|{siteId}|{status.value}`

    where `{*}` is the value of the parameter. All values should be treated as strings.

2. Apply HMAC-SHA256 function:

    `hash = HMAС(SHA256, invoice_parameters, <SECRET_KEY>)`

    where:

    * `<SECRET_KEY>` – [secret key](#auth) used for the invoice creation;
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

Notification field|Description|Type
---------|--------|---
bill|Invoice data|Object
billId|Invoice identifier in the merchant's system|String(200)
siteId|Merchant's site identifier in p2p.qiwi |String
amount|Information about the invoice amount|Object
amount.<br>value|The invoice amount. The number is rounded down to two decimals|Number(6.2)
amount.<br>currency|Currency identifier of the invoice amount (Alpha-3 ISO 4217 code)|String(3)
status|Invoice status data|Object
status.<br>value|Current [invoice status](#status)|String
status.<br>changedDateTime|Status refresh date. Date format:<br>`YYYY-MM-DDThh:mm:ssZ`|String
customFields|Additional invoice data provided by the merchant|Object
customer | Customer data of the invoice subject  (if specified in the invoice)|Object
customer.<br>phone | The customer’s phone (if specified in the invoice)|String
customer.<br>email|The customer's e-mail  (if specified in the invoice)|String
customer.<br>account|The customer's identifier in the merchant's system (if specified in the invoice)| String
comment|Comment to the invoice|String(255)
creationDateTime|System date of the invoice creation. Date format:<br>`YYYY-MM-DDThh:mm:ssZ`|String
payUrl|Payment form URL|String
expirationDateTime|Expiration date of the pay form link (invoice payment's due date). Date format:<br>`YYYY-MM-DDThh:mm:ssZ`|String
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

## Invoice payment form {#payurl}

<aside class="notice">
When opening Payment Form in Webview on Android, you should enable <code>settings.setDomStorageEnabled(true)</code>
</aside>

You can add parameters to URL from `payUrl` field of the response to [invoice request](#create):

> Invoice URL example

~~~shell
curl https://oplata.qiwi.com/form?invoiceUid=a8437e7e-dc48-44f7-9bdb-4d46ca8ef2e4&paySource=qw&successUrl=google.com
~~~

| Parameter | Description | Type |
|--------------|------------|-------------|
| paySource |Pre-selected payment method for the client on Payment Form. Possible values: <br>`qw` - QIWI Wallet<br>`card` - card payment<br>`mobile` - payment from phone balance<br> When specified method is inaccessible, the page automatically selects recommended method for the user.| String |
| successUrl | The URL on the merchant's site for redirection of the customer after the successful payment. | URL-Encoded String |

Add referal links for payments from your site. This confirms the site is real and will avoid [the wallet blocking problems](https://qiwi.com/support/products/p2p/kak_izbezhat_blokirovki_koshelka). **Payments from any page without [Refer](https://developer.mozilla.org/ru/docs/Web/HTTP/Headers/Referer) header will result in wallet block. See details in the [documentation](https://qiwi.com/support/products/p2p/referalnie_ssylki).**

> Referal link example

~~~php
<?php 
  header("Referrer-Policy: no-referrer-when-downgrade"); 
?>
~~~

To make sure the [full referer](https://web.dev/referrer-best-practices/) is tranferred for the new version of browsers, set `Referrer-Policy` in your server response with `no-referrer-when-downgrade` value. It can be done for all pages of your site or only for the page with payment form link.

## Personalization {#custom}

Personalization allows you to create a payment form with your style, customizable logo, background and color of the buttons.  

Example of a customized payment form:

![Customer form](/images/Custom.png)

To setup payment form appearance:

1. Login to your personal account on [p2p.qiwi.com](https://p2p.qiwi.com).
2. Open [Transfer acceptance form](https://qiwi.com/p2p-admin/transfers/link) tab. Your code style for `themeCode` parameter (you need it for API and SDK requests) is marked bold in grey block. Value `codeStyle` is different for different QIWI wallets.

   ![alt_text](/images/p2p-api/sdk-image1.png "Form settings")
3. Click **Tune** button and configure the form' parameters.
4. Click **Save** button.

 >Invoice Issue on Payment Form

~~~shell
curl https://oplata.qiwi.com/create?publicKey=Fnzr1yTebUiQaBLDnebLMMxL8nc6FF5zfmGQnypc*******&amount=100&billId=cc961e8d-d4d6-4f02-b737-2297e51fb48e&successUrl=http%3A%2F%2Fexample.com%3Fsuccess%3F&customFields%5BthemeCode%5D=codeStyle
~~~

 >Invoice Issue by API

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
   "customFields": {"themeCode":"codeStyle"}
 }'
~~~

~~~javascript
const billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';

const fields = {
    amount: 1.00,
    currency: 'RUB',
    comment: 'Hello world',
    customFields: {themeCode: 'codeStyle'},
    expirationDateTime: '2018-03-02T08:44:07+03:00'
};

qiwiApi.createBill( billId, fields ).then( data => {
    //do with data
});
~~~

~~~php
<?php

$billId = 'cc961e8d-d4d6-4f02-b737-2297e51fb48e';
$customFields = ['themeCode' => 'codeStyle'];
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
            ThemeCode = "codeStyle"
        }
    }
);
~~~

To apply the appearance to the payment form:

* Add `customFields` object with `themeCode` field where your code is specified in the body of the [invoice request](#create) or in the parameters of invoice creation method in the corresponding [SDK module](#sdk).

   For example, `"customFields": { "themeCode":"codeStyle"}`.
* Add `customFields` array element with `themeCode` field on [calling the payment form](#http-invoice) and put your code there.

   For example, `customFields[themeCode]=кодСтиля`.

## Checkout Popup library {#popup}

<button id="pop" class="button-popup">Demo popup</button>

The library methods open the payment form as a popup window over your site in browser.

[Download QIWI Checkout Popup](https://github.com/QIWI-API/qiwi-invoicing-popup)

Installation:

`<script src='https://oplata.qiwi.com/popup/v1.js'></script>`

The library has two methods:

* Open an existing invoice.
* Open your [personal payform](https://qiwi.com/p2p-admin/transfers/link).

### Open an existing invoice {#openpopup}

>Open an existing invoice

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

Call function  `QiwiCheckout.openInvoice`.

| Parameter | Description | Type |
|--------------|------------|-------------|
| payUrl | **Required**. Payment form URL| String

### Open personal payform {#openpopupcustom}

Call function  `QiwiCheckout.openPreorder`.

>Open your custom payform

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

| Parameter | Description | Type |
|---------|----------|----------|
| widgetAlias | **Required**. URL of your payform | String |
