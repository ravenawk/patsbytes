---
title: "Ansible-Builder: A tool for creating Execution Environments"
date: 2023-06-25T13:26:24-07:00
draft: false
---
An execution environment in Ansible refers to a container image that includes all the necessary dependencies, modules, and plugins to execute Ansible automation. It ensures consistent and reproducible execution of playbooks and roles by providing an environment with set configurations. By creating custom execution environments using ansible-builder, you can create reproducible environments tailored for the work you need to do. Ansible-builder makes maintaining and managing complex infrastructures easier while ensuring consistent and predictable outcomes. Let's use ansible-builder to create an executable environment and test that environment is working as expected. 

You must have either Podman or Docker installed on the system where you intend to install Ansible-builder. Once you have Podman or Docker, the next step is installing Ansible-builder. One method for installation is using pip, the Python package manager. Pip correctly installs dependencies and simplifies the installation process.

The basic command to install Ansible-builder with pip is by running the following:

`pip install ansible-builder`

You will see dependency packages installed in addition to ansible-builder. You can use other options with pip, such as using a Python virtual environment or installing it with the --user flag. Look at the documentation for pip to see what options are available.

Let’s build an execution environment that we can use to control a Proxmox server. Proxmox VE is an open-source server for managing virtual environments, similar to VMware vSphere. You can use various files when building your execution environment, but the main file is execution-environment.yml.

```yaml {title="execution-environment.yml"}
---
version: 3

images:
  base_image:
    name: quay.io/ansible/awx-ee:latest

dependencies:
  ansible_core:
    package_pip: ansible-core==2.14.0
  ansible_runner:
    package_pip: ansible-runner
  galaxy: 
    collections:
      - community.general
  python:
    - proxmoxer
    - requests

additional_build_files:
    - src: ansible.cfg
      dest: configs

additional_build_steps:
  prepend_final:
    - ADD _build/configs/ansible.cfg /etc/ansible/ansible.cfg
```

The above execution-environment.yml has all the Ansible and Python requirements in line. You can also specify them as separate files, such as requirements.yml (Ansible) and requirements.txt (python).

```yaml
--snip--
dependencies:
  ansible_core:
    package_pip: ansible-core==2.14.0
  ansible_runner:
    package_pip: ansible-runner
  galaxy: requirements.yml
  python: requirements.txt
--snip--
```
I have included an empty ansible.cfg here to show how you include it in your image build. Feel free to put any options in your ansible.cfg that you use in your execution environment. Our execution-environment.yml will put this ansible.cfg in the container image where we specify. We will look more at this later on.

Once you have all of your files set, you can run:

`ansible-builder build`

This is the message I see when running it:

```yaml
Running command:
  podman build -f context/Containerfile -t ansible-execution-env:latest context
Complete! The build context can be found at: /home/pmartin/ansible-builder/context
```
After the build process finishes you should have the image available to use.

```bash
$ podman images
REPOSITORY              TAG         IMAGE ID      CREATED        SIZE
localhost/proxmox-env   latest      0ce7182e8bab  4 minutes ago  1.58 GB
---snip---
```
We can test our execution environment in a number of ways. Let's start by looking at the collections in it using podman.

```bash
$ podman run localhost/proxmox-env ansible-galaxy collection list

# /usr/share/ansible/collections/ansible_collections
Collection              Version
----------------------- -------
amazon.aws              6.2.0
ansible.posix           1.5.4
ansible.windows         2.0.0
awx.awx                 22.6.0
azure.azcollection      1.16.0
community.general       7.2.1
community.vmware        3.8.0
google.cloud            1.2.0
kubernetes.core         2.4.0
openstack.cloud         2.1.0
ovirt.ovirt             3.1.2
redhatinsights.insights 1.0.8
theforeman.foreman      3.12.0
```

For comparison this is a list of the collections that were already in the container we used as a base image.

```bash
$ podman run quay.io/ansible/awx-ee ansible-galaxy collection list

# /usr/share/ansible/collections/ansible_collections
Collection              Version
----------------------- -------
amazon.aws              6.2.0
ansible.posix           1.5.4
ansible.windows         2.0.0
awx.awx                 22.6.0
azure.azcollection      1.16.0
community.vmware        3.8.0
google.cloud            1.2.0
kubernetes.core         2.4.0
openstack.cloud         2.1.0
ovirt.ovirt             3.1.2
redhatinsights.insights 1.0.8
theforeman.foreman      3.12.0
```

The base image included a lot of collections already. But for our custom execution environment, it needed the 'community.general' collection, which contains the proxmox module.

We have the collection we need for working with proxmox, now, let's test it. We will be using ansible-navigator to execute the playbook using our execution environment. A future article will go deeper into ansible-navigator.

This is simple playbook we will be running.

```yaml
---
- hosts: localhost
  connection: local
  vars_files:
    - vault.yml
  tasks:
    - name: Create vms from a list
      community.general.proxmox_kvm:
        api_user: "{{ api_user }}"
        api_password: "{{ api_password }}"
        api_host: 10.10.10.175
        validate_certs: false
        clone: rh8-tmplt
        name: prox-ee-test
        node: pve
        storage: Local_lun
        timeout: 300

```
Here is the playbook run.

```bash
ansible-navigator run --eei localhost/proxmox-ee clone_vm_proxmox.yml --pp=never -m stdout -i inventory

PLAY [localhost] ***************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Create vms from a list] **************************************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

This article isn't about ansible-navigator, but here is what each of the option does.

|||
|---|---------------|
|run|runs a playbook|
|--eei|What execution environment to use|
|--pp|Pull policy, whether or not to pull the image|
|-m | Output to stdout like a playbook instead of opening navigator|

After the playbook run if we check proxmox we see our vm.

![Proxmox vm](/images/proxmox.png)

It works! Now we can share that execution environment with others and know that everyone is working in the same environment. Now let's go back and check one last thing.

When we ran the build command with ansible-builder it creates a directory, context, where it stores files used in the build.

```bash
context
├── _build
│   ├── configs
│   │   └── ansible.cfg
│   ├── requirements.txt
│   ├── requirements.yml
│   └── scripts
│       ├── assemble
│       ├── check_ansible
│       ├── check_galaxy
│       ├── entrypoint
│       ├── install-from-bindep
│       └── introspect.py
└── Containerfile

```

The ones of note are the two requirement files, they contain what you expect, Python module and collection module lists. We see our ansible.cfg has been copied into the _build directory and a Containerfile that handles the actual creation of the image. I will leave it to you to explore and inspect what items are added.

Ansible-builder is a tool for creating your own execution environments. By leveraging Ansible-builder's capabilities to bundle dependencies, modules, and plugins, organizations can ensure consistent and reliable execution of their automation workflows. This article touched on just the tip of things you can do with ansible-builder, be sure to check out the documentation to see more options.

### References and further reading:

Ansible Builder github:
https://github.com/ansible/ansible-builder

Installing Packages: (Python)
https://packaging.python.org/en/latest/tutorials/installing-packages/

Proxmox:
https://proxmox.com/en/

Ansible Navigator github:
https://github.com/ansible/ansible-navigator
