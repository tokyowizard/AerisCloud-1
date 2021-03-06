Organization
============

What is an organization?
------------------------

In AerisCloud, and organization represents a collection of roles and playbooks
and is the backbone of your projects. A new organization is generated by
running the :ref:`cloud-organization-init` command.

Organizations can be found stored in ``{data folder}/organizations`` and can
be accessed easily from the commandline using :ref:`cloud-organization-goto`.

Folder structure
----------------

 ::

  ├── README.md
  ├── env_dev.yml
  ├── env_production.yml
  └── roles
      ├── aeriscloud.disk
      ├── aeriscloud.dotdeb
      ├── aeriscloud.elasticsearch
      ├── ... more roles
      └── dependencies.txt

Notable files are ``env_production.yml`` and ``roles/dependencies.txt``. The
first one describes what we call *services*, that is a list of roles with safe
defaults that can be enabled easily from the :doc:`configuration/aeriscloud.yml`
file.

Any custom roles for your organization should be created in the ``roles`` folder
while galaxy dependencies should be declared in the ``dependencies.txt`` file.
Dependencies are automatically installed when an organization is installed or
updated.

Creating a New Role
-------------------

The recommended way to create a new role for your organization is to use ::

  cloud organizations goto my-org
  cd roles
  ansible-galaxy init my-role

You can then proceed with editing your role to add the features you need.
More informations on how to write ansible roles can be found in the
`ansible documentation`_.

An easy way to quickly test your roles on many platforms is to use the
`ansible-role-test`_ tool, see the documentation for more informations.

.. _ansible-role-test: https://github.com/aeriscloud/ansible-role-test

If possible it is recommended to create generic roles that can be pushed later
on galaxy, if your role contains sensitive informations, it can make sense to
move those to variables and create a wrapper role that depends on it and just
defines those vars with private values.

.. _ansible documentation: http://docs.ansible.com/ansible/index.html

Once your role is created, make sure to update :ref:`env-production` so that
users can use it in their :doc:`configuration/aeriscloud.yml` file.

.. _env-production:

env_production.yml
------------------

.. highlight:: yaml

This is our main playbook that will be used both on the local environments and
and production servers.

It should starts with common roles that are to be run on every servers and be
followed by a list of what we call *services*, *services* are just hosts groups
with roles applied to them, and a specially crafted comment that makes them
available for selection in the :ref:`aeris-init` command::

  --- # This is a sample env_production.yml
  # Run our pre-provisioning roles
  - hosts: all
    gather_facts: true
    sudo: true
    user: "{{ username }}"
    roles:
    - common

  # The following block registers a new service named gateway

  # service: Used to setup a front-facing gateway/firewall server
  - hosts: gateway
    gather_facts: true
    sudo: true
    roles:
    - gateway

  # Default services are services that are automatically added to a
  # new project when running aeris init, but can be removed manually
  # by the user

  # default service: Install mongodb
  - hosts: mongodb
    gather_facts: true
    sudo: true
    roles:
    - mongodb

  # Run our post-provisioning roles
  - hosts: all
    gather_facts: true
    sudo: true
    user: "{{ username }}"
    roles:
    - setup_stuff
