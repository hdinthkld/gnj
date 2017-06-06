*************************************
Segregating Networks using Firewalls
*************************************

.. toctree::
  :titlesonly:
  :maxdepth: 1

* Packet Filter, Stateful and Stateless
* Ports and Protocols
* Static vs Dynamic Group Members

Introduction
============

As a general definition a Firewall is either a physical device or logical
software who's purpose is to to restrict the traffic which is allowed to
flow either into the device/network or to exit the device/network.

A network that doesn't allow any traffic to flow is not a very useful or
practical one, so a need for compromise exists between the organisations
security team wanting absolute security and the people within the organisation
being able to get their work done.

In the past the primary concern was to deploy firewalls on the perimeter of
the network, such as where the private (Trusted) network joins with a public
(Untrusted) network such as the Internet. Traffic was generally permitted
to leave the network but anything trying to come from the public network
into the private network was highly restricted.

As security has matured however there has been a greater emphasis on the need
to also restrict the flow of traffic between even different trusted networks,
such as where an office LAN connects with a critical datacentre environment.

With the greater risk of malware and the use of the network to either
take advantage of unsecured hosts (turning them into zombie hosts) and also
exfiltrate valuable data from the network, organisations are now taking a
more active look at restricted traffic which is leaving the network as well.

In this chapter we will look at the different types of Firewalls and the
advantages/disadvantages of both.  We will also review how to write good
firewall policies so that the organisations goals are met but also minimising
the risk of a security breach.

Network-Based Firewalls
=======================

When a firewall is mentioned, the first impression people think of is that of a
physical device sitting in a server room or data centre and restricting what
applications they can run over the Internet.  This is not the only type of
firewall however.

The description above, is what is referred to as a Network-Based Firewall. Its
role is to filter traffic for an entire LAN or in large scale environments
the organisations entire network. Depending on its position in the network
it either provides the last line of defence (such as when protecting outbound
traffic at the network perimeter) or the first line of defence when defending
against inbound traffic.

A device of this nature is usually a physical device, however more recently
there has been a need for deploying firewalls within a virtualised
infrastructure such as HyperV, VMWare or KVM/Openstack in order to provide
efficient filtering of traffic between different virtual machines.
Examples of this include the Cisco ASAv and Palo Alto VM series.

Most organisations will deploy a purpose built firewall from a major vendor
(such as those mentioned in the last paragraph or Checkpoint) which makes
it easier to meet compliance requirements and to keep up-to-date with
security patches.  These purpose built devices are hardened to only provide
the features needed to filter network traffic and not have unnecessary
software or libraries which the average desktop or server operating system
would have installed and could be exploited.

A large number of the current market of firewalls use some kind of Linux-based
operating system under the hood and it's certainly possible to roll your own
firewall appliance either directly (using tools such as IPTables/NetFilter),
communitity projects also exist which can provide equivilent features to
most of the major vendors for what a small organisation should need (such as
PFSense or Smoothwall Express).

Almost all network devices can be made to do some level of firewalling but
in many cases this is nothing more than a basic traffic filter requiring a
bit more work to ensure the firewalls allow return traffic but still not
offering the same level of security as a dedicated firewall.  An example of this
would be a Cisco IOS router with basic interface ACLs or a Linux device with
the minimal IPTables configuration and no additional modules to maintain
the state of connections.

A basic firewall should be capable of filtering traffic on at least the
following:

* Protocols (e.g. IP, TCP, UDP)
* IP addresses (source and destination)
* Service Port Numbers (source and destination)

These are capabilities of a basic packet filter.


Personal Firewalls
==================

A personal firewall is intended only to defend the host upon which it's running
on.  Whilst this can be cause of much frustration for users and administrators
when the firewalls blocks something they want to use, it's location can be
very beneficial to defending the network and stopping maliscious traffic
from ever going out onto the network.

The personal firewall can be either built into the operating system (such as
Windows Firewall or IPTables) or installed as a seperate piece of software
(such as ZoneAlarm).  Many Anti-Virus vendors provide suites of products that
also include Firewall capabilities and allow all of the features to be
managed from a single interface or even manage multiple hosts from a central
management station.

