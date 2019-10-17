*****************************
Cisco - Managing Switch Users
*****************************

.. _switch_aaa:

AAA Overview
============

- Used to manage user activity by controlling and reporting on their actions
- AAA Functions

  * Authentication - Who is the user?
  * Authorization - What is the user allowed to do?
  * Accounting - What did the user do?

- After successfull login, users are given "User Exec" level privileges
- An "Enable" secret is entered which grants additional privileges
- Local usernames can be configured on the switch, not scaleable
- Centralised authentication can be done through TACAC+ or RADIUS

Centralised Authentication
--------------------------

- Client/Server model
- Switches as clients known as "Network Access Device" (NAD) or 
  "Network Access Server" (NAS)
- Cisco provides AAA services through two products

  * Identity Service Engine (ISE)
  * Secure Access Control Server (ACS)

.. _switch_aaa_tacacs:

Terminal Access Controller Access Control System (TACACS+)
----------------------------------------------------------

- Cisco Proprietary
- Uses TCP Port 49 for secure/encrypted communication

.. _switch_aaa_radius:

Remote Access Dial-In User Service (RADIUS)
-------------------------------------------

- Standards Based
- Uses UDP Ports 1812 (Authentication) and 1813 (Accounting)
- Not all traffic is encrypted

.. _switch_aaa_methods:

Method Lists
============

- Used to group vvarius authentication/authorisation methods for easier reuse
- Authentication methods

  * TACACS+ - Try each server defined until success or positive denial
  * RADIUS  - Try each server defined until success or positive denial
  * LOCAL   - Check the entered details against configured users on the local switch
  * LINE    - Authenticate against password defined on VTP/console, does not use a username

- Authorization Methods

  * Commands - Server must give permit for any command at any privilege level
  * Config-Commands - Server must give permission for config commands
  * Configuration - Server must vie permisson to enter config mode
  * Exec - Server must give permission to run exec session
  * Network - Server must return permission to use network related sessions
  * Reverse-Access - Server must give permission for reverse telnet sessions

- Only TACACS+ supports per-command authorisation, RADIUS is all or nothing

Accounting
==========

- Records activities performed by a user
- Supported by RADIUS and TACACS+
- Accounting Levels

  * System - Major swich events (E.g. reload)
  * Exec - User authenticated into an Exec session (IP, Time and duration are recorded)
  * Commands - Info on commands running at the specified level
  
- Accounting  Times

  * Start-Stop - Events recorded at both beginning and end
  * Stop-Only - Events recorded at end of action
  * none - No events recorded

Configuring Authentication
==========================

**Enable AAA on the Switch**

::

  aaa new-model

**Create Local Users**

::

  username <username> password <password>

**Define RADIUS server**

::

  radius-server host {<hostname>|<ip>} [key <string>]

**Define TACACS+ server**

::

  tacacs-server host {<hostname>|<ip>} [key <string>]

**Define Server Groups and Included Servers**

::

  aaa group server {radius|tacacs+} <group-name>
    server <ip>

**Define Method List For Authentiation**

::

  aaa authentiation login {default | <list-name>} <method-1> [<method-x> ...]


**Apply the method list to a switch line**

::

  line {vty|con}<number>
    login authentication {default|<list-name>}

Configure Authorisation
=======================

**Define Authorization Method List**

::

  aaa authorization {commands|config-commands|configuration|exec|network|reverse-access} 
                    {default|<list-name>} <method-1> [<method-x> ...]

**Apply method list too specific line**

::

  line {vty|con}<number>
     login authorization {default|<list-name>}

Configure Accounting
====================

**Define Accounting Method List**

::

  aaa accounting {system|exec|commands <level>} {default|<list-name>}
                 {start-stop|stop-only|wait-start|none} <method> [<method-x> ...]

**Apply method list to required lines**

::

  line {vty|con}<number>
    login accounting {default|<list-name>}
