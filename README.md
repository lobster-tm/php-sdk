# UnitPay PHP-SDK
Php sdk for [UnitPay ](https://unitpay.ru) 
 
Documentation https://unitpay.ru/doc

## Examples ##
These are just some quick examples. Check out the samples in [`/sample`](https://github.com/unitpay/php-sdk/blob/master/sample).

### Payment integration using UnitPay Form
```php
<?php
include ('../UnitPay.php');

// Project Data
$secretKey  = '9e977d0c0e1bc8f5cc9775a8cc8744f1';
$publicId   = '15155-ae12d';

// My item Info
$itemName = 'Iphone 6 Skin Cover';

// My Order Data
$orderId        = 'a183f94-1434-1e44';
$orderSum       = 900;
$orderDesc      = 'Payment for item "'.$itemName.'"';
$orderCurrency  = 'RUB';

$unitPay = new UnitPay($secretKey);

$redirectUrl = $unitPay->form(
    $publicId,
    $orderSum,
    $orderId,
    $orderDesc,
    $orderCurrency
);

header("Location: " . $redirectUrl);
?>
```

### Payment integration using UnitPay Api

```php
<?php

header('Content-Type: text/html; charset=UTF-8');

/**
 * API integration
 *
 * @link https://unitpay.ru/doc#initPayment
 */

include ('../UnitPay.php');

// Project Data
$projectId  = 1;
$secretKey  = '9e977d0c0e1bc8f5cc9775a8cc8744f1';

// My item Info
$itemName = 'Iphone 6 Skin Cover';

// My Order Data
$orderId        = 'a183f94-1434-1e44';
$orderSum       = 900;
$orderDesc      = 'Payment for item "'.$itemName.'"';
$orderCurrency  = 'RUB';

$unitPay = new UnitPay($secretKey);

/**
 * Base params: account, desc, sum, currency, projectId, paymentType
 * Additional params:
 *  Qiwi, Mc:
 *      phone
 * alfaClick:
 *      clientId
 *
 * @link https://unitpay.ru/doc#initPayment
 * @link https://unitpay.ru/doc#paymentTypes
 */
$response = $unitPay->api('initPayment', [
    'account' => $orderId,
    'desc' => $orderDesc,
    'sum' => $orderSum,
    'paymentType' => 'yandex',
    'currency' => $orderCurrency,
    'projectId' => $projectId
]);

// If need user redirect on Payment Gate
if (isset($response->result->type)
    && $response->result->type == 'redirect') {
    // Url on PaymentGate
    $redirectUrl = $response->result->redirectUrl;
    // Payment ID in Unitpay (you can save it)
    $paymentId = $response->result->paymentId;
    // User redirect
    header("Location: " . $redirectUrl);

// If without redirect (invoice)
} elseif (isset($response->result->type)
    && $response->result->type == 'invoice') {
    // Url on receipt page in Unitpay
    $statusUrl = $response->result->statusUrl;
    // Payment ID in Unitpay (you can save it)
    $paymentId = $response->result->paymentId;
    // Invoice Id in Payment Gate (you can save it)
    $invoiceId = $response->result->invoiceId;
    // User redirect
    header("Location: " . $statusUrl);

// If error during api request
} elseif (isset($response->error->message)) {
    $error = $response->error->message;
    print 'Error: '.$error;
}
```

### Handler sample

```php
<?php

/**
 *  Demo handler for your projects
 *
 * @link https://unitpay.ru/doc#confirmPayment
 */
include ('../UnitPay.php');

// Project Data
$projectId  = 1;
$secretKey  = '9e977d0c0e1bc8f5cc9775a8cc8744f1';

// My item Info
$itemName = 'Iphone 6 Skin Cover';

// My Order Data
$orderId        = 'a183f94-1434-1e44';
$orderSum       = 900;
$orderDesc      = 'Payment for item "'.$itemName.'"';
$orderCurrency  = 'RUB';

$unitPay = new UnitPay($secretKey);

try {
    // Validate request (check ip address, signature and etc)
    $unitPay->checkHandlerRequest();

    list($method, $params) = array($_GET['method'], $_GET['params']);

    // Very important! Validate request with your order data, before complete order
    if (
        $params['orderSum'] != $orderSum ||
        $params['orderCurrency'] != $orderCurrency ||
        $params['account'] != $orderId ||
        $params['projectId'] != $projectId
    ) {
        // logging data and throw exception
        throw new InvalidArgumentException('Order validation Error!');
    }
    switch ($method) {
        // Just check order (check server status, check order in DB and etc)
        case 'check':
            print $unitPay->getSuccessHandlerResponse('Check Success. Ready to pay.');
            break;
        // Method Pay means that the money received
        case 'pay':
            // Please complete order
            print $unitPay->getSuccessHandlerResponse('Pay Success');
            break;
        // Method Error means that an error has occurred.
        case 'error':
            // Please log error text.
            print $unitPay->getSuccessHandlerResponse('Error logged');
            break;
        // Method Refund means that the money returned to the client
        case 'refund':
            // Please cancel the order
            print $unitPay->getSuccessHandlerResponse('Order canceled');
            break;
    }
// Oops! Something went wrong.
} catch (Exception $e) {
    print $unitPay->getErrorHandlerResponse($e->getMessage());
}
```

## Installation

### Install composer package
Set up `composer.json` in your project directory:
```
{
  "require":{"unitpay/php-sdk":"master"}
}
```

Run [composer](http://getcomposer.org/doc/00-intro.md#installation):
```sh
$ php composer.phar install
```

### Direct download

Download [last version ](https://github.com/unitpay/php-sdk/archive/master.zip) , unzip and copy to your project folder.

## Contributing ##

Please feel free to contribute to this project! Pull requests and feature requests welcome!