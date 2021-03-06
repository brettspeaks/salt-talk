:data-transition-duration: 1000
:skip-help: False
:auto-console: True
:css: css/presentation.css

.. title: SaltStack

----

SaltStack
=======================

What is salt?

  *"Salt is a new approach to infrastructure management built on a dynamic communication bus.
  Salt can be used for data-driven orchestration, remote execution for any infrastructure,
  configuration management for any app stack, and much more."*

----

Installing and Configuring a minion
=====================================

Note: Skip this if you use any of our custom gce images (ex. ``centos-7-bare-20160817``) you do not need to complete these
steps as salt-minion is part of our base image.

1. Install with yum/apt-get

    .. code:: bash

      sudo yum install salt-minion

#. Configure the minion to talk to the salt master.

    .. code:: bash

      sudo vi /etc/salt/minion

      salt-master: salt1

#. Restart salt-minion

    .. code:: bash

      sudo service restart salt-minion

----

But wait, this isn't working!
=============================

salt-key
--------
  *"Salt-key executes simple management of Salt server public keys used for authentication."*

It is basically salt's way of authenticating a minion. On inital connection, the minion will send the salt-master its
public key so on the salt master we have to manually accept/reject the minion's key.\*

The salt-key command has a lot of options, but the ones I use the most:

    -l ARG              List the public keys. The args "pre", "un", and
                        "unaccepted" will list unaccepted/unsigned keys. "acc"
                        or "accepted" will list accepted/signed keys. "rej" or
                        "rejected" will list rejected keys. "den" or "denied"
                        will list denied keys. Finally, "all" will list all
                        keys.
    --list-all          List all public keys.
    -a ACCEPT           Accept the specified public key (use --include-all to
                        match rejected keys in addition to pending keys).
                        Globs are supported.
    -A                  Accept all pending keys
    -r REJECT           Reject the specified public key (use --include-all to
                        match accepted keys in addition to pending keys).
                        Globs are supported.
    -d DELETE           Delete the specified key. Globs are supported.


\* NOTE: You don't have to do this on salt2 (thin client salt master) because auto key accept is turned on. Though, if
rebuild a machine you may have to delete the thin client's public key and have it re-added to the list.

----

Minion Targeting
=================

  .. code:: bash

    sudo salt 'test1' test.ping

What's going on here?
---------------------
    Well, from the salt-master we are pushing ping state from the test module to the minion and asking for its response.

    More information on the test module here_.

.. _here: https://docs.saltstack.com/en/latest/ref/modules/all/salt.modules.test.html

salt vs salt-call
-----------------
  salt from the salt-master will **push** a given state to a minion.

  VS.

  salt-call from the minion which will **pull** the state from the master.

----

:data-x: r0
:data-y: r2000

Minion Targeting
================

  *"Targeting minions is specifying which minions should run a command or execute a state by matching against
  hostnames, or system information, or defined groups, or even combinations thereof."*


There are lots of ways to target_ your minions, but the most common method is using ``shell-style``
globbing targeting minions by minion id.

.. _target: https://docs.saltstack.com/en/latest/topics/targeting/#advanced-targeting-methods

        .. code:: bash

          sudo salt '*' test.ping
          sudo salt 'edi*' test.ping
          sudo salt 'vb[1,3]' test.ping
          sudo salt 'ds[1-2][0-9]' test.ping

Or another useful way is using flat lists.

        .. code:: bash

          sudo salt -L 'edi1,edi2,vb1,reports1' test.ping

----

:data-x: r2000
:data-y: r0

States and SLS files
====================

  *"The core of the Salt State system is the SLS, or SaLt State file. The SLS is a representation of the state in
  which a system should be in, and is set up to contain this data in a simple format. This is often called configuration
  management."*

It's just data.
---------------

SLS files are just data representations made up of lists, dictionaries, strings and numbers. Python stuff you're all
used to.

These sls files are compiled together to form a state tree.

A (very small) part of state tree for edi1 looks as follows:
  top.sls
    edi1/init.sls
        batch-cron-dell-asp.sls

----

:data-x: r0
:data-y: r1000

And becasue it's just data.
---------------------------

We can describe about anything. The following ensures nginx is installed, user is present, and the service is running.

  .. code:: yaml

    nginx:
      pkg:
        - installed
      service:
        - running
        - require:
          - pkg: nginx
      user.present:
        - shell: /bin/bash
        - home: /usr/share/nginx
        - uid: 498
        - gid: 499

----

:data-x: r0
:data-y: r1000

Some useful state functions.
----------------------------
  ::

    sudo salt '*' state.highstate

        Run the highstate. Meaning for every minion build and push the entire state tree to matching minions.

    sudo salt 'edi1' state.apply edi1/batch-cron-dell-asp

        Apply only the batch-cron-dell-asp.sls to edi 1. must specify edi1/<state file> becasue salt always starts
        from it's file root which is defined in /etc/salt/master which for us is /srv/salt/base

    sudo salt 'edi1' state.sls_id /home/scripts/dell_asp_exp_status_request.sh edi1/batch-cron-dell-asp

        Run one specific state given a state id and a module.

----

:data-x: r2000
:data-y: r0

States templating, include, and extends.
========================================

Templating
----------

Some sls modules may require programming logic or inline logic. We use the default templaing language jinja_.

.. _jinja: http://jinja.pocoo.org/docs/2.9/

It's very similar to other templating languages, pug, handlebars, etc., where where you have logic constructs and loops.

An example how we use jinja to config phones.

\*In this example, note the use of grains. Grains are objects about the
minion made available to the templating system. We look at id here, but could target on hostname, os type, etc.

    .. code:: yaml

        {% if grains['id'] == 'ds54' %}
        user1:
        sipext: MPA160714170951
        sippass: plivo_password
        corvisa_sipext: 1181
        corvisa_sippass: corvisa_password
        {% endif %}

  Or you could do something like this.

    .. code:: yaml

        {% for usr in ['moe','larry','curly'] %}
        {{ usr }}:
          user.present
        {% endfor %}

----

:data-x: r0
:data-y: r1000

Includes
--------
    A super helpful feature which lets us break state trees into smaller more manageable and modular parts.

    An example include statement for edi1:

      .. code:: yaml

            include:
              - .batch-cron
              - .batch-cron-backoffice
              - .batch-cron-ge
              ...

Extends
-------

  You can also extend previous declarations by using extend. From our previous nginx example...

    .. code:: yaml

      include:
        - prod/webserver/nginx
      extend:
        nginx:
          service:
            - running
            - watch:
              - file: /etc/nginx/servers/*

----

:data-x: r2000
:data-y: r0

Grains
======

  *"Salt comes with an interface to derive information about the underlying system. This is called the grains interface,
  because it presents salt with grains of information. Grains are collected for the operating system, domain name,
  IP address, kernel, OS type, memory, and many other system properties."*

Here you can find more information on grains_.

.. _grains: https://docs.saltstack.com/en/latest/topics/grains/

Common examples.

  ::

    sudo salt 'edi1' grains.ls
    sudo salt 'edi1' grains.get saltversion

The grains are then exposed to the templating language as we saw in our previous example or you can use them to target
specific minions using the -G option.

----

:data-x: r2000
:data-y: r0

And finally salt pillars
========================

  *"Pillar is an interface for Salt designed to offer global values that can be distributed to minions.
  Pillar data is managed in a similar way as the Salt State Tree."*

Pillar are set up by default /srv/pillar.

Some common pillar commands

  ::

    sudo salt 'edi1' pillar.items
    sudo salt 'edi1' pillar.get git
    sudo salt 'edi1' state.apply pillar='{"cheese": "cheddar"}'
