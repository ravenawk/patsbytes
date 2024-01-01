---
title: "Ansible-Navigator: Running playbooks in Execution Environments"
date: 2023-12-20:26:24-07:00
draft: false
---
The shift from Ansible Tower to the Ansible Automation Platform (AAP) represents a significant evolution in development, notably through the adoption of containerized job execution. Amidst these changes is Ansible Navigator, a new utility that consolidates multiple Ansible tools into one. This first article is just an introduction to Ansible Navigator; additional articles that explain different parts and usage will follow.

### Installation
For Red Hat subscribers, ensure you have the `ansible-automation-platform-2.4-for-rhel-9-x86_64-rpms` repo enabled (adjust the version numbers according to your specific AAP and RHEL versions), then run `sudo dnf install ansible-navigator`.

For a more general approach, Install Ansible Navigator using pip3 — this requires a container engine (either Docker or Podman) pre-installed on your system.

Although I use Fedora 39 in this guide, you can adapt the instructions for other operating systems. Fedora has Podman installed on a typical install, which can vary on other OSes. In addition to Podman, we also need to install the `python3-pip` package. As with Podman, other OSes may name the package differently. When those requirements are out of the way, we can install Navigator with the command:

`pip3 install ansible-navigator`

### Exploring ansible-navigator

With ansible-navigator installed you can run it by typing:


```
$ ansible-navigator
---------------------------------------------------------------------
Execution environment image and pull policy overview
---------------------------------------------------------------------
Execution environment image name:     ghcr.io/ansible/creator-ee:v0.20.0
Execution environment image tag:      v0.20.0
Execution environment pull arguments: None
Execution environment pull policy:    tag
Execution environment pull needed:    True
---------------------------------------------------------------------
Updating the execution environment
---------------------------------------------------------------------
Running the command: podman pull ghcr.io/ansible/creator-ee:v0.20.0
Trying to pull ghcr.io/ansible/creator-ee:v0.20.0...
Getting image source signatures
Copying blob 6a4437d12ad5 done   |
Copying blob 6fe2ec5efede done   |
Copying blob eaac7334e7ab done   |
Copying blob 7b05cec1a1b5 done   |
Copying blob 0afce0612807 done   |
Copying blob dea7f84d568f done   |
Copying blob 90e5f6bd07c8 done   |
Copying blob e194cc6358ad done   |
Copying blob 322be10184d0 done   |
Copying blob b61759fbc163 done   |
Copying blob 39b21c1fa8b3 done   |
Copying blob 09a131db2ab7 done   |
Copying blob 807c75530636 done   |
Copying config 278a94cdb5 done   |
Writing manifest to image destination
278a94cdb5159adf04f0da2d8c84a9fb3368fe368627806a5ac38e96810b7d45
```

The first time you run it, it will download a creator-ee container as shown above.

Once that't done you will see the following screen:
![Ansible Navigator](/images/ansible-navigator.png)

You can see from the list that some tools incorporated into ansible-navigator are ansible-doc, ansible-playbook, and ansible-inventory. For now, let's look at the images option.

Before we do that let's exit out and download the homelab-ee that is an evolution of the execution environment I made in the article, [Ansible Builder]({{< ref "ansible_builder.md" >}} ), by doing a podman pull.

Now let's reopen ansible-navigator and go to the images section. You do that by typing a colon followed by the option you want, so in our case `:images`.

![Images](/images/images.png)

> Note it tells us if the container images we have on our system are usable as execution environments.

We have two container images on our system, as shown above. The default image is the creator-ee; we will look at one way to change that later.

Let's examine homelab-ee; you do that by pressing the number next to the image name, in our case, 1.

![Image Info](/images/image_info.png)

It shows the image name at the top and gives us more choices. Let's look at info about Ansible and Python packages installed in the execution environment. You can select "Ansible version and collections" by pressing 2.

![Ansible Info](/images/ansible_info.png)

Here, we can see all the collections and the version of Ansible installed in the execution environment.

Let's look at one more option. Hitting 'esc' takes us back one page and then let's select "Python packages" by hitting 3.

This gives us a list of all the Python packages installed on the system. If we use the up and down cursor keys it scrolls through the list, in this case there are 71 Python packages installed in the image. An interesting one to note is the *proxmoxer* package #37 which was installed in the execution environment to be albe to run the proxmox modules.

