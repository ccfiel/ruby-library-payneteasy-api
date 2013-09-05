# Transfer transactions

Список запросов сценария:
* [Запрос "create-card-ref"](#create-card-ref)
* [Запрос "transfer-by-ref"](#transfer-by-ref)
* [Запрос "status"](#status)

## Общие положения

* В данной статье описывается исключительно работа с библиотекой. Полная информация о выполнении Transfer transactions расположена в [статье в wiki PaynetEasy](http://wiki.payneteasy.com/index.php/PnE:Transfer_Transactions).
* Описание правил валидации можно найти в описании метода **[Validator.validate_by_rule()](../library-internals/02-validator.md#validate_by_rule)**.
* Описание работы с цепочками свойств можно найти в описании класса **[PropertyAccessor](../library-internals/03-property-accessor.md)**

## <a name="create-card-ref"></a> Запрос "create-card-ref"

Запрос применяется для получения id для кредитной карты, сохраненной на стороне PaynetEasy. Этот id позволяет совершать повторные платежи без ввода данных кредитной карты как на стороне сервиса мерчанта так и на стороне PaynetEasy.
Перед выполнением этого запроса необходимо выполнить один из следующих сценариев для проверки данных, которые ввел клиент:
* [Sale Transactions](00-sale-transactions.md)
* [Preauth/Capture Transactions](01-preauth-capture-transactions.md)
* [Payment Form Integration](05-payment-form-integration.md)

##### Обязательные параметры запроса

Поле запроса        |Цепочка свойств платежа|Правило валидации
--------------------|-----------------------|-----------------
client_orderid      |payment.client_id      |Validator::ID
orderid             |payment.paynet_id      |Validator::ID
login               |query_config.login     |Validator::MEDIUM_STRING

[Пример выполнения запроса create-card-ref](../../example/create-card-ref.rb)

После выполнения данного запроса будет получен id сохраненной кредитной карты и создан объект **[RecurrentCard](../library-internals/00-payment-data.md#RecurrentCard)**. Получить доступ к **RecurrentCard** можно с помощью вызова `payment_transaction.payment.recurrent_card_from`, а к ее id с помощью вызова `payment_transaction.payment.recurrent_card_from.card_reference_id`

## <a name="transfer-by-ref"></a> Запрос "transfer-by-ref"

Запрос применяется для перевода определенной суммы с одного счета на другой.
Перед выполнением этого запроса необходимо как минимум один запрос [create-card-ref](#create-card-ref), для получения id сохраненной карты, на которую производится перевод средств. Если перевод выполняется между двумя картами клиентов, то необходимо выполнить два запроса [create-card-ref](#create-card-ref), для получения id сохраненных карт, между которымы переводятся средства.
После выполнения данного запроса необходимо выполнить серию запросов "**status**" для обновления статуса платежа. Для этого сервис мерчанта может вывести самообновляющуюся страницу, каждая перезагрузка которой будет выполнять запрос "**status**".

##### Обязательные параметры запроса

Поле запроса            |Цепочка свойств платежа            |Правило валидации
------------------------|-----------------------------------|-----------------
client_orderid          |payment.client_id                  |Validator::ID
amount                  |payment.amount                     |Validator::AMOUNT
currency                |payment.currency                   |Validator::CURRENCY
ipaddress               |payment.customer.ip_address        |Validator::IP
destination-card-ref-id |payment.recurrent_card_to.paynet_id|Validator::ID
login                   |query_config.login                 |Validator::MEDIUM_STRING

##### Необязательные параметры запроса

Поле запроса            |Цепочка свойств платежа              |Правило валидации
------------------------|-------------------------------------|-----------------
order_desc              |payment.description                  |Validator::LONG_STRING
source-card-ref-id      |payment.recurrent_card_from.paynet_id|Validator::ID
cvv2                    |payment.recurrent_card_from.cvv2     |Validator::CVV2
redirect_url            |query_config.redirect_url            |Validator::URL
server_callback_url     |query_config.callback_url            |Validator::URL

[Пример выполнения запроса transfer-by-ref](../../example/transfer-by-ref.rb)

## <a name="status"></a> Запрос "status"

Запрос применяется для проверки статуса платежа. Обычно требуется серия таких запросов из-за того, что обработка платежа занимает некоторое время. В зависимости от статуса платежа обработка результата этого запроса может происходить несколькими путями.

##### Необходимо обновление платежа

В том случае, если статус платежа не изменился (значение поля **status** - **processing**) и нет необходимости в дополнительных шагах авторизации, то запустить проверку статуса еще раз.

##### Обработка платежа завершена

В ответе на запрос поле **status** содержит результат обработки платежа - **approved**, **filtered**, **declined**, **error**

##### Обязательные параметры запроса

Поле запроса        |Цепочка свойств платежа|Правило валидации
--------------------|-----------------------|-----------------
client_orderid      |payment.client_id      |Validator::ID
orderid             |payment.paynet_id      |Validator::ID
login               |query_config.login     |Validator::MEDIUM_STRING

[Пример выполнения запроса status](../../example/status.rb)

## <a name="callback"></a> Обработка обратного вызова

После завершения обработки платежа на стороне PaynetEasy, данные с результатом обработки передаются в сервис мерчанта с помощью обратного вызова. Это необходимо, чтобы платеж был обработан сервисом мерчанта независимо от того, выполнил пользователь корректно возврат с шлюза PaynetEasy или нет.
[Подробнее о Merchant callbacks](06-merchant-callbacks.md)
