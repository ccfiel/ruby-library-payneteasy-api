# Sale transactions

Список запросов сценария:
* [Запрос "sale"](#sale)
* [Запрос "status"](#status)

## Общие положения

* В данной статье описывается исключительно работа с библиотекой. Полная информация о выполнении Sale transactions расположена в [статье в wiki PaynetEasy](http://wiki.payneteasy.com/index.php/PnE:Sale_Transactions).
* Описание правил валидации можно найти в описании метода **[Validator.validate_by_rule()](../library-internals/02-validator.md#validate_by_rule)**.
* Описание работы с цепочками свойств можно найти в описании класса **[PropertyAccessor](../library-internals/03-property-accessor.md)**

## <a name="sale"></a> Запрос "sale"

Запрос применяется для оплаты с помощью кредитной карты. При этом информация о карте вводится на стороне сервиса мерчанта и передается в запросе к PaynetEasy. После выполнения данного запроса необходимо выполнить серию запросов "**status**" для обновления статуса платежа. Для этого сервис мерчанта может вывести самообновляющуюся страницу, каждая перезагрузка которой будет выполнять запрос "**status**".

[Пример самообновляющейся страницы](../../example/common/wait_page.erb)

##### Обязательные параметры запроса

Поле запроса        |Цепочка свойств платежа                |Правило валидации
--------------------|---------------------------------------|-----------------
client_orderid      |payment.client_id                      |Validator::ID
order_desc          |payment.description                    |Validator::LONG_STRING
amount              |payment.amount                         |Validator::AMOUNT
currency            |payment.currency                       |Validator::CURRENCY
address1            |payment.billing_address.first_line     |Validator::MEDIUM_STRING
city                |payment.billing_address.city           |Validator::MEDIUM_STRING
zip_code            |payment.billing_address.zip_code       |Validator::ZIP_CODE
country             |payment.billing_address.country        |Validator::COUNTRY
phone               |payment.billing_address.phone          |Validator::PHONE
ipaddress           |payment.customer.ip_address            |Validator::IP
email               |payment.customer.email                 |Validator::EMAIL
card_printed_name   |payment.credit_card.card_printed_name  |Validator::LONG_STRING
credit_card_number  |payment.credit_card.credit_card_number |Validator::CREDIT_CARD_NUMBER
expire_month        |payment.credit_card.expire_month       |Validator::MONTH
expire_year         |payment.credit_card.expire_year        |Validator::YEAR
cvv2                |payment.credit_card.cvv2               |Validator::CVV2
redirect_url        |query_config.redirect_url              |Validator::URL

##### Необязательные параметры запроса

Поле запроса        |Цепочка свойств платежа            |Правило валидации
--------------------|-----------------------------------|-----------------
first_name          |payment.customer.first_name        |Validator::MEDIUM_STRING
last_name           |payment.customer.last_name         |Validator::MEDIUM_STRING
ssn                 |payment.customer.ssn               |Validator::SSN
birthday            |payment.customer.birthday          |Validator::DATE
state               |payment.billing_address.state      |Validator::COUNTRY
cell_phone          |payment.billing_address.cell_phone |Validator::PHONE
destination         |payment.destination                |Validator::LONG_STRING
site_url            |query_config.site_url              |Validator::URL
server_callback_url |query_config.callback_url          |Validator::URL

[Пример выполнения запроса sale](../../example/sale.rb)

## <a name="status"></a> Запрос "status"

Запрос применяется для проверки статуса платежа. Обычно требуется серия таких запросов из-за того, что обработка платежа занимает некоторое время. В зависимости от типа авторизации клиента (необходима 3D-авторизация или нет) и статуса платежа обработка результата этого запроса может происходить несколькими путями.

##### Необходимо обновление платежа

В том случае, если статус платежа не изменился (значение поля **status** - **processing**) и нет необходимости в дополнительных шагах авторизации, то запустить проверку статуса еще раз.

##### Необходима 3D-аторизация

В ответе на запрос будет передано поле **html**, содержимое которого необходимо вывести на экран браузера клиента. Содержимое поля представляет собой форму, которая переадресует пользователя для выполнения 3D-авторизации.

##### Обработка платежа завершена

В ответе на запрос поле **status** содержит результат обработки платежа - **approved**, **filtered**, **declined**, **error**

##### Обязательные параметры запроса

Поле запроса        |Цепочка свойств платежа|Правило валидации
--------------------|-----------------------|-----------------
client_orderid      |payment.client_id      |Validator::ID
orderid             |payment.paynet_id      |Validator::ID
login               |query_config.login     |Validator::MEDIUM_STRING

[Пример выполнения запроса status](../../example/status.rb)

## <a name="3d-redirect"></a> Обработка результата платежа после 3D-авторизации

Если при обработке платежа выполнялась 3D-авторизация, то при возвращении пользователя с формы авторизации на сервис мерчанта будут переданы данные с результатом обработки платежа. Обработка этих данных совпадает с обработкой данных для [sale-form, preauth-form или transfer-form](05-payment-form-integration.md) и описана в [базовом примере использования библиотеки](../00-basic-tutorial.md#stage_2).

## <a name="callback"></a> Обработка обратного вызова

После завершения обработки платежа на стороне PaynetEasy, данные с результатом обработки передаются в сервис мерчанта с помощью обратного вызова. Это необходимо, чтобы платеж был обработан сервисом мерчанта независимо от того, выполнил пользователь корректно возврат с шлюза PaynetEasy или нет.
[Подробнее о Merchant callbacks](06-merchant-callbacks.md)