I will leave the rest of the options under images up to you to explore.
![Python packages](/images/python_packages.png)


### Configuring Ansible Navigator
First lets see what kind of options we have, let's run:

`ansible-navigator --help`

```
$ ansible-navigator --help
Usage: ansible-navigator [options]

Options (global):
 -h     --help                                   Show this help message and exit
 --version                                       Show the application version and
                                                 exit
 --rad  --ansible-runner-artifact-dir            The directory path to store
                                                 artifacts generated by ansible-
                                                 runner
 --rac  --ansible-runner-rotate-artifacts-count  Keep ansible-runner artifact
                                                 directories, for last n runs, if
                                                 set to 0 artifact directories
                                                 won't be deleted
 --rt   --ansible-runner-timeout                 The timeout value after which
                                                 ansible-runner will forcefully
                                                 stop the execution
 --rwje --ansible-runner-write-job-events        Write ansible-runner job_events in
                                                 the artifact directory
                                                 (true|false)
 --cdcp --collection-doc-cache-path              The path to collection doc cache
                                                 (default:
                                                 /home/pmartin/.cache/ansible-
                                                 navigator/collection_doc_cache.db)
 --ce   --container-engine                       Specify the container engine
                                                 (auto=podman then docker)
                                                 (auto|podman|docker) (default:
                                                 auto)
 --co   --container-options                      Extra parameters passed to the
                                                 container engine command
 --dc   --display-color                          Enable the use of color for mode
                                                 interactive and stdout
                                                 (true|false) (default: true)
 --ecmd --editor-command                         Specify the editor command
                                                 (default: /usr/bin/nano
                                                 {filename})
 --econ --editor-console                         Specify if the editor is console
                                                 based (true|false) (default: true)
 --ee   --execution-environment                  Enable or disable the use of an
                                                 execution environment (true|false)
                                                 (default: true)
 --eei  --execution-environment-image            Specify the name of the execution
                                                 environment image (default:
                                                 ghcr.io/ansible/creator-
                                                 ee:v0.20.0)
 --eev  --execution-environment-volume-mounts    Specify volume to be bind mounted
                                                 within an execution environment
                                                 (--eev
                                                 /home/user/test:/home/user/test:Z)
 --la   --log-append                             Specify if log messages should be
                                                 appended to an existing log file,
                                                 otherwise a new log file will be
                                                 created per session (true|false)
                                                 (default: true)
 --lf   --log-file                               Specify the full path for the
                                                 ansible-navigator log file
                                                 (default: /home/pmartin/ansible-
                                                 navigator.log)
 --ll   --log-level                              Specify the ansible-navigator log
                                                 level (debug|info|warning|error|cr
                                                 itical) (default: warning)
 -m     --mode                                   Specify the user-interface mode
                                                 (stdout|interactive) (default:
                                                 interactive)
 --osc4 --osc4                                   Enable or disable terminal color
                                                 changing support with OSC 4
                                                 (true|false) (default: true)
 --penv --pass-environment-variable              Specify an existing environment
                                                 variable to be passed through to
                                                 and set within the execution
                                                 environment (--penv MY_VAR)
 --pa   --pull-arguments                         Specify any additional parameters
                                                 that should be added to the pull
                                                 command when pulling an execution
                                                 environment from a container
                                                 registry. e.g. --pa='--tls-
                                                 verify=false'
 --pp   --pull-policy                            Specify the image pull policy
                                                 always:Always pull the image,
                                                 missing:Pull if not locally
                                                 available, never:Never pull the
                                                 image, tag:if the image tag is
                                                 'latest', always pull the image,
                                                 otherwise pull if not locally
                                                 available
                                                 (always|missing|never|tag)
                                                 (default: tag)
 --senv --set-environment-variable               Specify an environment variable
                                                 and a value to be set within the
                                                 execution environment (--senv
                                                 MY_VAR=42)
 --tz   --time-zone                              Specify the IANA time zone to use
                                                 or 'local' to use the system time
                                                 zone (default: utc)

Subcommands:
 {subcommand} --help
  builder                                        Build [execution environment](http
                                                 s://docs.ansible.com/ansible/devel
                                                 /getting_started_ee/index.html)
                                                 (container image)
  collections                                    Explore available collections
  config                                         Explore the current ansible
                                                 configuration
  doc                                            Review documentation for a module
                                                 or plugin
  exec                                           Run a command within an execution
                                                 environment
  images                                         Explore execution environment
                                                 images
  inventory                                      Explore an inventory
  lint                                           Lint a file or directory for
                                                 common errors and issues
  replay                                         Explore a previous run using a
                                                 playbook artifact
  run                                            Run a playbook
  settings                                       Review the current ansible-
                                                 navigator settings
  welcome                                        Start at the welcome page
```

