---
layout: post
title: "Locking down Blackbaud NetCommunity content author's powers"
date: 2010-12-20 18:21
comments: true
categories: 
- Ambrose
- Web
- Javascript
- Blackbaud NetCommunity
---
At Ambrose University College, we run Blackbaud NetCommunity as our CMS, and I've encountered a few issues with permissions for content authors, and how the editor itself works. Over the past few weeks, in an attempt to improve the experience of both our visitors, and our content authors, I have added a few lines to our website's javascript file (hosted in a document part).

One issue we have run into is that Internet Explorer does not play nice with our CSS3 box shadows (using CSS3Pie), and will save the progressive enhancements that are loaded specifically for IE, as part of the page and/or part, for all browsers. This results in pages that have been edited using IE, showing two box shadows in CSS3 compliant browsers, and only one in IE. Since IE is to blame (and has issues with other parts of the editor), I decided to kill off support for using IE when in administration mode.

While I was working on my IE-proofing, one of our content authors accidentally clicked the "Template Designer" button, which resulted in her inserting a part on every page on the website. This is not the first time we have encountered this issue, and last time it happened, it took about two hours to fix, as changes were also made to the HTML of the template. Two lines of javascript later, and the Stylesheet editor and Template Designer buttons no longer appear. If I need to make changes, I can always go through the page finder, but changes are so infrequent that having a front-and-center button is just asking for trouble.

Incase you would like to implement similar "fixes" on your NetCommunity website, I have the code for 6.10 posted below :)

``` javascript NetCommunity 6.10
$(function() {
  var $adminMenu = $('#bbAdminMenuDiv', top.document);
  var ieDisabledMessage = 'In order to edit the Ambrose website, please use Firefox. IE support has been disabled due to compatibility issues.';
  if ($.browser.msie && $adminMenu.length > 0) {
    $('div, table, a', $adminMenu).remove();
    $adminMenu.append(ieDisabledMessage);
  }

  if (top.document.location.href.indexOf('/AdminPage.aspx') > -1) {
    $('#pagecntnt_AdminToolbar_btn2, #pagecntnt_AdminToolbar_btn3', top.document).remove();
    if ($.browser.msie) { $('#pagecntnt_divContentPane', top.document).html(ieDisabledMessage); }
  }
});
```

*As of December 22nd, we have upgraded to 6.15, and the administration menu and screens have changed. As such, I have rewritten the code, and it is also posted below (for 6.15)*

``` javascript NetCommunity 6.15
$(function() {
  var $adminMenu = $('#ctl05_bbBetaAdminMenuDiv', top.document);
  var ieDisabledMessage = 'In order to edit the Ambrose website, please use Firefox. IE support has been disabled due to compatibility issues.';
  if ($.browser.msie && $adminMenu.length > 0) {
    $('ul.bb_mainMenu li:gt(0), .bb_subMenu', $adminMenu).remove();
    $('li:first', $adminMenu).html('<h2 style="padding: 0px 12px;">' + ieDisabledMessage + '</h2>');
  }

  if (top.document.location.href.indexOf('ambrose.edu/cms/') > -1) {
    $('#pagecntnt_AdminToolbar_btn1lnk, #pagecntnt_AdminToolbar_btn2lnk', top.document).remove();
    if ($.browser.msie) { $('#divPage', top.document).css('color','#fff').html(ieDisabledMessage); }
  }
});
```