Because the personal firewall is running on the host, it can gain much greater
detail than that of a network-based firewall.  Usually the firewall software
(either built-in or addon) will have some view it what programs are running
on the host and the traffic they are generating/receiving, this means it can
make decisions based not only the basic packet filtering capabilities above
but also:

* User running the application
* Name of the application/process
* The hash value of the executable on the machine

Using this additional insight can enable an administrator to prevent malicious
applications even getting a single packet on the network.   It can also be
used to permit traffic based on the application where it may use dynamic IP
addressing that can't be written as a traditional packet filter rule.

Next Generation Firewalls
=========================

The current trend is in a next wave of firewalls that are said to offer a number
of "Next Generation" features.  This is nothing new as was seen before in
what used to be referred to as Unified Threat Management (UTM) devices. The
basic idea behind these NGFWs is to offer feature beyond that of
just making decisions based on the protocol, IP addresses and service ports.


The NGFW can also pull in external information that allows it to judge the
reputation of either then sender or destination of the traffic, such as banning
traffic to/from known botnets.  Blocking traffic based on the domain name
being accessed.

They can also do deeper levels of inspection, in the same way as what an
IDS/IPS would do by comparing traffic against known signatures.  Certain
vendors preach that their key benefits are the ability of their platform
to determine the application involved by analysing the stream of data
as it comes through.

The ability to analysis the traffic will of course by hampered by the use of
encryption so many NGFW's also include the ability to  a man-in-the-middle and
break the communication between sender and receiver so they can snoop into
what is being sent.  Where this is used for HTTP/HTTPS traffic, the NGFW is
taking on an additional role that previous was the responsibility of dedicated
web filtering appliances. The NGFW can make decisions based on URL categories
rather than having to deal with specific IP addresses or URLs.


Firewall Methodologies
======================

Packet Filtering
----------------

A packet filtering firewall makes decisions purely based on the most simple of
details in the packet. This is often referred to as the Packet Tuple and includes:

* Protocol
* Source IP
* Destination IP
* Source Port
* Destination Service Port

This type of firewall is available quite cheaply and most home broadband
routers will include these basic features.  It does however suffer from a number
of weakernesses

* Only considers the basic details above so is subject to IP spoofing
* No session information is maintained therefore traffic must be permitted
  in each direction regardless of if an active flow is currently maintained.
* Cannot handle applications with dynamic ports, the rules must be open all
  the time even if not currently neeeded.

Application Layer Gateway
--------------------------

Sometimes also called a Proxy Firewall or Application Gateway. These devices
are able to operate at OSI levels 3 and above and included specilised software
which enables them to decode the application traffic.

Often acting as an intermediate devie so there is no direct communication
between client and server.

Because the ALG can look all the way up the network stack it can make very
fine grained decisions. The disadvantage of this is that the ALG must also
be written to know how to handle the application and cannot understand that
which it hasn't been programmed to process.

The example of the Web filtering used in NGFW's would be an example of an ALG,
also in older firewalls the ability for them to handle dynamic ports used
by either FTP or SIP traffic is also the realm of an ALG to handle.

Older Cisco firewalls offer basic ALG features as described above where as
Palo Alto tout one of the main features of their products is the ability for
them to understand hundreds (if not thousands) of different applications and
make their policy decisions based upon the appliation detected.


Stateful Packet Filtering
-------------------------

Almost all modern firewalls are at least capable of performing Stateful Packet
Filtering. The key difference between a basic packet filter and stateful packet
filter is that the firewall remembers what has occured in the session up to that
point.

In the case of TCP sessions for example, a firewall is aware of if the 3-Way
handshake has occured and in what order.  If a packet is marked with incorrect
flags the firewall can choose to drop the packet as an invalid selection.

More advanced stateful firewalls will also maintain additional details such as
sequence numbering so as to provide anti-replay detection.

As covered under the Application Layer Gateway section, a stateful firewall
can also look deeper into the packets and discern additional detail (such as
FTP port commands to open dynamic connections).  Unlike the ALG it does this
without acting as a proxy so is more limited and is further constrained where
any kind of encryption is involved.

Transparent Firewalls
---------------------