There are a lot of different options and subcommands, let's take a look at the help page for the settings subcommand.

```
$ ansible-navigator settings --help
Usage: ansible-navigator settings [options]

settings: Review the current ansible-navigator settings

Options (global):
 -h     --help                                   Show this help message and exit
 --version                                       Show the application version and
                                                 exit
 --rad  --ansible-runner-artifact-dir            The directory path to store
                                                 artifacts generated by ansible-
                                                 runner
 --rac  --ansible-runner-rotate-artifacts-count  Keep ansible-runner artifact
                                                 directories, for last n runs, if
                                                 set to 0 artifact directories
                                                 won't be deleted
 --rt   --ansible-runner-timeout                 The timeout value after which
                                                 ansible-runner will forcefully
                                                 stop the execution
 --rwje --ansible-runner-write-job-events        Write ansible-runner job_events in
                                                 the artifact directory
                                                 (true|false)
 --cdcp --collection-doc-cache-path              The path to collection doc cache
                                                 (default:
                                                 /home/pmartin/.cache/ansible-
                                                 navigator/collection_doc_cache.db)
 --ce   --container-engine                       Specify the container engine
                                                 (auto=podman then docker)
                                                 (auto|podman|docker) (default:
                                                 auto)
 --co   --container-options                      Extra parameters passed to the
                                                 container engine command
 --dc   --display-color                          Enable the use of color for mode
                                                 interactive and stdout
                                                 (true|false) (default: true)
 --ecmd --editor-command                         Specify the editor command
                                                 (default: /usr/bin/nano
                                                 {filename})
 --econ --editor-console                         Specify if the editor is console
                                                 based (true|false) (default: true)
 --ee   --execution-environment                  Enable or disable the use of an
                                                 execution environment (true|false)
                                                 (default: true)
 --eei  --execution-environment-image            Specify the name of the execution
                                                 environment image (default:
                                                 ghcr.io/ansible/creator-
                                                 ee:v0.20.0)
 --eev  --execution-environment-volume-mounts    Specify volume to be bind mounted
                                                 within an execution environment
                                                 (--eev
                                                 /home/user/test:/home/user/test:Z)
 --la   --log-append                             Specify if log messages should be
                                                 appended to an existing log file,
                                                 otherwise a new log file will be
                                                 created per session (true|false)
                                                 (default: true)
 --lf   --log-file                               Specify the full path for the
                                                 ansible-navigator log file
                                                 (default: /home/pmartin/ansible-
                                                 navigator.log)
 --ll   --log-level                              Specify the ansible-navigator log
                                                 level (debug|info|warning|error|cr
                                                 itical) (default: warning)
 -m     --mode                                   Specify the user-interface mode
                                                 (stdout|interactive) (default:
                                                 interactive)
 --osc4 --osc4                                   Enable or disable terminal color
                                                 changing support with OSC 4
                                                 (true|false) (default: true)
 --penv --pass-environment-variable              Specify an existing environment
                                                 variable to be passed through to
                                                 and set within the execution
                                                 environment (--penv MY_VAR)
 --pa   --pull-arguments                         Specify any additional parameters
                                                 that should be added to the pull
                                                 command when pulling an execution
                                                 environment from a container
                                                 registry. e.g. --pa='--tls-
                                                 verify=false'
 --pp   --pull-policy                            Specify the image pull policy
                                                 always:Always pull the image,
                                                 missing:Pull if not locally
                                                 available, never:Never pull the
                                                 image, tag:if the image tag is
                                                 'latest', always pull the image,
                                                 otherwise pull if not locally
                                                 available
                                                 (always|missing|never|tag)
                                                 (default: tag)
 --senv --set-environment-variable               Specify an environment variable
                                                 and a value to be set within the
                                                 execution environment (--senv
                                                 MY_VAR=42)
 --tz   --time-zone                              Specify the IANA time zone to use
                                                 or 'local' to use the system time
                                                 zone (default: utc)

Options (settings subcommand):
 --se   --effective                              Show the effective settings.
                                                 Defaults, CLI parameters,
                                                 environment variables, and the
                                                 settings file will be combined
 --gs   --sample                                 Generate a sample settings file
 --ss   --schema                                 Generate a schema for the settings
                                                 file ('json'= draft-07 JSON
                                                 Schema) (json) (default: json)
 --so   --sources                                Show the source of each current
                                                 settings entry
```

