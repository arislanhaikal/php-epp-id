
php-epp-id
========

This is a modified files from [AfriCC](https://github.com/AfriCC/php-epp2)

**php-epp-id** is a High Level Extensible Provisioning Protocol (EPP) TCP/SSL client written in modern PHP.

Released under the GPLv3 License, feel free to contribute (fork, create
meaningful branchname, issue pull request with thus branchname)!



Requirements
------------

* PHP 5.5+
* php-ext-intl
* php-ext-openssl


Features
--------

* modern PHP standards
    * [PSR-1](http://www.php-fig.org/psr/psr-1/), [PSR-2](http://www.php-fig.org/psr/psr-2/) & [PSR-4](http://www.php-fig.org/psr/psr-4/)
    * notice and warning free (find them, and I'll fix it!)
* high-level usage (Plug & Play)
* simplified client (auto login/logout, auto inject clTRID)
* SSL (+local-cert)
* XPath like setter to simplify the creation of complex XML structures
* XML based responses for direct traversal via XPath
* [RFC 5730](http://tools.ietf.org/html/rfc5730), [RFC 5731](http://tools.ietf.org/html/rfc5731), [RFC 5732](http://tools.ietf.org/html/rfc5732), [RFC 5733](http://tools.ietf.org/html/rfc5733), [RFC 5734](http://tools.ietf.org/html/rfc5734) & [RFC 3915](http://tools.ietf.org/html/rfc3915)


Install
-------

Via Composer

```
$ composer require arislanhaikal/php-epp-id
```


Usage
-----

See the [examples](https://github.com/pandi-id/php-epp-id/tree/master/examples)
folder for a more or less complete usage reference.


### Basic Client Connection

this will automatically login on connect() and logout on close()

```php
<?php
require 'vendor/autoload.php';

use Pandi\EPP\Client as EPPClient;

$epp_client = new EPPClient([
    'host' => 'epptest.org',
    'username' => 'foo',
    'password' => 'bar',
    'services' => [
        'urn:ietf:params:xml:ns:domain-1.0',
        'urn:ietf:params:xml:ns:contact-1.0'
    ],
    'debug' => true,
]);

try {
    $greeting = $epp_client->connect();
} catch(Exception $e) {
    echo $e->getMessage() . PHP_EOL;
    unset($epp_client);
    exit(1);
}

$epp_client->close();
```


### Create Frame Objects

setXXX() indicates that value can only be set once, re-calling the method will
overwrite the previous value.

addXXX() indicates that multiple values can exist, re-calling the method will
add values.

```php
<?php
require 'vendor/autoload.php';

use Pandi\EPP\Frame\Command\Create\Host as CreateHost;

$frame = new CreateHost;
$frame->setHost('ns1.example.com');
$frame->setHost('ns2.example.com');
$frame->addAddr('8.8.8.8');
$frame->addAddr('8.8.4.4');
$frame->addAddr('2a00:1450:4009:809::1001');
echo $frame;

// or send frame to previously established connection
$epp_client->sendFrame($frame);
```


### Parse Response

You can either access nodes directly by passing through a xpath or use the data()
Method which will return an assoc array.

```php
use Pandi\EPP\Frame\Command\Check\Domain as DomainCheck;
use Pandi\EPP\Frame\Response;

$frame = new DomainCheck;
$frame->addDomain('example.org');
$frame->addDomain('example.net');
$frame->addDomain('example.com');

$response = $epp_client->request($frame);
if (!($response instanceof Response)) {
    echo 'response error' . PHP_EOL;
    unset($epp_client);
    exit(1);
}

echo $response->code() . PHP_EOL;
echo $response->message() . PHP_EOL;
echo $response->clientTransactionId() . PHP_EOL;
echo $response->serverTransactionId() . PHP_EOL;
$data = $response->data();
if (empty($data) || !is_array($data)) {
    echo 'empty response data' . PHP_EOL;
    unset($epp_client);
    exit(1);
}

foreach ($data['chkData']['cd'] as $cd) {
    printf('Domain: %s, available: %d' . PHP_EOL, $cd['name'], $cd['@name']['avail']);
}
```

Credits
-------

* [AfriCC](https://github.com/AfriCC/php-epp2)

License
-------

php-epp-id is released under the GPLv3 License. See the bundled
[LICENSE](https://github.com/Pandi/php-epp-id/blob/master/LICENSE) file for
details.

