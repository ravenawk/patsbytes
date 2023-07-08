---
title: "Ansible-Builder: A tool for creating Execution Environments"
date: 2023-06-25T13:26:24-07:00
draft: false
---
An execution environment in Ansible refers to a container image that includes all the necessary dependencies, modules, and plugins to execute Ansible automation. It ensures consistent and reproducible execution of playbooks and roles by providing an environment with set configurations. By creating custom execution environments using ansible-builder, you can create reproducible environments tailored for the work you need to do. Ansible-builder makes maintaining and managing complex infrastructures easier while ensuring consistent and predictable outcomes. Let's use ansible-builder to create an executable environment and test that environment is working as expected. 

You must have either Podman or Docker installed on the system where you intend to install Ansible-builder. Once you have Podman or Docker installed, the next step is installing Ansible-builder. The recommended method for installation is using pip, the Python package manager. Pip properly installs dependencies and simplifies the installation process.

The basic command to install Ansible-builder with pip is by run the following:

`pip install anisible-builder`

You will see some dependency packages installed in addition to ansible-builder. You can pass or use other options, such as using a Python virtual environment or installing it with the `--user` flag. Look at the documentation for pip to see what options are available.

Let's build an execution environment that we can use to control a Proxmox server. Proxmox VE is an open-source server for managing virtual environments, similar to VMware vSphere. You can use various files when building your execution environment, but the main file is *execution-environment.yml*. 

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
The above execution-environment.yml has all the ansible and python requirements inline. You can also specify them as separate files, such as requirements.yml (Ansible) and requirements.txt (python). Such as:

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

Once you have all of your files set run

`ansible-builder build`

This show some output while building and then will give you a success when it is fully built.

After we finish building it we can test our execution environment with ansible-builder.

Ansible-builder is a tool for creating your own execution environments. By leveraging Ansible-builder's capabilities to bundle dependencies, modules, and plugins, organizations can ensure consistent and reliable execution of their automation workflows. You now have the knowledge and tools to confidently build execution environments tailored to your specific needs.

_Note:_ This article is currently a work in progress and will be updated frequently over the next couple days to expand out on some of the topics mentioned.
