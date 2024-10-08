# Tinkoff Payment API client

Клиент для проведения платежей [API Тинькофф Платежи](https://www.tinkoff.ru/kassa/develop/api/payments/).

## Установка

```sh
composer require tinkoffpay/tinkoffpay
```

## Реализованные API
#### Платежи
- [Создание платежа](https://www.tinkoff.ru/kassa/develop/api/payments/init-description/)
- [Отмена платежа](https://www.tinkoff.ru/kassa/develop/api/payments/cancel-description/)
- [Получение статуса платежа](https://www.tinkoff.ru/kassa/develop/api/payments/getstate-description/)
- [Получения статуса заказа](https://www.tinkoff.ru/kassa/develop/api/payments/checkorder-description/)
- [Повторить неуспешные нотификации](https://www.tinkoff.ru/kassa/develop/api/payments/resend-description/)
- [Подтверждение платежа](https://www.tinkoff.ru/kassa/develop/api/payments/confirm-description/)

#### Автоплатежи
- [Автоплатеж](https://www.tinkoff.ru/kassa/develop/api/autopayments/charge-description/)

## Примеры

### Создание клиента

```php
/** @var PaymentClientInterface $paymentClient */
$paymentClient = PaymentClient::create('<terminal_key>', '<password>');
```

### Создание платежа

[API создания платежа](https://www.tinkoff.ru/kassa/develop/api/payments/init-description/)

[Полный пример](examples/init.php)

```php
$payment = newPayment()
    ->amount(1000)
    ->orderId('1234')
    ->oneStep()
->build();

/** @var NewPaymentResultInterface $result */
$result = $paymentClient->init($payment);

if ($result->isSuccess()) {
    print 'PaymentId: ' . $result->getPaymentId() . "\n";
    print 'PaymentURL: ' . $result->getPaymentURL() . "\n";
} else {
    print 'Error: ' . $result->getMessage() . "\n";
}
```

### Создание платежа с чеком

[Полный пример](examples/init.php)

```php
$payment = newPayment()
    ->orderId('33101')
    ->oneStep()
    ->receipt(newReceipt()
        ->email('pavel.k.danilin@gmail.com')
        ->taxationOSN()
        ->addItem(newReceiptItem()
            ->name('Кружка')
            ->price(1000)
            ->quantity(1)
            ->taxNone()
            ->build())
        ->build())
    ->build();

/** @var NewPaymentResultInterface $result */
$result = $paymentClient->init($payment);

if ($result->isSuccess()) {
    print 'PaymentId: ' . $result->getPaymentId() . "\n";
    print 'PaymentURL: ' . $result->getPaymentURL() . "\n";
} else {
    print 'Error: ' . $result->getMessage() . "\n";
}
```

### Отмена платежа

[API отмена платежа](https://www.tinkoff.ru/kassa/develop/api/payments/cancel-description/)

[Полный пример](examples/cancel.php)

```php
/** @var CancelResultInterface $result */
$result = $paymentClient->cancel(1645861116);

if ($result->isSuccess()) {
    print 'PaymentId: ' . $result->getPaymentId() . ' [' . $result->getStatus() . "]\n";
} else {
    print 'Error: ' . $result->getMessage() . "\n";
}
```

### Получение текущего статуса платежа

[API получение текущего статуса платежа](https://www.tinkoff.ru/kassa/develop/api/payments/getstate-description/)

[Полный пример](examples/get_state.php)

```php
/** @var GetStateResultInterface $result */
$result = $paymentClient->getState(1645861116);

if ($result->isSuccess()) {
    print 'PaymentId: ' . $result->getPaymentId() . ' [' . $result->getStatus() . "]\n";
} else {
    print 'Error: ' . $result->getMessage() . "\n";
}
```

### Получение статуса заказа

[API получение статуса заказа](https://www.tinkoff.ru/kassa/develop/api/payments/checkorder-description/)

[Полный пример](examples/check_order.php)

```php
/** @var CheckOrderResultInterface $result */
$result = $paymentClient->checkOrder('333335556669');

if ($result->isSuccess()) {
    print 'OrderID: ' . $result->getOrderId() . "\n";
    print '-----' . "\n";
    foreach ($result->getPayments() as $payment) {
        print 'PaymentID: ' . $payment->getPaymentId() . "\n";
        print 'Status: ' . $payment->getStatus() . "\n";
        print 'Success: ' . $payment->getSuccess() . "\n";
        print 'RRN: ' . $payment->getRRN() . "\n";
        print '=====' . "\n";
    }
} else {
    print 'Error: ' . $result->getMessage() . "\n";
}
```

### Повторить неуспешные нотификации

[API повторной отправки неуспешных нотификаций](https://www.tinkoff.ru/kassa/develop/api/payments/resend-description/)

[Полный пример](examples/resend.php)

```php
/** @var ResendResultInterface $result */
$result = $paymentClient->resendNotifications();

if ($result->isSuccess()) {
    print 'Count: ' . $result->getCount() . "\n";
} else {
    print 'Error: ' . $result->getMessage() . "\n";
}
```


### Автоплатежи

#### Родительский платеж

Вызовите метод Init с параметрами Recurrent и CustomerKey
Переадресуйте покупателя на страницу оплаты и дождитесь совершения платежа
В нотификации AUTHORIZED или CONFIRMED получите и сохраните идентификатор RebillID

[API проведения автоплатежа](https://www.tinkoff.ru/kassa/develop/api/autopayments/charge-description/)

```php
$charge = new Charge();
$charge->setPaymentId(10063);
$charge->setRebillId(1234);
$result = $paymentClient->charge($charge);

if ($result->isSuccess()) {
    // Успех
} else {
    // Ошибка
}
```
