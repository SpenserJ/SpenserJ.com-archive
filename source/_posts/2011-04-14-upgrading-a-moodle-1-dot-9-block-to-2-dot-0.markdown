---
layout: post
title: "Upgrading a Moodle 1.9 Block to 2.0"
date: 2011-04-14 12:55
comments: true
categories: 
- Web
- Moodle
---
I recently went through the process of upgrading a Moodle 1.9 block for Panopto, to support Moodle 2.0, which came out in January. As Moodle 2.0 is a huge rewrite, and is still fairly fresh, there is very little documentation on what changes need to be made, or even what functions are deprecated and need replacing. In order to upgrade the plugin at Ambrose University College, I read through multiple core classes, and compared functions in 1.9 to 2.0. In order to make things easier for myself in the future, and for anyone else upgrading a block, I have documented my changes.

<!-- more -->

Language strings must be listed in `blockname/lang/en/block_blockname.php`, and one of the strings must be the plugin name:

``` php blockname/lang/en/block_blockname.php
<?php
$string['pluginname'] = 'Epic Block';
$string['name_of_string'] = 'English version of string';
```

The version has been moved from `blockname/blocks_blockname.php` to `blockname/version.php`. The format of the version remains the same (yyyymmdd00, where 00 can increment if you release multiple patches on a single day)

``` php blockname/version.php
<?php
$plugin->version = 2011040800;
```

The function for checking if the current user is a teacher who can edit the current course has changed, as have a few others, so use this as a base, and read through the core classes for the correct values.

``` php 
if(isteacheredit($COURSE->id)) {
```

has now become

``` php 
$context = get_context_instance(CONTEXT_COURSE, $COURSE->id);
if(has_capability('moodle/course:update', $context)) {
```

Stylesheets are now in `style.css`, instead of `style.php`.

Configuration files have also changed, as `config_instance.html` has been replaced with `edit_form.php` (below), and `config_global.html` has been replaced with `settings.php` (below `edit_form.php`).

``` php edit_form.php
<?php
require_once("lib/panopto_data.php");

class block_panopto_edit_form extends block_edit_form {
  protected function specific_definition($mform) {
    global $COURSE, $CFG;
    
    // Construct the Panopto data proxy object
    $panopto_data = new panopto_data($COURSE->id);
    
    if(!empty($panopto_data->servername) && !empty($panopto_data->instancename) && !empty($panopto_data->applicationkey))
    {
      $mform->addElement('header', 'configheader', 'Select the Panopto CourseCast course to display in this block.');
  
      $params->course_ids = $COURSE->id;
      $params->return_url = urlencode($_SERVER['REQUEST_URI']);
      $query_string = http_build_query($params);
      $provision_url = "$CFG->wwwroot/blocks/panopto/provision_course.php?" . $query_string;
      $course_list = $panopto_data->get_course_options($provision_url);
      $mform->addElement('selectgroups', 'config_course', 'Course', $course_list['courses']);
      $mform->setDefault('config_course', $course_list['selected']);
    } else {
      $mform->addElement('static', 'error', '', 'Cannot configure block instance: Global configuration incomplete. Please contact your system administrator.');
    }
  }
}
```

``` php settings.php
<?php

defined('MOODLE_INTERNAL') || die;

if ($ADMIN->fulltree) {
    $settings->add(new admin_setting_configtext('block_panopto_instance_name', 'Moodle Instance Name',
                       'This value is prefixed before usernames and course-names in Panopto.', 'moodle', PARAM_TEXT));
    
    $settings->add(new admin_setting_configtext('block_panopto_server_name', 'Panopto Server Hostname', '', '', PARAM_TEXT));

    $settings->add(new admin_setting_configtext('block_panopto_application_key', 'Application Key', '', '', PARAM_TEXT));

    $link ='<a href="'.$CFG->wwwroot.'/blocks/panopto/provision_course.php">Add Moodle courses to Panopto CourseCast</a>';
    $settings->add(new admin_setting_heading('block_panopto_add_courses', '', $link));
}
```

There are many more changes that occurred, and this post is only the tip of the iceberg, so post a comment if you can't find the replacement for a deprecated function, and I'll do my best to help you out. If you want to see a legitimate commit for the changes that I made to upgrade a 1.9 block to a working 2.0 block, take a look at [this commit](https://github.com/SpenserJ/Moodle2-Panopto/commit/ca893577b7d34adbeca6e95f5fe2f54194d488a3).
