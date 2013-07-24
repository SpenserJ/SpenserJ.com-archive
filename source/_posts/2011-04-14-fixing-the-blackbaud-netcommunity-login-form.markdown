---
layout: post
title: "Fixing the Blackbaud NetCommunity Login Form"
date: 2011-04-14 12:52
comments: true
categories: 
- Web
- Javascript
- Blackbaud NetCommunity
description: "Blackbaud NetCommunity's login form fails to handle the enter key properly, and will reload the page without submitting, but we can fix it with less than 10 lines of JS"
keywords: Blackbaud NetCommunity, Javascript, Form
---
Blackbaud NetCommunity 6.15 has a bug in the login form, where tabbing onto the "Remember Me" field and pressing enter (normal functionality to submit a form) will take you back to the login form, with your username filled in, but no password or error. This is due to the fact that javascript is being (poorly) added directly to the HTML tags, instead of writing a simple, unobtrusive javascript function, that targets all fields on the form.

To remedy this, add a JS file to any layout that may contain a login field, and add this code:

``` javascript 
$(function() {
  if (($loginForm = $('.LoginFormTable')).length &gt; 0) {
    $('input[type!=submit]', $loginForm).keypress(function(e) {
      if ((e.which &amp;&amp; e.which == 13) || (e.keyCode &amp;&amp; event.ke == 13)) {
        $('input[type=submit]', $loginForm).click();
        return false;
      } else { return true; }
    });
  }
});
```
