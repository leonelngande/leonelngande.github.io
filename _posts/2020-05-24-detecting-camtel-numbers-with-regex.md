---
layout: post
hidden: false
title: Detecting CAMTEL Numbers with Regex
author: Leonel Elimpe
tags:
  - regex
  - PHP
---
Am currently working on a project that deals with phone number validation and parsing so started out using Google's [libphonenumber](https://github.com/google/libphonenumber) (the PHP [port](https://github.com/giggsey/libphonenumber-for-php)). It works really well for MTN Cameroon, Orange, and NEXTTEL phone numbers.

```php
<?php

namespace App\Services;

use libphonenumber\PhoneNumber;
use libphonenumber\PhoneNumberToCarrierMapper;
use libphonenumber\PhoneNumberUtil;

class LibPhoneNumberService
{

    /** @var PhoneNumberUtil $phoneNumberUtil */
    private $phoneNumberUtil;

    /** @var PhoneNumberToCarrierMapper $phoneNumberUtil */
    private $carrierMapper;

    public function __construct()
    {
        $this->phoneNumberUtil = PhoneNumberUtil::getInstance();
        $this->carrierMapper = PhoneNumberToCarrierMapper::getInstance();
    }

    /**
     * Gets the name of the carrier for the given phone number, in the language provided. As per
     * {@link #getNameForValidNumber(PhoneNumber, Locale)} but explicitly checks the validity of
     * the number passed in.
     *
     * @param PhoneNumber|string $number
     * @param string $languageCode Language code for which the description should be written
     * @return string a carrier name for the given phone number, or empty string if the number passed in is
     *     invalid
     * @throws \libphonenumber\NumberParseException
     */
    public function getCarrierForNumber($number, string $languageCode = 'en'): string
    {
        if (is_string($number)) {
            $number = $this->parse($number);
        }

        return $this->carrierMapper->getNameForNumber($number, $languageCode);
    }
}
```

This is a Laravel Framework project. Calling `getCarrierForNumber()` above returns `MTN Cameroon` for an MTN number, `Orange` for an Orange number, `NEXTTEL` for a Nexttel number, and `''` (an empty string) for a CAMTEL number.

This was a problem. I then found out CAMTEL only became a mobile operator [this year](https://www.businessincameroon.com/telecom/1203-10074-camtel-becomes-the-4th-leading-mobile-operator-in-cameroon) (2020), and the libphonenumber library is still to be updated. Searching further, I found out from [here](https://www.cirt.cm/en/node/17?language=en) and [here](http://www.camtel.cm/assistance-espace-client/) CAMTEL numbers look like this:

* 222 XX XX XX (fixed line CAMTEL numbers)
* 233 XX XX XX (fixed line CAMTEL numbers)
* 242 XX XX XX (CDMA CAMTEL numbers)
* 243 XX XX XX (CDMA CAMTEL numbers)
* 620 XX XX XX (mobile CAMTEL numbers, operational as of this year 2020)

To fix this, I discussed with my teammate [Diyen Momjang](https://diyenmomjang.info) who came up with this regex based solution: `/^(222|233|242|243|620)\d{6}$/`.

The regex reads as follows: match all instances that start with 222 or 233 or 242 or 243 or 620, and followed by 6 digits.

Given we'll by matching against a 9 digit numbers, we first have to pass the phone number string through libphonenumber's `PhoneNumberUtil` `parse()` method, which returns a `PhoneNumber` object from which we call the `getNationalNumber()` method. It returns the national number of the phone number, e.g `620XXXXXX`.

Here's what my service ended up looking like:

```php
<?php

namespace App\Services;

use libphonenumber\PhoneNumber;
use libphonenumber\PhoneNumberToCarrierMapper;
use libphonenumber\PhoneNumberUtil;

class PhoneNumberService
{

    /** @var PhoneNumberUtil $phoneNumberUtil */
    private $phoneNumberUtil;

    /** @var PhoneNumberToCarrierMapper $phoneNumberUtil */
    private $carrierMapper;

    public function __construct()
    {
        $this->phoneNumberUtil = PhoneNumberUtil::getInstance();
        $this->carrierMapper = PhoneNumberToCarrierMapper::getInstance();
    }

    /**
     * Gets the name of the carrier for the given phone number, in the language provided. As per
     * {@link #getNameForValidNumber(PhoneNumber, Locale)} but explicitly checks the validity of
     * the number passed in.
     *
     * @param PhoneNumber|string $number
     * @param string $languageCode Language code for which the description should be written
     * @return string a carrier name for the given phone number, or empty string if the number passed in is
     *     invalid
     * @throws \libphonenumber\NumberParseException
     */
    public function getCarrierForNumber($number, string $languageCode = 'en'): string
    {
        if (is_string($number)) {
            $number = $this->parse($number);
        }

        $carrier = $this->carrierMapper->getNameForNumber($number, $languageCode);

        // Adds detection for CAMTEL numbers
        if (empty($carrier) && $this->isCamtelNumber($number)) {

            $carrier = 'CAMTEL';

        }

        return $carrier;
    }

    /**
     * Uses regex to determine if a number belongs to the CAMTEL network.
     * Here's what CAMTEL numbers look like:
     *      222 XX XX XX (fixed line camtel numbers)
     *      233 XX XX XX (fixed line camtel numbers)
     *      242 XX XX XX (CDMA camtel numbers)
     *      243 XX XX XX (CDMA camtel numbers)
     *      620 XX XX XX (mobile camtel numbers)
     *
     * @see https://www.cirt.cm/en/node/17?language=en
     * @see http://www.camtel.cm/assistance-espace-client/
     *
     * @param PhoneNumber $number
     * @return bool
     */
    public function isCamtelNumber(PhoneNumber $number)
    {
        // The regex reads: match all instances that start with 222 | 233 | 242 | 243 | 620, and followed by 6 digits
        if (preg_match("/^(222|233|242|243|620)\d{6}$/", $number->getNationalNumber())) {
            return true;
        }

        return false;
    }

    /**
     * @param string $number
     * @param string $defaultRegion
     * @return PhoneNumber
     * @throws \libphonenumber\NumberParseException
     */
    public function parse(string $number, $defaultRegion = 'CM'): PhoneNumber {
        return $this->phoneNumberUtil->parse($number, $defaultRegion);
    }
}

```