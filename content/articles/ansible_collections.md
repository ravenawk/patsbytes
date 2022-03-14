---
title: "Ansible Collections"
date: 2022-03-10T21:40:44-08:00
draft: false
---

Ansible content collections, collections for short, are the way to include content not included in ansible-core. Roles, playbooks, modules, and plugins can all be included in a single collection, allowing you to bundle together the content of similar use.
The form "namespace.collection.item" refers to an item in a collection. Where namespace is the overarching group that a collection is a part of, you can have different collections as part of the same namespace. 

```amazon.aws```  
```community.general```  
```cisco.ios```  

Installing collections
Installing in the CLI
A collection can be installed using the ansible-galaxy command.

```bash
ansible-galaxy collection install ansible.netcommon
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

