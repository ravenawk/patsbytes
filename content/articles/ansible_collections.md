---
title: "Ansible Collections"
date: 2022-03-10T21:40:44-08:00
draft: false
---

Ansible content collections, collections for short, are the way to include content not included in ansible-core. You can download collections from  galaxy.ansible.com or cloud.redhat.com (with a subscription) and even create your own. Roles, playbooks, modules, and plugins can be included in a single collection, allowing you to bundle content with similar use cases together. The form used when referring to an item in a collection is "namespace.collection.item". Think of a namespace as the container that groups similar collections, a collection of collections. Let's take a look at some examples.

| Collection Name         | Description                          |
|-------------------------|--------------------------------------|
| ```amazon.aws```        | Managing aws                         |
| ```ansible.netcommon``` | General networking tasks             |
| ```ansible.posix```     | Posix compliant systems              |
| ```community.general``` | Content on a variety of subjects     |

### Installing collections

A collection can be installed using the ansible-galaxy command.

```bash
ansible-galaxy collection install ansible.general
```

The collection is installed to the first path in the COLLECTIONS_PATHS variable, by default this is ~/.
ansible/collections.
A collection can also be installed using a requirements.yml file.

```bash
ansible-galaxy collection install -r requirements.yml
```

The format in the requirements.yml file is:

```yaml
---
collections:
# With just the collection name
- namespace.collection
# With the collection name, version, and
source options
- name: namespace.other_collection 
  version: 'version range identifiers (default: ``*``)'
  source: 'The Galaxy URL to pull the collection from (default: ``--api-server`` from cmdline)'
```

For example:

---
collections:
- name: ansible.netcommon
version: 1.1.0
source: https://galaxy.ansible.com

As above, this will install it to the first path in the COLLECTIONS_PATHS variable.
Installing in tower
To use a collection in ansible tower put the requirements.yml file in the collections directory located in
the root of the project. As seen below:

project/
ansible.cfg
collections
requirements.yml
inventory
playbook.yml
roles
requirements.yml

Using collections
In writing plays and roles refer to a module or plugin from a collection by including the namespace.
collection before the module or plugin name:

---
- hosts: all
connection: ansible.netcommon.netconf
tasks:
- name: use lookup filter to provide xml
configuration
ansible.netcommon.netconf_config:
content: "{{ lookup('file', './config.
xml') }}"

To avoid a lot of typing the collections keyword can be used in the play. By doing this the namespace.
collection can then be removed from the front of the module name. The fully qualified collection
name (FQCN) for plugins and lookups will still need to be used:

---
- hosts: all
connection: ansible.netcommon.netconf
collections:
- ansible.netcommon
tasks:
- name: use lookup filter to provide xml
configuration
netconf_config:
content: "{{ lookup('file', './config.
xml')}}"

When using the collections keyword be careful of clashing namespaces (namespaces that have the
same base module name)
References
On collections in general
https://docs.ansible.com/ansible/latest/user_guide/collections_using.html
For developing collections
https://docs.ansible.com/ansible/latest/dev_guide/developing_collections.html

