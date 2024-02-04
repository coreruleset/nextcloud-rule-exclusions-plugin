# OWASP ModSecurity Core Rule Set - Nextcloud Rule Exclusions Plugin

## Description

This plugin contains rule exclusions for [Nextcloud](https://nextcloud.com/), a productivity platform and file hosting service. These rule exclusions are designed to resolve common false positives and allow for easier integration with the OWASP ModSecurity Core Rule Set (CRS).

## Installation

For full and up to date instructions for the different available plugin installation methods, refer to [How to Install a Plugin](https://coreruleset.org/docs/concepts/plugins/#how-to-install-a-plugin) in the official CRS documentation.

## Increasing max upload size

Large uploads can be modified with SecRequestBodyLimit. Or they can be more controlled by using the following:

```
SecRule REQUEST_URI "@endsWith /index.php/apps/files/ajax/upload.php" \
    "id:9508610,\
    phase:1,\
    t:none,\
    nolog,\
    ctl:requestBodyLimit=1073741824"
```

ctl:requestBodyLimit is not supported in libmodsecurity3, Nginx users can increase max upload size
by using the following:

```
location /index.php/apps/files/ajax/upload.php { modsecurity_rules 'SecRequestBodyLimit 1073741824'; }
```

Apache libmodsecurity3 Example:
```
 <location "/index.php/apps/files/ajax/upload.php">
    modsecurity_rules 'SecRequestBodyLimit 1073741824'
</location>
```

## Relaxing file upload restrictions

To relax upload restrictions for only the php files that need it, you put something like this in crs-setup.conf:

```
SecRule REQUEST_FILENAME "@rx /(?:remote\.php|index\.php)/" \
    "id:9508600,\
    phase:2,\
    t:none,\
    nolog,\
    pass,\
    setvar:'tx.restricted_extensions=.bak/ .config/ .conf/'"
```

## Increasing max request body size

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

## Testing

After the plugin is enabled, Nextcloud should work without problems caused by CRS (for example, false positives while blocking requests). If problems still occur then please file a new issue on [GitHub](https://github.com/coreruleset/nextcloud-rule-exclusions-plugin). (Note that high paranoia level deployments may require additional tuning beyond this plugin.)

## License

Copyright (c) 2022-2024 OWASP ModSecurity Core Rule Set project. All rights reserved.

The OWASP ModSecurity Core Rule Set and its official plugins are distributed under Apache Software License (ASL) version 2. Please see the enclosed LICENSE file for full details.