The option I want to look at here is --gs.

```
$ ansible-navigator settings --gs
---
ansible-navigator:
#   ansible:
#     config:
#       # Help options for ansible-config command in stdout mode
#       help: False
#       # Specify the path to the ansible configuration file
#       path: ./ansible.cfg
#     # Extra parameters passed to the corresponding command
#     cmdline: "--forks 15"
#     doc:
#       # Help options for ansible-doc command in stdout mode
#       help: False
#       plugin:
#         # Specify the plugin name
#         name: debug
#         # Specify the plugin type, 'become', 'cache', 'callback', 'cliconf',
#         # 'connection', 'filter', 'httpapi', 'inventory', 'keyword', 'lookup',
#         # 'module', 'netconf', 'role', 'shell', 'strategy', 'test' or 'vars'
#         type: module
#     inventory:
#       # Help options for ansible-inventory command in stdout mode
#       help: True
#       # Specify an inventory file path or comma separated host list
#       entries:
#         - host1,
#         - router1,router2
#         - inventory.yml
#     playbook:
#       # Help options for ansible-playbook command in stdout mode
#       help: False
#       # Specify the playbook name
#       path: site.yml
#   ansible-builder:
#     # Help options for ansible-builder command in stdout mode
#     help: False
#     # Specify the path that contains ansible-builder manifest files
#     workdir: /tmp/
#   ansible-lint:
#     # Specify the path to the ansible-lint configuration file
#     config: ~/lint-config.yml
#     # Path to files on which to run ansible-lint
#     lintables: ~/myproject/
#   ansible-runner:
#     # The directory path to store artifacts generated by ansible-runner
#     artifact-dir: ./runner-artifacts
#     # Keep ansible-runner artifact directories, for last n runs, if set to 0
#     # artifact directories won't be deleted
#     rotate-artifacts-count: 10
#     # The timeout value after which ansible-runner will forcefully stop the
#     # execution
#     timeout: 300
#     # Write ansible-runner job_events in the artifact directory
#     job-events: True
#   # Subcommands
#   app: welcome
#   # The path to collection doc cache
#   collection-doc-cache-path: $HOME/.cache/ansible-navigator/collection_doc_cache.db
#   color:
#     # Enable the use of color for mode interactive and stdout
#     enable: True
#     # Enable or disable terminal color changing support with OSC 4
#     osc4: True
#   editor:
#     # Specify the editor command
#     command: vim_from_setting
#     # Specify if the editor is console based
#     console: False
#   # Enable prompts for password and in playbooks. This will set mode to
#   # stdout and disable playbook artifact creation
#   enable-prompts: False
#   exec:
#     # Specify the exec command should be run in a shell
#     shell: True
#     # Specify the command to run within the execution environment
#     command: /bin/bash
#   execution-environment:
#     # Specify the container engine (auto=podman then docker)
#     container-engine: auto
#     # Extra parameters passed to the container engine command
#     container-options:
#       - "--net=host"
#     # Enable or disable the use of an execution environment
#     enabled: True
#     environment-variables:
#       # Specify an existing environment variable to be passed through to and
#       # set within the execution environment (--penv MY_VAR)
#       pass:
#         - ONE
#         - TWO
#         - THREE
#       # Specify an environment variable and a value to be set within the
#       # execution environment (--senv MY_VAR=42)
#       set:
#         KEY1: VALUE1
#         KEY2: VALUE2
#         KEY3: VALUE3
#     # Specify the name of the execution environment image
#     image: quay.io/organization/custom-ee:latest
#     pull:
#       # Specify any additional parameters that should be added to the pull
#       # command when pulling an execution environment from a container
#       # registry. e.g. --pa='--tls-verify=false'
#       arguments:
#         - "--tls-verify=false"
#       # Specify the image pull policy always:Always pull the image,
#       # missing:Pull if not locally available, never:Never pull the image,
#       # tag:if the image tag is 'latest', always pull the image, otherwise
#       # pull if not locally available
#       policy: tag
#     # Specify volume to be bind mounted within an execution environment
#     # (--eev /home/user/test:/home/user/test:Z)
#     volume-mounts:
#       - src: "/tmp/directory"
#         dest: "/tmp/directory"
#         options: "Z"
#   # Specify the format for stdout output.
#   format: json
#   images:
#     # Provide detailed information about the selected execution environment
#     # image
#     details:
#       - ansible_collections
#       - ansible_version
#   # Specify a host attribute to show in the inventory view
#   inventory-columns:
#     - ansible_network_os
#     - ansible_network_cli_ssh_type
#     - ansible_connection
  logging:
#     # Specify the ansible-navigator log level
    level: debug
#     # Specify if log messages should be appended to an existing log file,
#     # otherwise a new log file will be created per session
    append: False
#     # Specify the full path for the ansible-navigator log file
#     file: $PWD/ansible-navigator.log
#   # Specify the user-interface mode
#   mode: interactive
#   playbook-artifact:
#     # Enable or disable the creation of artifacts for completed playbooks.
#     # Note: not compatible with '--mode stdout' when playbooks require user
#     # input
#     enable: True
#     # Specify the path for the playbook artifact to replay
#     replay: /tmp/test_artifact.json
#     # Specify the name for artifacts created from completed playbooks. The
#     # following placeholders are available: {playbook_dir}, {playbook_name},
#     # {playbook_status}, and {time_stamp}
#     save-as: "{playbook_dir}/{playbook_name}-artifact-{time_stamp}.json"
#   settings:
#     # Show the effective settings. Defaults, CLI parameters, environment
#     # variables, and the settings file will be combined
#     effective: False
#     # Generate a sample settings file
#     sample: False
#     # Generate a schema for the settings file ('json'= draft-07 JSON Schema)
#     schema: json
#     # Show the source of each current settings entry
#     sources: False
#   # Specify the IANA time zone to use or 'local' to use the system time
#   # zone
#   time-zone: UTC
```
Here you can set all kind of options, if you look under `execution-environment` you can see how we specify `image` the default image used by ansible-navigator. In this case its the creator-ee that navigator downloaded when we first started it up. Let's make that our homelab-ee.

