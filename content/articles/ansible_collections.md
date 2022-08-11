---
title: "Ansible Collections - A Primer"
date: 2022-06-05
draft: false
---

Ansible content collections, collections for short, are a way to add content not included in Ansible core. Roles, playbooks, modules, and plugins can be included in a collection, allowing bundling of similar content together. 

Collections first appeared in Ansible 2.8 as a technical preview. Ansible 2.10 moved most modules from the main Ansible repository to separate collections repositories. Individual collection updates are now not reliant on the release of Ansible itself and can be released more frequently if needed. 

In this article, I will focus on using the command line to install and use Ansible collections.

### A variable that is useful to know

Before we dive into installing and using collections, let's take a moment and look at the _COLLECTIONS_PATHS_ environmental variable.

The _COLLECTIONS_PATHS_ environment variable is a list of paths that Ansible uses to locate installed collections on the system.

> **NOTE:** There is an additional location where ansible searches for collections. It will search for the collections folder in the directory in which a playbook runs.

Let's look at a couple ways to see this variable's value.

`ansible-config dump | grep COLLECTIONS_PATHS`

and

`ansible --version`

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

The `ansible-config dump` command shows that **COLLECTIONS_PATHS** is currently set to two directries and they are the default values. 
The `ansible --version` command shows the same list but refers to it as **ansible collection location** and doesn't tell you if it is set to defaults or not.

We can modify this list, in our ansible.cfg under the **defaults** section let's add the key **_collections_paths_** and add two paths. 
Let's keep the home directory path and add `/opt/ansible/collections` instead of the /usr/share path.

```bashsession {linenos=false,hl_lines=[4]}
$ cat ansible.cfg
[defaults]
inventory=/etc/ansible/hosts
collections_paths=~/.ansible/collections:/opt/ansible/collections
```

Let's run the ansible-config dump command again.

```bashsession {linenos=false,hl_lines=[4]}
$ ansible-config dump | grep COLLECTIONS_PATHS
COLLECTIONS_PATHS(/home/pbytes/Projects/ansible/ansible.cfg) = ['/home/pbytes/.ansible/collections', '/opt/ansible/collections']
```

Notice `ansible-config dump` shows which ansible.cfg is setting these new values, just like it showed it was using defaults before.

### Installing collections
We install a collection at the command line with `ansible-galaxy collection install <collection-name>`.

Before we do that, let's check if there are any collections on our system. We do this by using the `ansible-galaxy collection list` command.

```bashsession
$ ansible-galaxy collection list
$
```

The command found no collections in the defined paths, so the command returns no output. 
Depending on how you installed ansible on your system this might have a long list of collections that your package manager includes. 

>**NOTE:** If your collections paths are set but you get an error with the comment: 
>
>**`ERROR! - None of the provided paths were usable. Please specify a valid path with --collections-path`**
>
>This is another form of ansible telling you that you have no collections in your collection paths. 
>This is a bug that might be fixed by the time you read this but wanted to include it just in case.
>This is just like above where nothing is returned, so you can safely ignore the error.

Let's install a collection.

```bashsession
$ ansible-galaxy collection install community.general
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/community-general-5.4.0.tar.gz to /home/pbytes/.ansible/tmp/ansible-local-3884l5oljsp/tmpozyir8e3/community-general-5.4.0-e9ebd_hd
Installing 'community.general:5.4.0' to '/home/pbytes/.ansible/collections/ansible_collections/community/general'
community.general:5.4.0 was installed successfully
$
```

Here we install `community.general` collection. 
The ansible-galaxy command installs it to the first path listed in **COLLECTIONS_PATHS**, in our case `/home/pbytes/.ansible/collections/`

*Some options you can pass to the install command*  

- -p "path" : Path to install the collection
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
$ cat requirements.yml 
---
collections:
- ansible.posix

- name: ansible.netcommon
  version: 2.6.1
  source: https://galaxy.ansible.com
```
```bash
$ ansible-galaxy collection install -r requirements.yml 
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
$
```

Now let's look at what collections are on our system.

```
pbytes@patsbyes:~$ ansible-galaxy collection list

# /home/pbytes/.ansible/collections/ansible_collections
Collection        Version
----------------- -------
ansible.netcommon 2.6.1
ansible.posix     1.3.0
ansible.utils     2.5.2
community.general 4.6.1
pbytes@patsbyes:~$
```

The above shows all collections we have installed, and as expected, they are in the first path in the COLLECTIONS_PATHS variable.
You might also notice an extra collection listed, ansible.utils, that is because of a dependency by ansible.netcommon.

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
