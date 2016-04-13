Marketo Forms 2.0 Prefill Module
Author: Domenic Santangelo <dsantangelo@magento.com>, @entendu

Description
===========
    Uses Marketo's REST API to prefill Marketo forms that are implemented with the Forms 2.0 JS API. This module gives
    you the building blocks you need; however, you need to do some custom JS coding to take advantage of it. See the
    "setup" section below.

Rationale
=========
    There are some good Marketo modules for Drupal already (such as https://www.drupal.org/project/marketo_ma), but
    none I'm aware of that help with Forms 2.0 Prefilling. Importantly, the Marketo REST API has a 10k request/day limit
    by default; this module has aggressive caching and flood protection built in to minimize the number of API calls
    that are performed -- either with malice or without.

Configuration & Security
========================
    After installing and enabling the module as usual, head over to <your site>/admin/config/system/marketo_prefill and
    punch in your API details. Follow these instructions to set up your API user and service:
    http://developers.marketo.com/blog/quick-start-guide-for-marketo-rest-api/. The access token has a lifetime and thus
    is automatically managed for you.

    Responses from Marketo are cached for 5 minutes. This is useful if you have a couple forms on one page, or forms
    on many pages.

    Anti-automation (flood) protection is built in, but you must enable it on the configuration screen. Note that a
    flood event is only fired when the Marketo API is called -- cached responses do not count.

Setup
=====
    To actually take adventage of prefilling, you must configure the field names you wish to prefill at
    <your site>/admin/config/system/marketo_prefill, and also configure your JS call to make use of this module.
    Essentially the flow is like this:

    +----------------------------------------+
    | User has the Marketo cookie, _mkto_trk |
    +------------------+---------------------+
                       |
     +-----------------v---------------------+
     | Your JS calls the MktoForms2.loadForm |
     | method to instantiate the form        |
     +-----------------+---------------------+
                       |
      +----------------v-------------------+
      | In a callback, you make an ajax    |
      | request to this module's endpoint, |
      | POSTing the _mkto_trk cookie       |
      +----------------+-------------------+
                       |
     +-----------------v-------------------+
     | You take the JSON response from the |
     | endpoint and use form.vals() to     |
     | populate the form fields            |
     +-------------------------------------+

    So, your JS may look something like this:

    // Use Forms 2.0 to load the form. You should already be doing this, although you may not have a callback function
    // today.
    MktoForms2.loadForm("//app-xxx.marketo.com", "XXX-YYY-ZZZ", formID, function(form) {
        // Callback function

        // You need some jQuery plugin to handle cookies, or write your own function
        var mktoCookie = $.cookie('_mkto_trk');
        // "Drupal.settings.marketo_prefill_enabled" is controlled by the "enabled" checkbox in the module settings
        if (mktoCookie && (typeof(Drupal.settings.marketo_prefill_enabled) != "undefined") && Drupal.settings.marketo_prefill_enabled) {
          // Post to the Drupal API endpoint that talks to marketo:
          $.ajax({
              type: "POST",
              url: "/m/pf",
              data: { cv: mktoCookie }
          })
          .done(function(res) {
            // If prefill info exists, fill the form:
            if (Object.keys(res).length) {
              form.vals(res);
            }
          });
        };
    )}

Credits
=======
Authored by Domenic Santangelo for Magento. Visit https://magento.com.