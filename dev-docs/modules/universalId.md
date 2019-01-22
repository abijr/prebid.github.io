---
layout: page_v2
page_type: module
title: Module - Universal ID
description: Supports multiple cross-vendor user IDs
module_code : universalId
display_name : universalId
enable_download : true
sidebarType : 1

---

# Universal User ID Module
{:.no_toc}

* TOC
{:toc}

## Overview

The Universal User ID module supports multiple ways of establishing pseudonymous IDs for users, which is an important way of increasing the value of header bidding. Instead of having several exchanges sync IDs with dozens of demand sources, a publisher can choose to integrate with one of these ID schemes:

* **Unified ID** – a simple cross-vendor approach – it calls out to a URL that responds with that user’s ID in one or more ID spaces (e.g. TradeDesk). The result is stored in the user’s browser for future requests and is passed to bidder adapters to pass it through to SSPs and DSPs that support the ID scheme.
* **PubCommon ID** – an ID is generated on the user’s browser and stored for later use on this publisher’s domain.

## How It Works

1. The publisher builds Prebid.js with the optional Universal ID module
1. The page defines universal ID configuration in `pbjs.setConfig()`
1. When `setConfig()` is called, and if the user has consented to storing IDs locally, the module is invoked to call the URL if needed
   1. If the relevant local storage is present, the module doesn't call the URL and instead parses the scheme-dependent format, injecting the resulting ID into bidRequest.universalID.
1. An object containing one or more IDs (bidRequest.universalID) is made available to Prebid.js adapters and Prebid Server S2S adapters.

Note that universal IDs aren't needed in the mobile app world because device ID is available in those ad serving scenarios.

Also note that not all bidder adapters support all forms of user ID. See the tables below for a list of which bidders support which ID schemes.

{: .alert.alert-success :}
While the Unified ID approach is open to other cookie vendors, the
only one currently supporting Prebid.js is The Tradedesk. Prebid.org
welcomes other ID vendors - create a PR or email support@prebid.org.

## Universal User ID and GDPR

When paired with the `CookieConsent` module, privacy rules are enforced:

* The module checks the GDPR consent string
* If no consent string is available OR if the user has not consented to Purpose 1 (local storage):
  * Calls to an external user ID vendor are not made.
  * Nothing is stored to cookies or HTML5 local storage.

## Examples

1) Publisher supports the default Tradedesk-based Unified ID and first party domain cookie storage:

{% highlight javascript %}
pbjs.setConfig({
    usersync: {
        universalIds: [{
            name: "unifiedId",
            storage: {
                type: "cookie",  
                name: "pbjs-unifiedid",       // create a cookie with this name
                expires: 60                   // cookie can last for 60 days
            }
        }],
        syncDelay: 5000              // 5 seconds after the first bidRequest()
    }
});
{% endhighlight %}

2) Publisher has a custom partner ID with Tradedesk Unified ID:

{% highlight javascript %}
pbjs.setConfig({
    usersync: {
        universalIds: [{
            name: "unifiedId",
            params: {
                partner: "myTtlPid"
            },
            storage: {
                type: "cookie",  
                name: "pbjs-unifiedid",       // create a cookie with this name
                expires: 60                   // cookie can last for 60 days
            }
        }],
        syncDelay: 5000              // 5 seconds after the first bidRequest()
    }
});
{% endhighlight %}

3) Publisher supports UnifiedID with a non-Tradedesk vendor, HTML5 local storage, and wants to delay the auction up to 250ms to obtain the user ID:

{% highlight javascript %}
pbjs.setConfig({
    usersync: {
        universalIds: [{
            name: "unifiedId",
            params: {
                url: "URL_TO_UNIFIED_ID_SERVER"
            },
            storage: {
                type: "html5",
                name: "pbjs-unifiedid"    // set localstorage with this name
            },
            maxDelayToAuction: 250   // wait up to 250ms before starting auction
				     // implies syncDelay of 0
        }]
    }
});
{% endhighlight %}

4) Publisher has integrated with unifiedID on their own and wants to pass the unifiedID directly through to Prebid.js

{% highlight javascript %}
pbjs.setConfig({
    usersync: {
        universalIds: [{
            name: "unifiedId",
            value: {"tdid": "D6885E90-2A7A-4E0F-87CB-7734ED1B99A3", 
                     "appnexus_id": "1234"}
        }]
    }
});
{% endhighlight %}

5) Publisher supports PubCommonID and first party domain cookie storage

