---
layout: page
title: Antelope Quickstart
permalink: /guidebook-quickstart
---

Antelope can be [installed]({% link guidebook/installation.md %}) directly from `pip` and operated either with cloud-based or local data.  This page gives instructions on how to:
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
Enter username to access blackbook server at https://sc.vault.lc/: satyr
Enter password to access blackbook server at https://sc.vault.lc/: 
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
 'lcia.openlca.2.7.5',
 'lcia.traci.2.1']
'''
```

### Obtain a resource

```pycon
>>> cat.get_blackbook_resources('lcacommons.uslci.fy24.q1.01')
'GET https://sc.vault.lc/origins/lcacommons.uslci.fy24.q1.01/token.. 200 [0.73 sec]'
>>> cat.show_interfaces()
'lcacommons.uslci.fy24.q1.01 [background, basic, exchange, index]'
'local.qdb [basic, index, quantity]'
>>> cat.query('lcacommons.uslci').get('0aaf1e13-5d80-37f9-b7bb-81a6b8965c71').name
' Petroleum refining, at refinery [United States] '
```

## Local Operation
Running locally means having local data files available for the software to use. This approach is necessary if the modeler wishes to make alterations to background allocation or linking, or if you want to make use of a resource that is not available on vault.lc.

### Accessing local files

To create a local resource, you need to identify the following items:
 - `origin` - the dotted reference string you will use to refer to it
 - `source` - the file path or URL that provides access to the data
 - `ds_type` - the Antelope [provider](/guidebook/providers) that understands how to interpret the data source
 - `interfaces` - the list of [interfaces](/guidebook/interfaces) the data source supports. 

> Note: Many useful providers, including `EcospoldV1Archive`, `EcospoldV2Archive`, and `IlcdArchive`,
> require the python `lxml` library to be installed (this is not installed by default)
{: .prompt-tip }

For this example, we will use the "FHWA Asphalt Framework" dataset from the LCA commons.  (see also 
[FHWA Part I]({ % link /posts/fhwa-part-I % }) for more details)

Note that this archive contains ambiguous links; therefore we must provide instructions about how to resolve these links when we generate the background matrix. We do this by providing the argument `multi_term` to `check_bg()` and specify it as `'cutoff'`.

First, obtain the OpenLCA v2 JSON-LD archive (.zip) from the Federal Commons website and save it somewhere on your computer (note its `/path/to/the-file.zip`). Then, start up python.

#### Recurring access (saved content)

If you want to access the material repeatedly over several sessions, you should create a permanent catalog to store the derived datasets (namely: an index and an ordered background).  To do this, you simply need to provide a *catalog root* to the `LcCatalog()` command-- this is a directory on your computer that will store the content. If the directory doesn't exist, it will be created.
]
In this approach, since we are saving our work, we will also create and save an index and an ordered background.
```python
from antelope_core import LcCatalog
cat = LcCatalog('/path/to/catalog')
cat.new_resource('fhwa', '/path/to/the-file.zip', 'OpenLcaJsonLdArchive', 
                 interfaces=('basic', 'exchange', 'quantity'))
cat.index_ref('fhwa')  # takes about 30 seconds to load and store all content
cat.background_for_origin('fhwa')  # creates a background interface
cat.query('fhwa').check_bg(multi_term='cutoff')  # takes about 10 seconds to create the ordered background
cat.show_interfaces()
''' Output:
fhwa [basic, exchange, quantity]
fhwa.index-20250904 [basic, index, background]
local.qdb [basic, index, quantity]
'''
q = cat.query('fhwa')
q.count('process')
1298
```


#### One-off access (no saved content)

If you want to access the database on a one-off basis, the process is a little simpler. We don't bother creating a permanent index, we only treat the data source as *static* (meaning we load it in its entirety 
on first access). It's not any faster, just a little less cluttered.
```python
from antelope_core import LcCatalog
cat = LcCatalog()
cat.new_resource('fhwa', '/path/to/the-file.zip', 'OpenLcaJsonLdArchive', interfaces=('basic', 'exchange', 'index', 'quantity'), static=True)
cat.background_for_origin('fhwa')  # takes about 30 seconds to load the archive
cat.query('fhwa').check_bg(multi_term='cutoff')  # takes about 10 seconds to create the ordered background
cat.show_interfaces()
'''
fhwa [background, basic, exchange, index, quantity]
local.qdb [basic, index, quantity]
'''
q = cat.query('fhwa')
q.count('process')
1298
```

## Combining cloud and local access
You can use both cloud and local access in the same session.  Here we can use the above local FHWA model with a cloud-based LCIA method to perform LCIA:

```python
cat.blackbook_guest('https://sc.vault.lc')
cat.get_blackbook_resources('lcia.openlca.2.7.5')
from antelope import enum  # this is a (perhaps poorly-named) utility for enumerating outputs
ts = enum(cat.query('lcia.openlca.2.7.5').lcia(method='TRACI'))
'''
 [00] [lcia.openlca.2.7.5] TRACI 2.2 [0]
 [01] [lcia.openlca.2.7.5] TRACI 2.1 [0]
