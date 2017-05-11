:data-transition-duration: 1000
:skip-help: false
:auto-console: true

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
  * sudo salt-key list - list all accepted, rejected, denied, and unaccepted keys.
  * sudo salt-key -A - accept all unaccepted keys.
  * sudo salt-key -a <key> - accept <key>.
  * sudo salt-key -r <reject> - reject <reject>.
  * sudo salt-key -d <delete> - delete the public key of <delete>

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

    More information on the test module here https://docs.saltstack.com/en/latest/ref/modules/all/salt.modules.test.html

salt vs salt-call
-----------------
  salt from the salt-master will **push** a given state to a minion.

  VS.

  salt-call from the minion which will **pull** the state from the master.

---

Minion Targeting (cont'd)
=========================

  *"Targeting minions is specifying which minions should run a command or execute a state by matching against
  hostnames, or system information, or defined groups, or even combinations thereof."*

  There's lots of ways to target your minions, https://docs.saltstack.com/en/latest/topics/targeting/#advanced-targeting-methods

  But, the most common method is using shell-style globbing targeting minions by minion id.

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
