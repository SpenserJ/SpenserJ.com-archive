---
layout: post
title: "Prevent Server Version Leaks in MediaWiki's Special:Version"
date: 2013-07-19 16:34
comments: true
categories: 
- Web
- PHP
- MediaWiki
- Security
---
Setting up a knowledge base or wiki is a great way of keeping people (coworkers, clients, strangers, and anyone else you can think of) informed about certain topics, and MediaWiki is one of the most popular choices for this. It powers Wikipedia, the [worlds largest general reference work](https://en.wikipedia.org/wiki/Wikipedia), and thousands of other wikis on a variety of subjects.

While obscuring version information should never be your only form of security, it makes life slightly harder for anyone looking to vandalize or penetrate your webserver. By default, MediaWiki will tell anyone who will listen what software you're running ([example](https://www.mediawiki.org/wiki/Special:Version)), and there is no obvious way of disabling this. The solution? Let's dig into the code behind this page, and cut it off at the source.

Open up `includes/specials/SpecialVersion.php`, and locate the `execute` function (somewhere around line 50). You'll find a variable called `$text` that will contain all of the information to be displayed, and one of the functions it calls is `softwareInformation()`. If you comment out the entire line and save the file, you'll plug MediaWiki's version leak.

``` php includes/specials/SpecialVersion.php - SpecialVersion->execute
public function execute( $par ) {
  global $wgSpecialVersionShowHooks;
  $this->setHeaders();
  $this->outputHeader();
  $out = $this->getOutput();
  $out->allowClickjacking();

  $text =
    $this->getMediaWikiCredits() .
    // Commented out to stop the version leak
    // $this->softwareInformation() .
    $this->getEntryPointInfo() .
    $this->getExtensionCredits();
```
