---
title: "Ansible Collections - A Primer"
date: 2022-06-05
draft: false
---

Ansible content collections, collections for short, are a way to add content not included in Ansible core. Roles, playbooks, modules, and plugins can be included in a collection, allowing bundling of similar content together. 

Collections first appeared in Ansible 2.8 as a technical preview. Ansible 2.10 moved most modules from the main Ansible repository to separate collections repositories. Individual collection updates are now not reliant on the release of Ansible itself and can be released more frequently if needed. 

In this article, I will focus on using the command line to install and use Ansible collections.

### A variable that is useful to know

Before we dive into installing and using collections, let's take a moment and look at the _COLLECTIONS_PATHS_ environment variable.
This variable is a list of paths that Ansible uses to locate installed collections on the system.

> **NOTE:** There is an additional location where ansible searches for collections. It will search for the collections folder in the directory in which a playbook runs.

Let's look at a couple of ways to see this variable's value using the two command _ansible-config dump | grep COLLECTIONS_PATHS_ and _ansible --version_.

```bashsession {linenos=false,hl_lines=[2, 8]}
$ ansible-config dump | grep COLLECTIONS_PATHS
COLLECTIONS_PATHS(default) = ['/home/pbytes/.ansible/collections', '/usr/share/ansible/collections']
$ ansible --version
ansible [core 2.13.2]
  config file = None
  configured module search path = ['/home/pbytes/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/pbytes/Projects/ansible/venv/lib/python3.9/site-packages/ansible
  ansible collection location = /home/pbytes/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/pbytes/Projects/ansible/venv/bin/ansible
  python version = 3.9.2 (default, Feb 28 2021, 17:03:44) [GCC 10.2.1 20210110]
  jinja version = 3.1.2
  libyaml = True
$
```

The _ansible-config dump_ command shows the defaults for COLLECTIONS_PATHS as a list of two directories.
The _ansible --version_ command shows the same list but refers to it as _ansible collection location_ and doesn't tell you if it is set to defaults or not.

We can modify this list in our ansible.cfg. Under the defaults section, let's add the key _collections_paths_ and change one of the paths. 
We'll keep the home directory path and add _/opt/ansible/collections_ instead of the _/usr/share..._ path.

```bashsession
$ cat ansible.cfg
```
```ini
[defaults]
inventory=/etc/ansible/hosts
collections_paths=~/.ansible/collections:/opt/ansible/collections
```

Let's run the _ansible-config_ dump command again.

```bashsession {linenos=false,hl_lines=[4]}
$ ansible-config dump | grep COLLECTIONS_PATHS
COLLECTIONS_PATHS(/home/pbytes/Projects/ansible/ansible.cfg) = ['/home/pbytes/.ansible/collections', '/opt/ansible/collections']
```

Notice _ansible-config dump_ shows which ansible.cfg is setting these new values, just like it showed it was using defaults before.


Our path is configured, let's check if there are any collections on our system in those paths. We do this by using _ansible-galaxy collection list_.

```bashsession
$ ansible-galaxy collection list
$
```

The command found no collections in the defined paths, so the command returns no output. 
Depending on how you installed ansible on your system, this might have a long list of collections that your package manager included during the ansible install.

>**NOTE:** There is a bug that might return the following error.
>
>**`ERROR! - None of the provided paths were usable. Please specify a valid path with --collections-path`**
>
>This also means you have no collections in your defined paths, so you can safely ignore the error.

### Installing collections
One way to install a collection at the command line is with _ansible-galaxy collection install <collection.name>_.

```bashsession
$ ansible-galaxy collection install ansible.windows
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/ansible-windows-1.11.0.tar.gz to /home/pbytes/.ansible/tmp/ansible-local-37420dwgkpdj/tmpqwyitqni/ansible-windows-1.11.0-6w0zvtsa
Installing 'ansible.windows:1.11.0' to '/home/pbytes/.ansible/collections/ansible_collections/ansible/windows'
ansible.windows:1.11.0 was installed successfully
$
```

Here we installed the _ansible.windows_ collection. 
The ansible-galaxy command installs it to the first path listed in _COLLECTIONS_PATHS_, in our case `/home/pbytes/.ansible/collections/`

*Some options you can pass to the install command*  

- -p "path": Path to install the collection
- -r "file": Install collections based on a requirements file (see below) 


Let's look at an example of a requirements.yml file:

```bashsession
$ cat requirements.yml 
```

```yaml
---
collections: 
# Can be a list of collections names
- ansible.posix

 # Or Can be a list with name, version(opttional), and source(optional)
- name: ansible.netcommon
  version: 2.6.1
  source: https://galaxy.ansible.com
```
Let's use this requirements file and install those collections.

```bashsession
$ ansible-galaxy collection install -r collections/requirements.yml -p ./collections
Starting galaxy collection install process
[WARNING]: The specified collections path
'/home/pbytes/Projects/ansible/collections' is not part of the configured
Ansible collections paths
'/home/pbytes/.ansible/collections:/opt/ansible/collections'. The installed
collection won't be picked up in an Ansible run.
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/ansible-netcommon-2.6.1.tar.gz to /home/pbytes/.ansible/tmp/ansible-local-38033kku_abc/tmp1n81kclb/ansible-netcommon-2.6.1-zz9b60p6
Installing 'ansible.netcommon:2.6.1' to '/home/pbytes/Projects/ansible/collections/ansible_collections/ansible/netcommon'
Downloading https://galaxy.ansible.com/download/ansible-utils-2.6.1.tar.gz to /home/pbytes/.ansible/tmp/ansible-local-38033kku_abc/tmp1n81kclb/ansible-utils-2.6.1-wgeklro8
ansible.netcommon:2.6.1 was installed successfully
Installing 'ansible.utils:2.6.1' to '/home/pbytes/Projects/ansible/collections/ansible_collections/ansible/utils'
Downloading https://galaxy.ansible.com/download/ansible-posix-1.4.0.tar.gz to /home/pbytes/.ansible/tmp/ansible-local-38033kku_abc/tmp1n81kclb/ansible-posix-1.4.0-dxj412mk
ansible.utils:2.6.1 was installed successfully
Installing 'ansible.posix:1.4.0' to '/home/pbytes/Projects/ansible/collections/ansible_collections/ansible/posix'
Downloading https://galaxy.ansible.com/download/community-general-5.4.0.tar.gz to /home/pbytes/.ansible/tmp/ansible-local-38033kku_abc/tmp1n81kclb/community-general-5.4.0-wmqxdqfe
ansible.posix:1.4.0 was installed successfully
Installing 'community.general:5.4.0' to '/home/pbytes/Projects/ansible/collections/ansible_collections/community/general'
community.general:5.4.0 was installed successfully
$
```

Now let's look at what collections are on our system.

```
$ ansible-galaxy collection list

# /home/pbytes/.ansible/collections/ansible_collections
Collection        Version
----------------- -------
ansible.netcommon 2.6.1
ansible.posix     1.4.0
ansible.utils     2.6.1
community.general 5.4.0
$
```

The above shows all collections we have installed, and as expected, they are in the first path in COLLECTIONS_PATHS.
You might also notice an extra collection listed, **ansible.utils**, there was a dependency for it by ansible.netcommon.

### Using collections
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
