---
title: "Ansible Collections"
date: 2022-03-10T21:40:44-08:00
draft: false
---

Ansible content collections, collections for short, are the way to include content not included in ansible-core. You can download collections from  https://galaxy.ansible.com or https://console.redhat.com (with a subscription) and even create your own.  

Roles, playbooks, modules, and plugins can be included in a single collection, allowing you to bundle content with similar use cases together. The form used when referring to an item in a collection is "namespace.collection.item". Think of a namespace as the container that groups similar collections, a collection of collections. Let's take a look at some examples.

| Collection Name         | Description                          |
|-------------------------|--------------------------------------|
| ```amazon.aws```        | Managing aws                         |
| ```ansible.netcommon``` | General networking tasks             |
| ```ansible.posix```     | Posix compliant systems              |
| ```community.general``` | Content on a variety of subjects     |

First let's see if we have any collections on our system.
![Ansible version and collection list](/images/version_collections_list.png)

The `ansible --version` shows **ansible collection location = /home/pbytes/.ansible/collections:/usr/share/ansible/collections**, we will come back to how to control that in a bit. For now realize this is the default and is what the *COLLECTIONS_PATHS* environmental variable contains.
The `ansible-galaxy collection list` found no collections in the above paths, so the command returns no output. 

### Installing collections

A collection can be installed by using the ansible-galaxy command.

![Install a collection](/images/install_collection.png)

Here we install `community.general` in `/home/pbytes/.ansible/collections/`

*Some options you can pass to the install command*  

- -p "path" : Path to install the collection (overrides *COLLECTION_PATHS*)   
- -r "file" : Install collections based off a requirements file (see below) 

If you don't use the -p option the collection is installed to the first path listed in the *COLLECTIONS_PATHS* environmental variable. By default, this is `~/.ansible/collections`.  
We have mentioned the *COLLECTIONS_PATHS* variable a few times now. Let's take a look at another way to view this variable and it's default value.

![COLLECTIONS_PATHS variable](/images/collections_paths_var.png)

If we want to change this variable, we need to add/edit the collections_paths key in the ansible.cfg under [defaults]. The format is a colon-separated list that overwrites the current value so we have to make sure to add back in any paths we still want ansible to search for collections. As an example, let's add a path for a collections directory in our current working directory:

```ini
[defaults]
collection_paths = ./collections:~/.ansible/collections:/usr/share/ansible/collections
```

Let's look at the new paths:

![COLLECTIONS_PATHS new variable](/images/new_collections_paths_var.png)

Using a requirements.yml file, you can also install a collection or list of collections.

The format of the requirements.yml file is:

```yaml
---
collections:
# With just the collection name
- namespace.collection

# With the collection name, version, and source options
- name: ansible.netcommon
  # Version range identifiers (default: ``*``)'
  version: 2.6.1
  # Source - The Galaxy URL to pull the collection from
  # (default: ``--api-server`` from cmdline)
  # Not normally needed for collections on galaxy.ansible.com, 
  # this would work without this line but inserted here for completeness.
  source: https://galaxy.ansible.com
```

For example:

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
Ansible config settings
https://docs.ansible.com/ansible/latest/reference_appendices/config.html