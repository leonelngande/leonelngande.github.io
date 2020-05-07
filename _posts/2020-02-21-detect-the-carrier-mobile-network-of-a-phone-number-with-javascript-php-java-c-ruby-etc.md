---
layout: post
hidden: false
title: >-
  Detect The Carrier/Mobile Network Of A Phone Number With Javascript, PHP,
  Java, C++, Ruby, etc
tags:
  - PHP
  - Javascript
author: Leonel Elimpe
---
For a long time I've struggled with this problem, determining the Mobile Network a given phone number belongs to. Today I found out Google's [libphonenumber](https://github.com/google/libphonenumber) library (or any of it's [third party ports for other languages](https://github.com/google/libphonenumber#third-party-ports)) have this functionality built in.

Specifically, the [PhoneNumberToCarrierMapper](https://github.com/google/libphonenumber#highlights-of-functionality) functionality. I am currently using this with the PHP port [here](https://github.com/giggsey/libphonenumber-for-php#mapping-phone-numbers-to-carrier).

While researching this today, I equally came across [this](https://www.itu.int/oth/T0202000024/en) [document](https://www.itu.int/dms_pub/itu-t/oth/02/02/T02020000240001PDFE.pdf) which describes the breakdown of phone numbers by Carrier in Cameroon (MTN, ORANGE, and NEXTTEL). You can equally check it out if you intend to write functionality for a more specific use case.
