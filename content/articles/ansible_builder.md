---
title: "Ansible-Builder: A tool for creating Execution Environments"
date: 2023-06-25T13:26:24-07:00
draft: false
---
An execution environment in Ansible refers to a container image that includes all the necessary dependencies, modules, and plugins to execute Ansible automation. It ensures consistent and reproducible execution of playbooks and roles by providing an environment with set configurations. By creating custom execution environments using ansible-builder, you can create reproducible environments tailored for the work you need to do. Ansible-builder makes maintaining and managing complex infrastructures easier while ensuring consistent and predictable outcomes. Let's use ansible-builder to create an executable environment and test that environment is working as expected. 

To build images, you need either Podman or Docker installed on the system where you install ansible-builder. To install ansible-builder, you can use pip.

`pip install anisible-builder`

The main file that calls everything else is execution-environments.yml:

```
---
version: 3

images:
  base_image:
    name: registry.redhat.io/ansible-automation-platform-23/ee-minimal-rhel8:latest

options:
  package_manager_path: /usr/bin/microdnf

dependencies:
  ansible_core:
    package_pip: ansible-core==2.14.0
  ansible_runner:
    package_pip: ansible-runner
  galaxy: requirements.yml

additional_build_files:
    - src: ansible.cfg
      dest: configs

additional_build_steps:
  prepend_galaxy:
    - ADD _build/configs/ansible.cfg /etc/ansible/ansible.cfg
```

From inside the execution environment, you can call specific collections using a requirements.yml file.

```
---
collections:
  - name: ansible.controller
  - name: infra.controller_configuration
```

Once you have all of your files set run

`ansible-builder build`

This show some output while building and then will give you a success when it is fully built.

After we finish building it we can test our execution environment with ansible-builder.

Ansible-builder is a tool for creating your own execution environments. By leveraging Ansible-builder's capabilities to bundle dependencies, modules, and plugins, organizations can ensure consistent and reliable execution of their automation workflows. You now have the knowledge and tools to confidently build execution environments tailored to your specific needs.

_Note:_ This article is currently a work in progress and will be updated frequently over the next couple days to expand out on some of the topics mentioned.