Most firewalls are implemented so that traffic needs to be routed to them
(such as using them as the default gateway on a LAN). This however can mean
quite substantial changes if the network curretny does not have any firewalling
in place.

The solution is to implement a Transparent Firewall.  This differents from a
typical firewall deployment in that no changes are needed on the network as
the firewall is not assigned an IP address (except perhaps for management). By
doing this a firewall can be deployed into an existing network in order to
segregate hosts or subnets.

In the case of a network that never had traffic filtering applied.  The policy
on the transparent firewall can be slowly built up until all traffic is
accounted for, at which point for more substantial change can be planned to
move to routed firewall model.

Firewall Polcy Management
=========================

Policy Enforcement
------------------

A firewall acts as an enforcement point in the network for the policy as
dictated by the organisations management. It is important that the firewall
is deployed in such a way as to secure the network but not limit the ability
of users to go about there daily tasks.

One of the most common tasks for a Firewall Administrator is to maintain the
rulebase on the firewall so that they are meeting the security needs of the
organisation yet still not allowing anything inbound or outbound that has
not been authorised.

Determing Policy
----------------

Whenever the firewall admin is planning to make a change to the existing
configuration he needs to ask himself the following questions:

#. Is the request reasonable and not too wide reaching?
#. Is there any existing policy which either allow or prevents me implementing
   this request?
#. Is all the information provided correct?
#. How can ensure that the implemented policy is managable?
#. What additional information do I need to ensure the changes can be
   suitably audited in the future?

Many users will request that changes are made to the firewall without fully
understanding what the issue is.  Even technical users (e.g. developers) fall
into this category and will often request the rules be placed on the firewall
which really don't need to be as unrestricted as they think. It is the role
of the firewall administrator to make this judgement call and only puts
rules on the firewall which meet the requirement and do not provide any more
access than is absolutely necessary.

All of the changes the firewall admin make to the firewall must have a precedent
in the organisations security policy.  For instance, if the security policy
states that telnet is not allowed then the firewall admin should not make
any change which permits this without checking with either the security or
management team. They can then make the decision either to deny it or to update
the security policy accordingly.

Often the request received from the user will be very vague simplying being
"I need to access Application X".  It is the role of the firewall admin to
gather further information from the user/requester so that it can be suitable
implemented on the device. At the very least the 5-tuple information should
be known so that further validation can be done.

It's all well and good to know the technical details to implement the firewall
rule but it is needs to be validated that this is correct and covers all the
requirements going forward.  Too many times a new rule will be implemented only
for a seperate (seemly unrelated) request to come in the following day which
results in a second rule being added for the same thing.  This is how a policy
which is only really handing 10-15 different applications ends up having
hundreds if not thousands of seperate rules.

The solution to this is to gather as much information as possible so that the
rules can be implemented in a way which allows easier identification and
possibly grouping of related rules together.  Most vendors will provide the
following features to make the entire firewall policy more managable:

* Named IP addresses and Subnets
* Named Applications and Services
* Grouping of related IP addresses,Networks, Service Ports and Application
  so many rules can be implemented as one (or fewer rules).

When changes are made to a firewall, larger organisations often require that
changes are approved before being made.  The process for doing this can range
from a simple request to another colleague to a full on meeting, often called
the Change Approval Board (CAB) whereby all changes are reviewed and changes
have to be planned for a certain date and time.  Either way this information
should be recorded in the firewall policy documentation and preferably on the
firewall itself so that any changes made can be linked with the approval.

A vendors firewall information should include an area to put a comment on a rule
so that a (potentially non-technical) auditor can review the policy and ensure
it is meeting the security requirements without any approved changes. An example
of a good comment could be:

*"20170220-CR12345-JB: Remote Management of Hosted Azure Servers"*

The above comment includes a freeform text to give a non-technical description
of why the rule exists along with when it was applied, the Change Reference and
the engineer who applied the change.  Information that could potentially be
gained elsewhere (such as who requested the rule or the date of the request)
should be recorded elsewhere (such as in the organisations ticketing system) and
it isn't something that would be needed at quick glance.

An regular firewall review may be take place to ensure that all rules are still
needed and this is where the requestor information is useful but it is not
something that would be helpful when troubleshooting an issue.
