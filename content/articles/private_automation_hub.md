---
title: "Private Automation Hub"
date: 2022-03-06T12:57:53-08:00
draft: False
---
Steps to create a collection

```ansible-galaxy collection init <namespace>.<collection_name>```

This creates the following structure:

```bash
namespace
└── collection_name
    ├── README.md
    ├── docs
    ├── galaxy.yml
    ├── plugins
    │   └── README.md
    └── roles
```

Create roles, plugins, modules, etc in the collection. When ready to build the collection change to its base directory and edit the galaxy.yml if needed, then run:

```bash
ansible-galaxy collection build
```

This will create a ```namespace-collection_name-1.0.0.tar.gz``` file. Now you can log into the private automation hub (PAH) and create the namespace and upload your new collection.

If you want to publish from the CLI the namespace must already exists in PAH, the collection does not have to exist just the namespace. Once the namespace is created run the following command (using the example file above):

```bash
ansible-galaxy collection publish 
-c --server=https://<ip or hosname of PAH>/api/galaxy/ 
--api-key=<your api key for PAH> 
namespace-collection_name-1.0.0.tar.gz
```

**-c : ignores ssl certificate errors**

If you would rather use your ansible.cfg to handle passing the key you will need the following in that file.
_*ansible.cfg*_

If you try to push an existing version of a namespace.collection from the CLI or through the web UI you will get a duplicate key error and the push will not happen.

The new collection will now be in the approval section of PAH.

