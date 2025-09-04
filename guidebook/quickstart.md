---
layout: page
title: Antelope Quickstart
permalink: /guidebook/quickstart
---

# Quick Start Guide
Antelope can be [installed]({% link installation.md %}) directly from `pip` and operated either with cloud-based or local data.  This page gives instructions on how to:
 - Start up a local catalog
 - obtain resources from the cloud
 - obtain resources from local files
 - use the query framework to get life cycle data

## Cloud-Based Operation

Cloud-based operation is supported by [vault.lc](https://vault.lc/). The quickest way to get started is to use the Guest service, which does not even require authentication.  The guest service is limited to a fixed number of queries per originating IP address per hour.

```python
# either antelope_core.LcCatalog or antelope_foreground.ForegroundCatalog may be used
from antelope_foreground import ForegroundCatalog
cat = ForegroundCatalog()

cat.blackbook_guest('https://sc.vault.lc/')
```

If/once you create your own account, you will need to determine your API password from the `vault.lc` Settings page by clicking the "Reveal API key" button.

![API Credentials are assigned by vault.lc](/assets/img/vault-credentials.png)

Use the username and API key reported there to authenticate to the server. You can also store the username and password as environment variables `BLACKBOOK_USERNAME` and `BLACKBOOK_PASSWORD` (antelope_core version 0.3.7 or higher).

```python
>>> cat.blackbook_authenticate('https://sc.vault.lc/')
'''
Enter username to access blackbook server at https://sc.vault.lc/: satyr' 
Enter password to access blackbook server at https://sc.vault.lc/: '
POST https://sc.vault.lc/auth/token.. 200 [0.63 sec]
Welcome back to blackbook, hosted by ANTELOPE_AUTHORITY.
Username: satyr, email: bkuczenski@bren.ucsb.edu
Last login: Sun Sep 22 23:21:13 2024 
token expires in 36000 s
''' 
```

### List available resources

The set of origins available to you is determined by the access grants that have been issued to you on the 
blackbook server.

```python
>>> list(cat.blackbook_origins)
'''
GET https://sc.vault.lc/origins.. 200 [0.13 sec]
['lcacommons.fhwa.asphaltframework',
 'lcacommons.useeio.2.0.1',
 'lcacommons.uslci.fy21.q1',
 'lcacommons.uslci.fy22.q3',
 'lcacommons.uslci.fy22.q4.01',
 'lcacommons.uslci.fy23.q4',
 'lcacommons.uslci.fy24.q1.01',
 'lcia.openlca.2.1.4',
 'lcia.traci.2.1']
'''
```

### Obtain a resource

```python
>>> cat.get_blackbook_resources('lcacommons.uslci.fy24.q1.01')
'''GET https://sc.vault.lc/origins/lcacommons.uslci.fy24.q1.01/token.. 200 [0.73 sec]'''
>>> cat.show_interfaces()
"""
lcacommons.uslci.fy24.q1.01 [background, basic, exchange, index]
local.qdb [basic, index, quantity]"""
>>> cat.query('lcacommons.uslci').get('0aaf1e13-5d80-37f9-b7bb-81a6b8965c71').name
''' Petroleum refining, at refinery [United States] '''
```
