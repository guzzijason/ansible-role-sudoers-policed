sudoers-policed
=========

This role is for managing sudoers. In addition to simply creating sudoer config entries, it also has the addiitonal capability of purging (or policing) foreign sudoers config files that are not managed by the Ansible playbook.

Requirements
------------

This role requires at least Ansible 2.3, as it relies on the `tempfile` module which does not exist in earlier versions. Also, requires root access to do its job, so `become: yes`.

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

The rest of the vars in the example should be self-expainatory - they map to standard sudoers config options.

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
```

Dependencies
------------

It is important to note that there are "finalization" tasks as part of this role, that *MUST* be run at the end of your playbook (see examples). The main portion of the role simply generates temporary file fragments, but doesn't install them. This allows for the role to be called multiple times from various plays, if necessary. The finalization steps then assemble and install the finished file. Optionally, the policing of "foreign" sudoer files will happen during finalization as well.

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

