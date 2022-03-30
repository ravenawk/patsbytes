---
title: "Ansible Collections"
date: 2022-03-16
draft: false
---

Ansible content collections, collections for short, are the way to include content not included in ansible-core. You can download collections from  https://galaxy.ansible.com or https://console.redhat.com (with a subscription) and even create your own.

Roles, playbooks, modules, and plugins can be included in a single collection, allowing you to bundle content with similar use cases together. When referring to a collection, use “namespace.collection”. Let’s look at some examples.

| Collection Name         | Description                          |
|-------------------------|--------------------------------------|
| ```amazon.aws```        | Managing aws                         |
| ```ansible.netcommon``` | General networking tools             |
| ```ansible.posix```     | Posix compliant systems              |
| ```community.general``` | Content on a variety of subjects     |

### A variable that is useful to know

Before we dive into installing and using collections, let's detour for a moment and look at the COLLECTIONS_PATHS environmental variable. 

_COLLECTIONS_PATHS_ is a list of paths that ansible uses to locate installed collections on the system, and when installing a collection, ansible uses the first path in this list. 

Let's look at a couple of ways to see this variable's value.

```bash
pbytes@patsbyes:~$ ansible-config dump | grep COLLECTIONS_PATHS
COLLECTIONS_PATHS(default) = ['/home/pbytes/.ansible/collections', '/usr/share/ansible/collections']
pbytes@patsbyes:~$ ansible --version
ansible [core 2.12.2]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/pbytes/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.10/site-packages/ansible
  ansible collection location = /home/pbytes/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.0 (default, Oct  4 2021, 00:00:00) [GCC 11.2.1 20210728 (Red Hat 11.2.1-1)]
  jinja version = 3.0.1
  libyaml = True
pbytes@patsbyes:~$
```

The `ansible-config dump` command shows the value of _COLLECTIONS_PATHS_ shown above is currently set to the default value. The ansible --version command shows the same list but refers to it as _ansible collection location_.

We can modify this list in the ansible.cfg under the defaults section by adding the key _collections_paths_. (Below is just an illustration we won't be keeping these values in our future examples.)

```bash 
pbytes@patsbyes:~$ cat /etc/ansible/ansible.cfg
[defaults]
collections_paths = ~/.ansible/collections:/opt/share/ansible/collections
pbytes@patsbyes:~$ ansible-config dump | grep COLLECTIONS_PATHS
COLLECTIONS_PATHS(/etc/ansible/ansible.cfg) = ['/home/pbytes/.ansible/collections', '/opt/share/ansible/collections']
pbytes@patsbyes:~$
```
There are two things of note in the above example. 
1. We don't see the _/usr/share/..._ path anymore because we did not add it to the list.
2. `ansible-config dump` shows which ansible.cfg is setting these new values, just like it showed it was using defaults before.

> **NOTE:** There is an additional location where ansible searches for collections. It will search for the collections folder in the directory in which a playbook runs.

### Installing collections
One way to install a collection is at the cli with `ansible collection install <collection-name>`.

Before we do that, let's check if there are any collections on our system. We do this by using the `ansible-galaxy collection list` command.

```none
pbytes@patsbyes:~$ ansible-galaxy collection list
pbytes@patsbyes:~$
```

The command found no collections in the defined paths, so the command returns no output. 

```bash
pbytes@patsbyes:~$ ansible-config dump | grep COLLECTIONS_PATHS
COLLECTIONS_PATHS(default) = ['/home/pbytes/.ansible/collections', '/usr/share/ansible/collections']
pbytes@patsbyes:~$ ansible-galaxy collection install community.general
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/community-general-4.6.1.tar.gz to /home/pbytes/.ansible/tmp/ansible-local-38223kusp6n1k/tmpp_08hvk1/community-general-4.6.1-am3e6v2d
Installing 'community.general:4.6.1' to '/home/pbytes/.ansible/collections/ansible_collections/community/general'
community.general:4.6.1 was installed successfully
pbytes@patsbyes:~$
```

Here we install `community.general` to `/home/pbytes/.ansible/collections/`

*Some options you can pass to the install command*  

- -p "path" : Path to install the collection (overrides *COLLECTION_PATHS*)   
- -r "file" : Install collections based off a requirements file (see below) 


We can also supply the names of collections we want to install supplying a requirements.yml file.

The format of the requirements.yml file is:

```yaml
---
collections:
# With just the collection name
- namespace.collection

# With the collection name, version, and source options
- name: namespace.collection
  version: version number (default: '*')
  source: The galaxy URL to pull the collection from (defaul: '--api-server' from cli)
```

Let's look at an example:

```yaml
pbytes@patsbyes:~$ cat requirements.yml 
---
collections:
- ansible.posix

- name: ansible.netcommon
  version: 2.6.1
  source: https://galaxy.ansible.com
```
```bash
pbytes@patsbyes:~$ ansible-galaxy collection install -r requirements.yml 
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/ansible-netcommon-2.6.1.tar.gz to /home/pbytes/.ansible/tmp/ansible-local-38295sowzo9w1/tmpf0mgtt0w/ansible-netcommon-2.6.1-_7ft872e
Installing 'ansible.netcommon:2.6.1' to '/home/pbytes/.ansible/collections/ansible_collections/ansible/netcommon'
Downloading https://galaxy.ansible.com/download/ansible-utils-2.5.2.tar.gz to /home/pbytes/.ansible/tmp/ansible-local-38295sowzo9w1/tmpf0mgtt0w/ansible-utils-2.5.2-5mv0a8o3
ansible.netcommon:2.6.1 was installed successfully
Installing 'ansible.utils:2.5.2' to '/home/pbytes/.ansible/collections/ansible_collections/ansible/utils'
Downloading https://galaxy.ansible.com/download/ansible-posix-1.3.0.tar.gz to /home/pbytes/.ansible/tmp/ansible-local-38295sowzo9w1/tmpf0mgtt0w/ansible-posix-1.3.0-fe2it8qx
ansible.utils:2.5.2 was installed successfully
Installing 'ansible.posix:1.3.0' to '/home/pbytes/.ansible/collections/ansible_collections/ansible/posix'
ansible.posix:1.3.0 was installed successfully
pbytes@patsbyes:~$
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

```yaml
---
- hosts: all
  connection: ansible.netcommon.netconf
  tasks:
    - name: use lookup filter to provide xml configuration
      ansible.netcommon.netconf_config:
        content: "{{ lookup('file', './config.xml') }}"
```

To avoid a lot of typing the collections keyword can be used in the play. By doing this the namespace.
collection can then be removed from the front of the module name. The fully qualified collection
name (FQCN) for plugins and lookups will still need to be used:

```yaml
---
- hosts: all
  connection: ansible.netcommon.netconf
  collections:
    - ansible.netcommon
  tasks:
  - name: use lookup filter to provide xml configuration
    netconf_config:
      content: "{{ lookup('file', './config.xml')}}"
```

When using the collections keyword be careful of clashing namespaces (namespaces that have the
same base module name)
References
On collections in general
https://docs.ansible.com/ansible/latest/user_guide/collections_using.html
For developing collections
https://docs.ansible.com/ansible/latest/dev_guide/developing_collections.html
Ansible config settings
https://docs.ansible.com/ansible/latest/reference_appendices/config.html
