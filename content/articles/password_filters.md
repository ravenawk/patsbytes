---
title: "Ansible: A Look at Password and Random String Lookups"
date: 2023-09-06
draft: false
---
Ansible has many built-in lookups and filters to simplify tasks. Today, we'll look at the password lookup and the random_string lookup. Both are useful when automating tasks involving generating random strings, like user passwords. Let's compare the two and see why we would want to use one over the other.

### The Password Lookup
The password lookup generates random passwords; no surprise there.

Example of generating a random password:

```yaml
---
- hosts: localhost
  vars:
    user_password: "{{ lookup('password', './generated_password length=12 chars=ascii_letters,digits') }}"

  tasks:
    - name: print out user_password
      debug:
        var: user_password
```
Here, the password will be 12 characters long, composed of ASCII letters and digits. If you leave off the `chars=` option, the default generated passwords contain a random mix of ASCII letters, digits, and a few punctuation characters (”. , : - _”).

The password lookup can save generated passwords to a file, ensuring you have access to them later if needed. It is not necessarily recommended but useful in some cases. 

```bash
➜ cat generated_password
Bq6KP6p0M1KH
```
If you don't need to keep them, replace the file path with '/dev/null'.

> Note: If you don't use the password in your playbook, the lookup won't generate the password file. The file would not have appeared if I had just had a message like "Password generated" instead of printing the password in the debug task.

You can also pass in an option to encrypt the password using the `encrypt=` option and pass a type of encryption, such as sha512_crypt. 

For example:

```yaml
---
- hosts: localhost
  vars:
    user_password: "{{ lookup('password', './generated_password encrypt=sha512_crypt length=12 chars=ascii_letters,digits') }}"

  tasks:
    - name: print out user_password
      debug:
        var: user_password
```
The playbook run prints the encrypted password not the actual password.

If you choose the encrypt option, the seed used to encrypt the password gets saved along with the unencrypted password. 

```bash
➜ cat generated_password
2z9cOpGZs7DM salt=lQWq1jbg04vWEIJO
```

Of course, you don't get the seed if you use '/dev/null' as the location. Passing a specific seed using `seed=<some_seed>` is also an option.

The password lookup does what it's named for – it generates passwords. But this lookup might be limited if you need more control of the characters you want to use.

### The Random String Lookup

Let's look at an example of the random_string password:

```yaml
---
- hosts: localhost
  vars:
    random_value: "{{ lookup('community.general.random_string', 'length=12 chars=ascii_letters,digits') }}"

  tasks:
    - name: print out user_password
      debug:
        var: random_value
```


It looks similar to the password lookup.

The random_string lookup gives you more control over the string’s properties, including length and characters. Although it doesn’t allow automatically saving the strings it generates, you have more control over how many of each character type you want and can use a specific list of characters you set. 

Take the following example:

```yaml
---
- hosts: localhost
  vars:
    random_value: "{{ lookup('community.general.random_string', length=12,min_digit=2, override_special='@#$') }}"

  tasks:
    - name: print out user_password
      debug:
        var: random_value
```

### Weighing the Pros and Cons

When deciding between the password and random_string lookups in Ansible, it's important to understand your needs.

#### Choose password if:
- Your primary goal is to generate passwords for users or services.
- You find the auto-save feature valuable.
- You require less granular control over the generated string's properties.

#### Choose random_string if:
- Control over the string properties is important.

- The need is to generate more than passwords, like unique IDs, tokens, or other random sequences.

---

Both the password and random_string lookups in Ansible cater to different needs. By understanding the features and limitations of each, you can make a choice based on the requirements of your project.

**References**  
Ansible lookups:
https://docs.ansible.com/ansible/latest/plugins/lookup.html

Password lookup:
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/password_lookup.html

Random String lookup:
https://docs.ansible.com/ansible/latest/collections/community/general/random_string_lookup.html

