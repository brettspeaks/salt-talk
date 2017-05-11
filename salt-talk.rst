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
=========
  *"Salt-key executes simple management of Salt server public keys used for authentication."*

It is basically salt's way of authenticating a minion. On inital connection, the minion will send the salt-master its
public key so on the salt master we have to manually accept/reject the minion's key.\*

\* NOTE: You don't have to do this on salt2 (thin client salt master) because auto key accept is turned on. Though, if
rebuild a machine you may have to delete the thin client's public key and have it re-added to the list.
----
