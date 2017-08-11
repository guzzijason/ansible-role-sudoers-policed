sudoers-policed
=========

This role is for managing sudoers. In addition to simply creating sudoer config entries, it also has the addiitonal capability of purging (or policing) foreign sudoers config files that are not managed by the Ansible playbook.

Requirements
------------

As far as I can tell, no specific version required currently.

Role Variables
--------------

The following default variables (`defaults/main.yml`):

```
cloud: True          # should we install the cloud-init default user's entry?
cloud_user: centos   # the name of the default cloud-init user
sudoer_purge: False   # should we also purge sudoers files that weren't created by Ansible?*
```

\* If enabling the `sudoer_purge` feature, please be sure that the necessary ansible user is in your sudoer cofig, otherwise you could break your anisble environment. For this reason, __this is disabled by default!__

Below is an example variable file. You can break up different individual or team sudo conifgs into different files for organizational purposes, and then specify each one as a parameter of the role. The output of this role will be a single `sudoers.d/sudoers` file, with all necessary entries contained.

The `priority` variable helps to arrange the order of rules, and should be given a value in the range "01"-"99". For a given user, the last rule generally "wins", so by setting different priorities, you can control where in the file the rules will be placed - the lower the value (closest to "01") of the priority is, the closer to the top of the file these entries will be.

The rest of the vars in the example should be self-expainatory - they map to standard sudoers config options. An optional `aliases:` section can be included in order to define User_Alias, Runas_Alias, Host_Alias, or Cmnd_Alias entries.

```
# vars/example.yml

priority: 09
sudoers:
  - user: 'foobar'
    become: root
    nopasswd: True
    command: /full/path/to/command ARG1 ARG2
  - user: '%foogroup'
    become: ALL
    nopasswd: True
    command: ALL
aliases:
  - type: Runas_Alias
    name: OP
    value: root, operator
```

Dependencies
------------

If you are (optionally) policing stray sudoer config files, be sure to use the finalization post_tasks as illustrated in the example. This is not necessary, however; if you omit the finalization tasks, you will still be adding sudoer configs - just not removing strays!

__NB!__ If you are policing stray sudoers configs and your ansible envioronment depends on `sudo`, then please __be sure that the necessary user for ansible is being added to the new config!__

Example Playbook
----------------

In the example below, note that the variables `team` and `user` are functionally the same - they do exactly the same thing, which is to reference a file in `vars/{{ team }}.yml` or `vars/{{ user }}.yml`. We are including both labels in order to make the playbook more readable.

```
- hosts: servers
  become: yes
  roles:
    - { role: sudoers, team: "example" }
    - { role: sudoers, user: "someuser" }
  post_tasks:
     - name: finalize sudoers config
       include_role:
         name: sudoers
         tasks_from: finalize
       vars:
         sudoer_purge: False
```

If your playbook includes multiple plays, you may need to have a play just for your finalization `post_tasks`, and include it like so:

```
- hosts: all
  become: yes

- include: playbooks/common.yml
- include: playbooks/stuff.yml

# finalize plays must come last
- include: playbooks/finalize.yml
```

To-Do
-------

Support for detecting additional cloud environments (besides openstack) in order to configure sudo privs for the cloud-init user.

License
-------

MIT

Author Information
------------------

Jason Tucker
https://github.com/guzzijason

