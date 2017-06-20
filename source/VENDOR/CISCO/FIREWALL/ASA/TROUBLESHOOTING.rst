#########################
Cisco ASA Troubleshooting
#########################

Packet Captures
===============

The packet capture process is useful when you troubleshoot connectivity problems
or monitor suspicious activity. In addition, you can create multiple captures in
order to analyze different types of traffic on multiple interfaces.

The packet capture feature can be used to capture packets as the enter an
interface (Ingress) as well as when they leave (Egress).

If both are runningit's possible that duplicate packets may be captured as each
capture on an interface occurs seperately.

When performing captures on the Egress interface, it is important to take NAT
into account.

Default Settings
----------------

The default type is raw-data.

The default buffer size is 512 KB

The default Ethernet type is IP packets.

The default packet-length is 1,518 bytes (68 bytes for circular buffe)

Running Packet Captures through ASDM
------------------------------------

Once logged into the ASDM GUI, the Packet Capture Wizard can be run from:

.. menuselection:: `Wizarrd --> Packet Capture Wizard`

Using the wizard it is possible to capture both source traffic (Ingress) and
destination traffic (Egress).  when specifying the packet match criteria it
is important to take NAT into consideration.

Both capture buffer size and packet size can be specified.

At the end of the wizard, ASDM will send a temporary set of commands that will
start the capture, possibly including an ACL. It is then possible to retrieve
the captures through the GUI and also to save the capture as either a PCAP or
ASCII file.

Running Packet Captures through the CLI
---------------------------------------

The `capture` command is used to run a packet capture from a CLI.

Originally it was required for an ACL to be included along with the capture
command.  This is still supported but as of 7.2(1) it's possible to use a
simpler `match` statement. The ACL method is still preferred where more complex
(such as multiple source/destination) captures are needed.

To create a capture using a match statement use this syntax:

.. code-block:: none

  capture <cap-name> match ip <criteria>

Upto 3 match criteria can be specified per line

To capture using an access-list use this syntax:

.. code-block:: none

  capture <cap-name> access-list <acl-name>

The access-list must already exist

Once the criteria has been set, specify the interface upon which the capture
should be performed:

.. code-block:: none

  capture <cap-name> interface <ifname>

Other options, such as buffer size and packet size are also available.

It is also possible to capture packets that would normally be dropped. To
do this specify a type of `asp-drop`. Other special types are available.

Obtaining the captured packets through a Web Browser
----------------------------------------------------

Once packets have been captured the file can be downloaded as either ASCII or
PCAP using one of the following URLS:

.. rubric:: ASCII File

https://<ip-of-asa>/admin/capture/<capture-name>

.. rubric:: PCAP File

https://<ip-of-asa>/admin/capture/<capture-name>/pcap

If the ASA is running in multiple context mode, admin would be replaced by
the name of the context from which the capture was obtained.

Further Information
===================

For more detailed information consult the Cisco configuration examples and
tech notes [c1]_.