>**Ansible Navigator config locations checked**
>
>First match is used(this may change in future releases):
>
>|
>|----------------------------------------|-----------------------------------------|
>| ANSIBLE_NAVIGATOR_CONFIG               | env variable                            |
>| ./ansible-navigator.\<ext\>            | No dot at begining of the file name, this is in the project directory |
>| ~/.ansible-navigator.\<ext\>           | Dot at the beginning of the file name, this is in the user home directory |
>
>__\<ext\> can be either yaml/yml or json debending on what format the file is in.__

In our local directory we will add an ansible-navigator.yml file and add our custom ee.
```
---
ansible-navigator:
  execution-environment:
    image: registry.patsbytes.net/homelab-ee:latest
```

Running ansible-navigator and selecting images we see that our image is now default:

![homelab-ee default](/images/homelab-ee_default.png)

As we wrap up this intro to Ansible Navigator, you now know how to install and perform basic configuration. Watch for future articles where we'll dive deeper into other parts of Ansible Navigator. Until then, happy automating, and stay tuned for more insights and guides!

### References and further reading:

Ansible Navigator documentation:
https://ansible.readthedocs.io/projects/navigator/

Installing Ansible Navigator:
https://ansible.readthedocs.io/projects/navigator/installation/

Installing Packages: (Python)
https://packaging.python.org/en/latest/tutorials/installing-packages/

Ansible Navigator settings:
https://ansible.readthedocs.io/projects/navigator/settings/
