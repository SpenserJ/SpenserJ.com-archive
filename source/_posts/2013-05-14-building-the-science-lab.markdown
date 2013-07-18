---
layout: post
title: "Building the Science Lab"
date: 2013-05-14 19:53
comments: true
alias: /posts/building-the-science-lab
categories: 
---
There have been a significant number of bugs with the [Alternative PHP Cache (APC)](http://pecl.php.net/package/APC) over the past year, causing me to pass over PHP updates for fear of segfaults, however that has finally come to an end. APC is still just as buggy, but [PHP 5.5](https://wiki.php.net/rfc/optimizerplus) will be bringing [Zend OPCache](http://pecl.php.net/package/ZendOpcache) to the core, so I've decided to transition early and configure the PECL package. As Zend OPCache does not come with a User Cache, I decided to set up Memcache on the [Ambrose University College](https://ambrose.edu/) Drupal website, and was seeing benchmarks of around 65 requests per second (up from 30-35 req/sec with APC only).

As [APCu](https://github.com/krakjoe/apcu) (userland caching) continues to branch from the crusty old APC branch, and the old APC Opcaching code is remove, I'll be running a few tests to see whether APCu can bring more to the table than Memcache can, as I have no need for the distributed functionality that Memcache provides.
