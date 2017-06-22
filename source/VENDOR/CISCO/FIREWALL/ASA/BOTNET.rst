.. _cisco_asa_botnet:

##########################
Cisco ASA BotNet Filtering
##########################

Overview
########

The Cisco ASA Botnet filter provides a means to detect and potentially block
traffic heading to known Botnet destinations.

It can do this by using both known IP addresses as well as by monitoring DNS
traffic for known bad DNS queries.

It has a number of pre-requisites:

* Seperate BotNnet (subscription) license required
* ASA configured to use DNS in order to lookup names and address in static
  database
* DNS Snooping enabled
* Live connectivity to the Internet so that latest database an be downloaded.

Implementation
##############

In order to implement the Botnet traffic filter the license must be enabled.
This can be checked via the *show version* command on the CLI.

If the license is not installed, the configuration options will not be shown
in ASDM but the commands will still be available via the CLI.

Configuration Steps
-------------------

#. Configure dynamic database
#. Configure the static database
#. Enable DNS snooping
#. Enable the Botnet traffic filter

.. rubric:: Configure the dynamic database

Via ASDM the database can be configured through:

:menuselection:`Configuration --> Firewall --> Botnet Traffic Filter`

Select the following options:

* Enable Botnet Updater Client
* Use Botnet data dynamically downloaded from updater server

After clicking :guilabel:`Apply` the database will be automatically downloaded the first
time.  To manually download the latest database select
:guilabel:`Fetch Botnet Database`.

Or via the CLI:

.. code-block:: none

  dynamic-filter updater client enable
  dynamic-filter use database

.. rubric:: Configure the static ddatabase

Under :guilabel:`Botnet Traffic Filter` select :guilabel:`Black and White Lists`

Any entries in the whitelist will be allowed and not checked against the
database or static entries. Entries in the blacklist will be used in conjunction
with the database and monitored by the Botnet filter.

Or via the CLI:

.. code-block:: none

  dynamic-filter { blacklist | whitelist }
    {name <hostname> | address <ip>}


.. rubric:: Enable DNS Snooping

Under :guilabel:`Botnet Traffic Filter` select :guilabel:`DNS Snooping`.

For global DNS Snooping, simply check the :guilabel:`DNS Snooping Enabled`
option under the :guilabel:`global` interface.

Or via the CLI:

.. code-block:: none

  ! By default the global policy is called 'global_policy'
  policy map <policy-name>
    class inspection_default
      inspect dns preset_dns_map dynamic filter snoop


.. rubric:: Enable the Botnet Traffic Filter

Under :guilabel:`Bonet Traffic Filter` select :guilabel:`Traffic Settings`.

In most situations it will be necessary to enable the Botnet filter on the
external facing interface. Check the option for :guilabel:`Traffic Classified`
on the appropriate interface.

It is also possible to only filter specific traffic, this can be done by
selecting :guilabel:`Manage ACL` and defining the appropriate traffic.  It's
also possible to specific what level of traffic will be dropped.

.. note:: The default traffic level to be dropped is moderate and above with
          all traffic being dropped.

Or via the CLI:

.. code-block:: none

  dynamic-filter enable [ interface <name>] [classify list <acl-name>]

  ! Optionally - Drop connections involved in botnet activity
  dynamic-filter drop blacklist [interface <name>] [action-clasify-list <acl>] [threat-level {eq <level> | range <min-level> <max-level>}]
