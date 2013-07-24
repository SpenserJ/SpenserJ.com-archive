---
layout: post
title: "Adding iFrames to Blackbaud NetCommunity <= 6.15.70"
date: 2011-03-25 12:47
comments: true
categories: 
- Web
- Blackbaud NetCommunity
- TinyMCE
description: "Adding support for iFrames to Blackbaud NetCommunity's TinyMCE editor in versions 6.15.70 and below"
keywords: Blackbaud NetCommunity, TinyMCE, iFrames
---
Version 6.15 of Blackbaud NetCommunity consisted of many large changes to how BBNC works, one of which was the editor. In this update, Blackbaud moved from BBCuteEditor to TinyMCE (a great decision IMO, as TinyMCE is one of the most popular WYSIWYG editors). Sadly, patch 70 and below do not support iFrames, even if you have the iFrame set as a "Safe HTML Tag" in the settings. Blackbaud is working on a patch for this (patch 71 to be exact), however they do not have an ETA for it yet.

<!-- more -->

We have always had maps embedded in our website, to help visitors find us, and we recently decided to also embed a few YouTube videos. Due to the change in 6.15 however, we could no longer do so, and I went on a hunt to find the key to the iFrame kingdom. After using the find-in-file feature of Windows, and coming up short on anything, I wrote a quick little script to rewrite the editor initialization code, remove existing editors, and reinitialize them, with the iframe allowed. It wasn't the most elegant solution, however it worked for the interm, as I waited for Blackbaud to release P71. While implementing it however, I stumbled across the original initialization code, and instantly threw what I had written out.

Since it took a while for me to find this, I figure it may come in handy for anyone else that needs to add iFrames to TinyMCE in Blackbaud NetCommunity, or for anyone wanting to tweak the editor in the future.

The file that you'll want to edit is `{NetCommunity}/admin/HTMLEditorControl2.ascx`. For adding iFrames to the valid elements, I changed:

``` plain 
<appfx:HtmlEditorSetting Name="extended_valid_elements" Value="'style[dir&lt;ltr?rtl|lang|media|title|type]
```

to:

``` plain 
<appfx:HtmlEditorSetting Name="extended_valid_elements" Value="'iframe[src|width|height|name|align],style[dir&lt;ltr?rtl|lang|media|title|type]
```

Save the file, and refresh the editor, and you should now be able to insert iFrames into your page. Through this file, you can also change the theme that is used, disable certain toolbar buttons, and adjust anything else that is in the [TinyMCE Configuration](http://tinymce.moxiecode.com/wiki.php/Configuration).
