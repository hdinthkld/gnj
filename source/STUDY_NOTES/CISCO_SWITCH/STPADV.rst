***************************************
Cisco - Advanced Spanning Tree Protocol
***************************************

Rapid Spanning Tree Protocol (RSTP)
===================================

- Traditional STP is considered too slow for modern networks
- RSTP is defined in IEEE 802.1w
- When combined with PVST+ it is called RPVST+
- Used as part of MST (IEEE 802.1s)
- Allows switches to interact through each port to gain quicker convergence times
- Uses same election process as traditional STP (IEEE 802.1d)
- Different Names are used for the port roles than in STP

  * Root Port - Has the best root path cost, won't exist on the root bridge
  * Designated Port - One per segment, best port to reach root on each segment
  * Alternate Port - The next-best path to the root bridge
  * Backup Port - Redudant connection on each segment

- Different port states are also used than in STP

  * Discarding - Combines the STP disabled, Blocking and Listening states
  * Learning   - Frames dropped, MAC adresses learned
  * Forwarding - Full Operation

BPDUs in RSTP
-------------

- Uses 802.1D BPDU format
- Sending switches identify by role and state
- BPDU version is set to 2
- Neighbouring switches negotiate using BPDU flags
- Sent every hello interface out all switch ports
- 3 lost BPDUs (6 seconds) mark the neighbour as down causing information to be aged out
- Can operate with traditional STP based on received BPDU version
- BPDU version is help until "Migration Delay Timer" expires

RSTP Convergence
----------------

- Two stages

  * Common Root bridge is elected
  * Switch port states assigned to build loop free topology

- Upon first joining network, RSTP bases it's forwarding decision on port type
- RSTP Port Types

  * Edge Port - Single host, portfast configure port
  * Root Port - Best Cost to Root bridge
  * Point-To-Point port - Any switch connecting to another switch and becomes designated port

- Ports running half-duplex cannot be point-to-point, traditional STP (802.1d) convergence used
  on these ports
- Complete convergence is done via propagation of handshakes over point-to-point links 
- After each successful decision, the handshake moves to the next switch towards the edge of the network
- During handshaking, loops are avoided via a synchronisation process

RSTP Synchronisation
--------------------

- Non-edge ports begin in discarding state
- After root bridge identifed thhrough BPDU exhcnage, port receiving support BPDU becomes root port
- Each switch assumes its port should be designated for the segment. Proposal sent to neighbouring switch
- If proposal is superior, synchronisation occurs

  * All non-edge ports put into discarding state
  * Aggrement message sent in reply
  * Root port moved to the forwarding state
  * On each non-edge port currently discarding, a proposal is sent to the next neighbour switch
  * Aggrement message received
  * Non-Edge port moved to forwarding state

- No timers are used
- If no agreement is received, falls back to 802.1d rules

Topology Changes and RSTP
-------------------------

- Detected only when non-edge port transitions to forwarding state
- Used only so briding tables are updates
- Topology Change (TC) messages propagate through the the network
- MAC Addresses flushed from the CAM table

Rapid Spanning Tree Configuration
---------------------------------

**Set a port as an RSTP edge port**

::

  interface <name>
    spanning-tree portfast

**Force a port to act as a point-to-point port**

::

  interface <name>
    spanning-tree link-type point-to-point

**Enable use of RPVST+**

::

  spanning-tree mode rapid-pvst

**Revert To PVST+**

::

  spanning-tree mode pvst

**Verify STP mode used by Switch and neighbours**

::

  show spanning-tree [vlan <id>]


Multiple Spanning Tree (MST) Protocol
=====================================

Overview
--------

- Allow for better management oof switch resources whilst still permitting load balancing over redudant links
- One or more VLANs are mapped to a single STP instance
- Multiple instances of STP

Implementing MST
----------------

- Determine numbber of STP instances
- Map a set of VLANs to each instance

MST Regions
-----------

- Defined by a set of attributes

  * Configuration name (32 characters)
  * Revision number (0 - 65535)
  * Instance to VLAN Mapping (0 - 4096 entries)

- Two switches with same attributes are in the same MST region
- MST BPDUs contain the attributes
- Region boundaries are defined where attributes differ or 802.1d (STP) is used
- Mapping of VLANs must be configured on each switch as they are not sent in the BPDU

Spanning-Tree Instances within MST
----------------------------------

- MST interoperates with all forms of STP
- MST treats the entire network as a single Common Spanning Tree (CST) topology
- CST regards each MST instance as a "Black Box" with no idea what happens inside
- CST maintains a loop free topology with links connecting outside the region

Internal Spanning Tree (IST) Regions
------------------------------------

- An IST instance uns to calculate a loop free topology within an MST region
- IST presents the entire region as a single bridge to the CST outside
- BPDUs exchanged at BPDU boundaries only over native VLAN trunks

MST Instances (MSTI)
--------------------

- Maximum of 15 MSTIs (0-15)
- MSTI 0 (zero) is reserved for the IST
- Instances define the VLAN mapping for each topology
- Only the IST (MSTI 0) is permitted to send BPDUs
- M-Records are added to MST BPDU's for each MSTI
- A single BPDU is needed regardless of number of instances
- If a BPDU is detected on more than one VLAN, MST assumes PVST+ is used so the BPDU is replicated
  to all VLANs on the trunk link.

MST Configuration
-----------------

**Enable MST on a Switch**

::

  spanning-tree mode mst

**Enter MST configuration mode and define attributes**

::

  spanning-tree mst configuration
    name <region-name>
    revision <0-65535>
    instance <number> vlan <vlan-list>

**Show pending MST configuration changes**

::

  show pending

**Commit outstanding changes from MST configuration mode**

::

  exit

**Set The MST root bridge via a macro**

::

  spanning-tree mst <instance> root {primary | secondary} [diameter <number>]

**Manually set MST Bridge Priority**

::

  spannning-tree mst <instance> priority <number>

**Set MST port cost**

::

  interface <name>
    spanning-tree mst <instance> port-priority <number>

**Set MST timers**

::

  spanning-tree mst hello-time <seconds>
  spanning-tree mst forward-time <seconds>
  spanning-tree mst <max-age> <seconds>

