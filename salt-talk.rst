:data-transition-duration: 1000
:skip-help: false
:auto-console: true
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

Installation and Configuring a minion
=====================================

1. Install with apt-get

  .. code:: bash

    sudo apt-get install salt-minion

2. Configure the minion to talk to the salt master.

  .. code:: bash

    sudo vi /etc/salt/minion

3. Restart salt-minion

  .. code:: bash

    sudo service restart salt-minion

4. Profit

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
    -L                  List all public keys. (Deprecated: use "--list all")
    -a ACCEPT           Accept the specified public key (use --include-all to
                        match rejected keys in addition to pending keys).
                        Globs are supported.
    -A                  Accept all pending keys
    -r REJECT           Reject the specified public key (use --include-all to
                        match accepted keys in addition to pending keys).
                        Globs are supported.
    -p PRINT            Print the specified public key
    -P                  Print all public keys
    -d DELETE           Delete the specified key. Globs are supported.


\* NOTE: You don't have to do this on salt2 (thin client salt master) because auto key accept is turned on. Though, if
rebuild a machine you may have to delete the thin client's public key and have it re-added to the list.

----

So now profit?
==============

Targeting minions
-----------------

  .. code:: bash

    sudo salt 'test1' test.ping

What's going on here?
---------------------
    Well, from the salt-master we are pushing ping state from the test module to the minion and asking for it's response.

    More information on the test module here_.

.. _here: https://docs.saltstack.com/en/latest/ref/modules/all/salt.modules.test.html

salt vs salt-call
-----------------
  salt from the salt-master will **push** a given state to a minion.

  VS.

  salt-call from the minion which will **pull** the state from the master.

----

Minion Targeting (cont'd)
=========================

  *"Targeting minions is specifying which minions should run a command or execute a state by matching against
  hostnames, or system information, or defined groups, or even combinations thereof."*


There are lots of ways to target_ your minions, but the most common method is using `shell-style`
globbing targeting minions by minion id.

.. _target: https://docs.saltstack.com/en/latest/topics/targeting/#advanced-targeting-methods


Ex.

.. code:: bash

  sudo salt '*' test.ping
  sudo salt 'edi*' test.ping
  sudo salt 'vb[1,3]' test.ping
  sudo salt 'ds[1-2][0-9]' test.ping

Or another useful way is using flat lists.

Ex.

.. code:: bash

  sudo salt 'edi1,edi2,vb1,reports1' test.ping

----

States and SLS files
====================

  *"The core of the Salt State system is the SLS, or SaLt State file. The SLS is a representation of the state in
  which a system should be in, and is set up to contain this data in a simple format. This is often called configuration
  management."*

It's just data.
---------------

SLS files are just data representations made up of lists, dictionaries, strings and numbers.

These sls files are compiled together to form a state tree.

A (very small) part of state tree for edi1 looks as follows:
  top.sls
    edi1/init.sls
        batch-cron-dell-asp.sls

----

Um... Ok so what?
=================

Well becasue it's just data. We can describe about anything.

The following ensures nginx is installed, user is present, and the service is running.

  ::

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