{% highlight javascript %}
pbjs.setConfig({
    usersync: {
        universalIds: [{
            name: "pubCommonId",
            storage: {
                type: "cookie",  
                name: "_pubCommonId",       // create a cookie with this name
                expires: 1825               // expires in 5 years
            }
        }]
    }
});
{% endhighlight %}

6) Publisher supports both unifiedID and PubCommonID and first party domain cookie storage

{% highlight javascript %}
pbjs.setConfig({
    usersync: {
        universalIds: [{
            name: "unifiedId",
            storage: {
                type: "cookie",  
                name: "pbjs-unifiedid"       // create a cookie with this name
            }
        },{
            name: "pubCommonId",
            storage: {
                type: "cookie",  
                name: "pbjs-pubCommonId"     // create a cookie with this name
            }
        }],
        syncDelay: 5000       // 5 seconds after the first bidRequest()
    }
});
{% endhighlight %}

## Configuration

By including this module, the following options become available in `setConfig()`,
all of them under the `usersync` object as attributes of the `universalIds` array
of sub-objects. See the examples above for specific use cases.

{: .table .table-bordered .table-striped }
| Param under usersync.universalIds[] | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| name | Required | String | May be: `"unifiedId"` or `"pubCommonId"` | `"unifiedId"` |
| params | Optional | Object | Details for unifiedId. Defaults to use the Tradedesk URL and a partner code of "prebid". | |
| params.partner | Optional | String | If specified for unifiedId, overrides the "prebid" default value. This is useful for Tradedesk attribution and reporting. | `"myTTlPid"` |
| params.url | Optional | String | If specified for unifiedId, overrides the default Tradedesk URL. | "https://unifiedid.org/somepath?args" |
| storage | Required (unless `value` is specified) | Object | The publisher must specify some kind of local storage in which to store the results of the call to get the user ID. This can be either cookie or HTML5 storage. | |
| storage.type | Required | String | Must be either `"cookie"` or `"html5"`. This is where the results of the user ID will be stored. | `"cookie"` |
| storage.name | Required | String | The name of the cookie or html5 local storage where the user ID will be stored. | `"_unifiedId"` |
| storage.expires | Optional | Integer | How long (in days) the user ID information will be stored. Default is 30 for unifiedId and 1825 for PubCommonID | `365` |
| value | Optional | Object | Used only if the page has a separate mechanism for storing the unified ID. The value is an object containing the values to be sent to the adapters. In this scenario, no URL is called and nothing is added to local storage | `{"tdid": "D6885E90-2A7A-4E0F-87CB-7734ED1B99A3"}` |
| maxDelayToAuction | Optional | Integer | The presence of this attribute will cause Prebid.js to delay the auction for specified number of milliseconds. | `300` |

## Adapters Supporting the Universal ID Module

{% assign bidder_pages = site.pages | where: "layout", "bidder" %}

<table class="pbTable">
<tr class="pbTr"><th class="pbTh">Bidder</th><th class="pbTh">IDs Supported</th></tr>
{% for item in bidder_pages %}
{% if item.universalIds != nil %}
<tr class="pbTr"><td class="pbTd">{{item.biddercode}}</td><td class="pbTd">{{item.universalIds}}</td></tr>
{% endif %}
{% endfor %}
</table>

## Implementation Details

For bidders that want to support one or more of these ID systems, and for publishers who want to understand their options, here are the specific details.

{: .table .table-bordered .table-striped }
| ID System Name | ID System Host | Prebid.js Attr | Prebid Server Attr | Notes |
| --- | --- | --- | --- | --- | --- |
| PubCommon ID | n/a | bidRequest.universalIds.pubcid | user.ext.tpid[].source="pubcid" | PubCommon is unique to each publisher domain. |
| Unified ID | Tradedesk | bidRequest.universalIds.ttid | user.ext.tpid[].source="tdid" | |

The OpenRTB request location of these IDs:
{% highlight bash %}
{
  "user": {
    "ext": {
      "tpid": [{
        "source": "tdid",
        "uid": "19cfaea8-a429-48fc-9537-8a19a8eb4f0c"
      },
      {
        "source": "pubcid",
        "uid": "29cfaea8-a429-48fc-9537-8a19a8eb4f0d"
      }]
    }
  }
}
{% endhighlight %}

## Further Reading

* [Prebid.js Usersync](/dev-docs/publisher-api-reference.html#setConfig-Configure-User-Syncing)
* [GDPR ConsentManagement Module](/dev-docs/modules/consentManagement.html)