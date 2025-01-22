# OWASP CRS - Nextcloud Rule Exclusions Plugin

![Integration tests](https://github.com/coreruleset/nextcloud-rule-exclusions-plugin/actions/workflows/integration.yml/badge.svg)

## Description

This plugin contains rule exclusions for [Nextcloud](https://nextcloud.com/), a productivity platform and file hosting service. These rule exclusions are designed to resolve common false positives and allow for easier integration with the OWASP CRS (CRS).

## Supported apps

Due to how feature rich Nextcloud is, not all apps in Nextcloud can be supported (Especially 3rd party apps). False positives that are due to unsupported apps within Nextcloud must be resolved by a [custom rule exclusion](https://coreruleset.org/docs/concepts/false_positives_tuning/).

Additional apps may be supported if there is enough demand from the community, but in general, support for apps made by 3rd parties (not Nextcloud) won't be added.

|             *App Name*              |      *Status*      |
|------------------------------------ |--------------------|
| Nextcloud Files                     |  Supported ✅     |
| Nextcloud Photos                    |  Supported ✅     |
| Nextcloud Calendar                  |  Supported ✅     |
| Nextcloud Contacts                  |  Supported ✅     |
| Nextcloud Teams (Formerly Circles)  |  Supported ✅     |
| Nextcloud Mail                      |  Supported ✅     |
| Nextcloud Deck                      |  Supported ✅     |
| Nextcloud Office                    |  Supported ✅     |
| Nextcloud Notes                     |  Supported ✅     |
| Nextcloud Text Editor               |  Supported ✅     |
| Nextcloud Recognize                 |  Supported ✅     |
| Nextcloud Cookbook                  |  Supported ✅     |
| Nextcloud Files HPB (Notify_Push)   |  Supported ✅     |
| Nextcloud Talk                      |  Not supported ❌ |
| Nextcloud Forms                     |  Not supported ❌ |
| Nextcloud Polls                     |  Not supported ❌ |
| Nextcloud Unspash                   |  Not supported ❌ |
| Nextcloud Collectives               |  Not supported ❌ |
| Nextcloud News                      |  Not supported ❌ |
| Nextcloud Maps                      |  Not supported ❌ |
| Cospend                             |  Not supported ❌ |
| Breeze Dark                         |  Not supported ❌ |
| OnlyOffice                          |  Not supported ❌ |
| Nextcloud Tables                    |  Not supported ❌ |
| Nextcloud Assistant                 |  Not supported ❌ |
| ownCloud Music                      |  Not supported ❌ |


## Installation

For full and up to date instructions for the different available plugin installation methods, refer to [How to Install a Plugin](https://coreruleset.org/docs/concepts/plugins/#how-to-install-a-plugin) in the official CRS documentation.

### Conditionally enable plugins for multi-application environments

For full and up to date instructions on how to conditionally enable/disable this plugin on a multisite environment, please refer to [Conditionally enable plugins for multi-application environments](https://coreruleset.org/docs/concepts/plugins/#conditionally-enable-plugins-for-multi-application-environments) in the official CRS documentation.

## Known Limitations

Due to some engine limitations, there are a few issues that can't be handled out of the box with this plugin (such as issues relating to uploading large file sizes).
Below you can find a list of known limitations along with workarounds for this issue.

### Increasing max upload size

Large uploads can be modified with SecRequestBodyLimit. Or they can be more controlled by using the following:

Apache with ModSecurity2:
```
SecRule REQUEST_FILENAME "@rx (?:/index\.php/apps/files/ajax/upload\.php|/remote\.php/dav/(?:bulk|files/|uploads/))" \
    "id:9508030,\
    phase:1,\
    t:none,\
    nolog,\
    ctl:requestBodyLimit=1073741824"
```

ctl:requestBodyLimit is not supported in libmodsecurity3, Nginx users can increase max upload size
by using the following:

```
location ~ (?:/index\.php/apps/files/ajax/upload\.php|/remote\.php/dav/(?:bulk|files/|uploads/)) { modsecurity_rules 'SecRequestBodyLimit 1073741824'; }
```

Apache libmodsecurity3 Example:
```
<LocationMatch "(?:/index\.php/apps/files/ajax/upload\.php|/remote\.php/dav/(?:bulk|files/|uploads/))">
    modsecurity_rules 'SecRequestBodyLimit 1073741824'
</LocationMatch>
```

### Increasing max request body size

The Nextcloud desktop client occasionally sends large request bodies not containing any uploaded files.
ModSecurity will block request bodies larger than 131KB, adjusting SecRequestBodyNoFilesLimit to 141KB works for all scenarios tested.

Nginx libmodsecurity3 Example:
```
location /remote.php/dav/files/ { modsecurity_rules 'SecRequestBodyNoFilesLimit 144384'; }
```

Apache modsecurity2 Example:
```
<location "/remote.php/dav/files/">
   SecRequestBodyNoFilesLimit 144384
</location>
```

Apache libmodsecurity3 Example:
```
<location "/remote.php/dav/files/">
   modsecurity_rules 'SecRequestBodyNoFilesLimit 144384'
</location>
```

### Nextcloud Server Crawler

The Nextcloud Server Crawler is used by Nextcloud for various functions of Nextcloud such as generating document previews with Collabora, and testing the functionality of Nextcloud (for example, when setting up the high performance backend for files).
Copy this rule and replace `your-server-ip` with your Nextcloud Server's IP address.
Your Nextcloud Server's IP address will either be your server's WAN IP, or if your server is behind a NAT firewall then it's either your server's private IP address or your default gateway's IP address (Depending on your NAT configuration).
```
# Allow Nextcloud Server Crawler to crawl Nextcloud
# Generating document previews with Collabora
# Sometimes the server crawler's user agent will be missing/empty or an accept header is missing
SecRule REMOTE_ADDR "@ipMatch your-server-ip" \
    "id:9508032,\
    phase:1,\
    pass,\
    t:none,\
    nolog,\
    ctl:ruleRemoveById=920300,\
    ctl:ruleRemoveById=920320,\
    ctl:ruleRemoveById=920330,\
    ctl:ruleRemoveTargetById=920120;FILES:data,\
    ctl:ruleRemoveTargetById=920121;FILES:data,\
    ctl:ruleRemoveTargetById=920120;FILES_NAMES,\
    ctl:ruleRemoveTargetById=920121;FILES_NAMES,\
    ctl:ruleRemoveTargetById=922130;MULTIPART_PART_HEADERS"
```

## Testing

After the plugin is enabled, Nextcloud should work without problems caused by CRS (for example, false positives while blocking requests). If problems still occur then please file a new issue on [GitHub](https://github.com/coreruleset/nextcloud-rule-exclusions-plugin). (Note that high paranoia level deployments may require additional tuning beyond this plugin.)

## License

Copyright (c) 2022-2025 OWASP CRS project. All rights reserved.

The OWASP CRS and its official plugins are distributed under Apache Software License (ASL) version 2. Please see the enclosed LICENSE file for full details.
