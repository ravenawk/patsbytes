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

```none
$ ansible-galaxy collection list
usage: ansible-galaxy [-h] [--version] [-v] TYPE ...

Perform various Role and Collection related operations.

positional arguments:
  TYPE
    collection   Manage an Ansible Galaxy collection.
    role         Manage an Ansible Galaxy role.

optional arguments:
  --version      show program's version number, config file location, configured module search path, module location, executable location
                 and exit
  -h, --help     show this help message and exit
  -v, --verbose  verbose mode (-vvv for more, -vvvv to enable connection debugging)
ERROR! - None of the provided paths were usable. Please specify a valid path with --collections-path
```

Did we get an error? Not really. Ansible looks at all the paths in the COLLECTIONS_PATHS environmental variable, and if it doesn't find any, it gives the above message.
### Installing collections

A collection can be installed using the ansible-galaxy command.

```none
$ ansible-galaxy collection install community.general
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/community-general-4.6.1.tar.gz to /home/patsbytes/.ansible/tmp/ansible-local-465vxd99hl5/tmpdy0zd9qn/community-general-4.6.1-_rg2s6mk
Installing 'community.general:4.6.1' to '/home/patsbytes/.ansible/collections/ansible_collections/community/general'
community.general:4.6.1 was installed successfully
```

Above `community.general` is installed at `/home/patsbytes/.ansible/collections/`

*Some options you can pass to the install command*  

- -p "path" : Path to install the collection (overrides `COLLECTION_PATHS` see below)   
- -r "file" : Install collections based off a requirements file (see below) 

If you don't use the -p option the collection is installed to the first path listed in the `COLLECTIONS_PATHS` environmental variable. By default, this is `~/.ansible/collections`.
Let's take a look at that default value.

```none
$ ansible-config dump | grep COLLECTIONS_PATHS
COLLECTIONS_PATHS(default) = ['/home/patsbytes/.ansible/collections', '/usr/share/ansible/collections']
```

If you want to change this list it can be edited in the ansible.cfg under `[defaults]` using the `collections_paths` key and adding a colon separated list for example:

```none
[defaults]
collection_paths = ./collections:~/.ansible/collections:/usr/share/ansible/collections
```

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