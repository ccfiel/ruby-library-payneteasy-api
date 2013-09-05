# Payment form integration

## Общие положения

* В данной статье описывается исключительно работа с библиотекой. Полная информация о выполнении Payment form integration расположена в [статье в wiki PaynetEasy](http://wiki.payneteasy.com/index.php/PnE:Payment_Form_integration).
* Описание правил валидации можно найти в описании метода **[Validator.validate_by_rule()](../library-internals/02-validator.md#validate_by_rule)**.
* Описание работы с цепочками свойств можно найти в описании класса **[PropertyAccessor](../library-internals/03-property-accessor.md)**

## <a name="form"></a> Запросы "sale-form", "preauth-form", "transfer-form"

* **sale-form** - применяется для оплаты с помощью кредитной карты
* **preauth-form** - применяется для блокирования части средств кредитной карты клиента. После успешного завершения этого запроса для списания средств с карты клиента необходимо выполнить запрос **[capture](01-preauth-capture-transactions.md#capture)**
* **transfer-form** - применяется для перевода средств с одного счета на другой

При выполнении запросов информация о карте вводится на стороне PaynetEasy.

##### Обязательные параметры запроса

Поле запроса        |Цепочка свойств платежа            |Правило валидации
--------------------|-----------------------------------|-----------------
client_orderid      |payment.client_id                  |Validator::ID
order_desc          |payment.description                |Validator::LONG_STRING
amount              |payment.amount                     |Validator::AMOUNT
currency            |payment.currency                   |Validator::CURRENCY
address1            |payment.billing_address.first_line |Validator::MEDIUM_STRING
city                |payment.billing_address.city       |Validator::MEDIUM_STRING
zip_code            |payment.billing_address.zip_code   |Validator::ZIP_CODE
country             |payment.billing_address.country    |Validator::COUNTRY
phone               |payment.billing_address.phone      |Validator::PHONE
ipaddress           |payment.customer.ip_address        |Validator::IP
email               |payment.customer.email             |Validator::EMAIL
redirect_url        |query_config.redirect_url          |Validator::URL

##### Необязательные параметры запроса

Поле запроса        |Цепочка свойств платежа            |Правило валидации
--------------------|-----------------------------------|-----------------
first_name          |payment.customer.first_name        |Validator::MEDIUM_STRING
last_name           |payment.customer.last_name         |Validator::MEDIUM_STRING
ssn                 |payment.customer.ssn               |Validator::SSN
birthday            |payment.customer.birthday          |Validator::DATE
state               |payment.billing_address.state      |Validator::COUNTRY
cell_phone          |payment.billing_address.cell_phone |Validator::PHONE
site_url            |query_config.site_url              |Validator::URL
server_callback_url |query_config.callback_url          |Validator::URL

[Пример выполнения запроса sale-form](../../example/sale-form.rb)
[Пример выполнения запроса preauth-form](../../example/preauth-form.rb)
[Пример выполнения запроса transfer-form](../../example/transfer-form.rb)

## <a name="form-redirect"></a> Обработка результата платежа после возвращения клиента с платежной формы

При возвращении пользователя с платежной формы на сервис мерчанта будут переданы данные с результатом обработки платежа. Обработка этих данных описана в [базовом примере использования библиотеки](../00-basic-tutorial.md#stage_2).

## <a name="callback"></a> Обработка обратного вызова

После завершения обработки платежа на стороне PaynetEasy, данные с результатом обработки передаются в сервис мерчанта с помощью обратного вызова. Это необходимо, чтобы платеж был обработан сервисом мерчанта независимо от того, выполнил пользователь корректно возврат с шлюза PaynetEasy или нет. Обработка этих данных совпадает с обработкой данных для [sale-form, preauth-form или transfer-form](05-payment-form-integration.md) и описана в [базовом примере использования библиотеки](../00-basic-tutorial.md#stage_2).

[Подробнее о Merchant callbacks](06-merchant-callbacks.md)
