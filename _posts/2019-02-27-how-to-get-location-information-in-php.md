---
title: "How to get someones location information in PHP"
published: true
---

There have been many services available with which to retrieve a clients GeoLocation, the most common that I have seen being [FreeGeoIP](https://github.com/fiorix/freegeoip). Today I’d like to share how you can go about retrieving this information using PHP.

> It is worth noting that FreeGeoIP has been deprecated and [IPStack](https://ipstack.com) has taken it’s place. **Luckily, the library I will be using in this post supports both FreeGeoIP and IPStack**!

Firstly we need to make a decision. Do we want to use the deprecated FreeGeoIP binary, or IPStack’s API.

  * The FreeGeoIP binary will mean unlimited requests **for free**, however it has been deprecated which means support for it has dropped and development has stopped. So if you go with this decision, you likely won’t be able to use it for long.
  * Using IPStack will be the more common solution. Their free plan offers 10,000 requests per month. They also offer higher tier plans which you can view [here](https://ipstack.com/product).

## Using IPStack

IPStack will be the easiest solution for retrieving a clients Geo information.

To begin with, create an IPStack API key [here](https://ipstack.com/product). That’s it. You’re done. You can now begin to implement the PHP library.

## Using FreeGeoIP Binary

If you’ve chosen to use the FreeGeoIP binary, you will need to download it from [here](https://github.com/fiorix/freegeoip/releases). Version 3.4.1 is the last version that has been released at the time this document is being written, and will likely be the last version ever released as FreeGeoIP is now deprecated.

Once you’ve downloaded the FreeGeoIP binary, you will need to start it on your web server.

```shell
./freegeoip -http :8080
```

This will start the binary on port 8080, which you will later be configuring your PHP library to communicate with. **It would be a good idea to have this binary run as a service.**

> At this point, you can already start looking up location information by opening a web browser and going to: _[http://127.0.0.1:8080/json/YOUR_IP_ADDRESS](http://127.0.0.1:8080/json/YOUR_IP_ADDRESS.)_[.](http://127.0.0.1:8080/json/YOUR_IP_ADDRESS.)

You can now begin to implement the PHP library.

## Integrating the PHP Library

Integrating the PHP library is much easier than you’d expect. Firstly, you will need to include the composer package used to retrieve the location information. _(If you aren’t familiar with composer, see __[here_](https://getcomposer.org/doc/01-basic-usage.md)_.)_

```shell
composer require nafisc/ipstackgeo-php
```

You can find this PHP library [hosted on GitHub here](https://github.com/nathan-fiscaletti/ipstackgeo-php).

Next, you will need to create the object that PHP will use to communicate with the servers that tell us the clients location.

**If you are using IPStack**

```php        
use IPStack\PHP\GeoLookup;

$geoLookup = new GeoLookup(
    // API Key
    'acecac3893c90871c3',

    // Use HTTPS (Basic plan and up only, default false)
    false,

    // Timeout in seconds (defaults to 10 seconds)
    10
);
```

**If you are using the FreeGeoIP Binary**

```php 
use IPStack\PHP\Legacy\FreeGeoIp as GeoLookup;

$geoLookup = new GeoLookup(
    // Address hosting the legacy FreeGeoIP Binary
    '127.0.0.1',

    // Port that the binary is running on (defaults to 8080)
    8080,

    // Protocol to use (defaults to http)
    'http',

    // Timeout (defaults to 10 seconds)
    10
);
```

Next, you will need to write the code necessary to retrieve the client location.
    
```php
$location = $geoLookup->getClientLocation();

if ($location == null) {
    echo 'Failed to find location.'.PHP_EOL;
} else {
    // Convert the location to a standard PHP array.
    print_r($location->_asStdArray());


    // Any of these formats will work for
    // retrieving a property.
    echo $location->latitude . PHP_EOL;
    echo $location['longitude'] . PHP_EOL;
    echo $location->region_name() . PHP_EOL;
}
```

> You can also retrieve the location for any IP address or host-name using this library. See below for an example of this.

See [here](https://github.com/nathan-fiscaletti/ipstackgeo-php/blob/v1.3/src/IPStack/Location.php) for a list of available properties on the `$location` object.

---

A finished implementation of this library might look something like this

[See Gist](https://gist.github.com/nathan-fiscaletti/260491c0f8d7e80ee2bbc27f3a732072)