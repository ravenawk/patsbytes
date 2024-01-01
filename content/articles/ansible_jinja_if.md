---
title: "Quick Byte: Using Jinja templating to put code in Ansible tasks"
date: 2023-12-28T13:26:24-07:00
draft: false
---
In the `microsoft.ad.group` Ansible module, there is an option to add users and a different option to remove users. That means you would have to write two tasks, one to add and one to remove. As shown below:

```
---
- name: add or remove user from AD group
  hosts: all
  tasks:
    - name: Add members to the group, preserving existing membership
      microsoft.ad.group:
        name: "{{ groupname }}"
        scope: "{{ scope }}"
        members:
          add: "{{ usernames }}"
      when: user_option == 'add'

    - name: Remove members from the group, preserving existing membership
      microsoft.ad.group:
        name: "{{ groupname }}"
        scope: "{{ scope }}"
        members:
          remove: "{{ usernames }}"
      when: user_option == 'remove'
```

But there is an alternative; you could use one task with a Jinja if statement.

```
- name: Remove members from the group, preserving existing membership
  microsoft.ad.group:
    name: "{{ groupname }}"
    members:
      add: "{{ username if user_option == 'add' else omit }}"
      remove: "{{ username if user_option == 'remove' else omit }}"
```
Using Jinja templating like this should be used sparingly, but it reads well, and we don't have to repeat code almost word for word.