'''
ts[0].show()
'''
QuantityRef catalog reference (752a90f0-5db5-4c02-8208-a8414dbb153a)
origin: lcia.openlca.2.7.5
   UUID: 752a90f0-5db5-4c02-8208-a8414dbb153a
   Name: TRACI 2.2
Comment: 
referenceUnit: 0
==Local Fields==
     Description: The Method is included in the openLCA LCIA methods package 2.7.5 and the impact directions are set. Databases from Nexus that are compatible with this method include ecoinvent 3.6, 3.7, 3.7.1, 3.8, 3.9.1, 3.10, 3.11 | Agribalyse 3.0, 3.01, 3.1 | Agrifootprint 5.0, 6.3 | OzLCI 2019.
ImpactCategories: ['448ee97a-bebc-467f-8b12-cdd8f11ca17a', '8e6ae039-3775-4c6c-91fa-ba830d3ea7a3', '34a8aabb-d140-439f-8813-b791e14c1a98', 'b8a0d65e-1879-4678-a5c0-1f49a6a117ec', '34f5feaa-1817-4a6d-8670-73353cdab45b', 'e7fe184a-2bc1-4975-906e-2e1f1b9ffae7', '47422855-d937-4f0d-b66e-b396ccd3db1f', '36853eb9-a379-4b40-8f47-48b210848c01', '8672839f-ee57-4578-9073-73bc025a02cb', '6deef80b-c723-4a0f-b70c-6738bb84be2d']
          Method: TRACI 2.2
  UnitConversion: {'0': 1.0}
        Synonyms: []
blackbook_origin: lcia.openlca.2.7.5
'''
tt = list(cat.query(ts[0].origin).get(k) for k in ts[0]['ImpactCategories'])
p = next(cat.query('fhwa').processes(name='asphalt mixture - framework'))
p.bg_lcia(tt[8]).show_details()
'''
completed 26 iterations
GET https://bk.vault.lc/lcia.openlca.2.7.5/8672839f-ee57-4578-9073-73bc025a02cb/factors.. 200 [0.81 sec]
Imported 113 factors for [lcia.openlca.2.7.5] TRACI 2.2, Eutrophication: freshwater [kg P eq] [TRACI 2.2]
[lcia.openlca.2.7.5] TRACI 2.2, Eutrophication: freshwater [kg P eq] [TRACI 2.2] kg P eq
------------------------------------------------------------

[fhwa] Asphalt mixture - Framework  [US]:
*  1.22e-06 =      0.953  x  -1.28e-06 [GLO] Phosphorus, ground-, long-term
   1.14e-07 =      0.953  x   1.19e-07 [GLO] Phosphorus, ground-, long-term
   1.09e-07 =      0.953  x   1.14e-07 [GLO] Phosphorus, ground-, long-term
   3.34e-09 =      0.953  x   3.51e-09 [GLO] Phosphorus, ground-, long-term
   1.59e-11 =      0.953  x   1.67e-11 [GLO] Phosphorus, ground-, long-term
   3.84e-13 =      0.953  x   4.04e-13 [GLO] Phosphorus, ground-, long-term
   3.33e-14 =      0.301  x   1.11e-13 [GLO] Phosphoric acid, to water
   1.87e-14 =      0.953  x   1.96e-14 [GLO] Phosphorus, ground-, long-term
   1.19e-27 =      0.301  x   3.96e-27 [GLO] Phosphoric acid, to water
* -2.93e-11 =      0.953  x   3.07e-11 [GLO] Phosphorus, ground-, long-term
  1.44e-06 [lcia.openlca.2.7.5] TRACI 2.2, Eutrophication: freshwater [kg P eq] [TRACI 2.2]

'''
```

[guidebook home](/guidebook)
