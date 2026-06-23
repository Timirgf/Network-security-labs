##  Network-Security-Labs
##  Lab 1: VLAN Segmentation, Trunking & Inter-VLAN Routing

## Objective
Segment a network into three VLANs by department (Engineering, HR, Sales), 
each on a /26 subnet, and enable inter-VLAN communication using 
router-on-a-stick with trunking between R1 and SW1.

## Topology
<img width="1648" height="368" alt="Screenshot 2026-06-19 at 7 33 15 PM" src="https://github.com/user-attachments/assets/9495f795-28e1-47ac-8757-a474e4119576" />
- VLAN 10 (Engineering): 10.0.0.0/26

- VLAN 20 (HR): 10.0.0.64/26

- VLAN 30 (Sales): 10.0.0.128/26

- Gateway for each VLAN = last usable IP in the subnet
- 

  ##  What I configured
  
- Created and named VLANs 10, 20, and 30 on SW1
- Assigned access ports to the correct VLAN per PC
- Configured a trunk link between SW1, SW25 and R1 carrying all three VLANs
- Configured three subinterfaces on R1 (one per VLAN), each set as the 
  default gateway for its subnet
- Set static IP addressing on each PC with the correct gateway


  Summary

- SW1 (access layer): VLAN 10 - PC1, PC2 (F0/1, F0/2) | VLAN 30 - PC3, PC4 (F0/3, F0/4)
- SW1 <-> SW2: trunk link (Gig0/1 on both sides)
- SW2 (access layer): VLAN 10 - PC6, PC7 | VLAN 20 - PC5 (F0/1)
- SW2 <-> R1: trunk link (Gig0/2 on SW2), router-on-a-stick


## Verification
   Vlan assignment

## On SW1 (only needs VLAN 10 and 30 - it has no VLAN 20 devices):

SW1configure terminal

SW1(config)vlan 10

SW1(config-vlan)exit

SW1(config)vlan 30

SW1(config-vlan)exit
WHY VLAN 30 must exist on SW2 even though SW2 has no VLAN 30 PCs:
A switch will only forward a VLAN's traffic across a trunk if that VLAN
exists in its local VLAN database. Since VLAN 30 traffic needs to pass
THROUGH SW2 to reach R1 (for routing), SW2 needs VLAN 30 in its database.


## STEP 2: CONFIGURE ACCESS PORTS (PCs)
--------------------------------------

On SW1:

SW1(config)interface fa0/1

SW1(config-if)switchport mode access

SW1(config-if)switchport access vlan 10

SW1(config-if)exit

SW1(config)interface fa0/2

SW1(config-if)switchport mode access

SW1(config-if)switchport access vlan 10

SW1(config-if)exit

SW1(config)interface fa0/3

SW1(config-if)switchport mode access

SW1(config-if)switchport access vlan 30

SW1(config-if)exit

SW1(config)interface fa0/4

SW1(config-if)switchport mode access

SW1(config-if)switchport access vlan 30

SW1(config-if)exit

On SW2 (same pattern - F0/1 -> VLAN 20, F0/2 & F0/3 -> VLAN 10):

SW2(config)interface fa0/1

SW2(config-if)switchport mode access

SW2(config-if)switchport access vlan 20

SW2(config-if)exit

SW2(config)interface fa0/2

SW2(config-if)switchport mode access

SW2(config-if)#switchport access vlan 10

SW2(config-if)#exit

SW2(config)#interface fa0/3

SW2(config-if)#switchport mode access

SW2(config-if)#switchport access vlan 10

SW2(config-if)#exit

## WHAT THE COMMANDS DO:
- "switchport mode access" locks the port as a non-trunking port.
- "switchport access vlan X" assigns it to that VLAN's broadcast domain.


## STEP 3: CONFIGURE THE SW1 <-> SW2 TRUNK
------------------------------------------

On SW1, Gig0/1:

SW1(config)#interface gigabitEthernet 0/1

SW1(config-if)#switchport trunk encapsulation dot1q

SW1(config-if)#switchport mode trunk

SW1(config-if)#switchport trunk native vlan 1001

SW1(config-if)#switchport trunk allowed vlan 10,30

SW1(config-if)#exit

On SW2, Gig0/1 (must match SW1):

SW2(config)#interface gigabitEthernet 0/1

SW2(config-if)#switchport trunk encapsulation dot1q

SW2(config-if)#switchport mode trunk

SW2(config-if)#switchport trunk native vlan 1001

SW2(config-if)#switchport trunk allowed vlan 10,30

SW2(config-if)#exit

WHY VLAN 20 IS EXCLUDED HERE:
This trunk's only job is carrying traffic between SW1 and SW2. VLAN 20
has zero devices on SW1 - PC5 (the only VLAN 20 host) lives entirely on
SW2. There's no VLAN 20 traffic that ever needs to cross this particular
link, so allowing it would just be wasted trunk overhead (and bad
practice - trunks should only carry VLANs they actually need, per the
lab instruction "allowing only the necessary VLANs"). Restricting
allowed VLANs on a trunk is also a security best practice - it limits
the blast radius if something goes wrong on that link.


## STEP 4: CONFIGURE THE SW2 <-> R1 TRUNK
------------------------------------------

On SW2, Gig0/2:

SW2(config)#interface gigabitEthernet 0/2

SW2(config-if)#switchport trunk encapsulation dot1q

SW2(config-if)#switchport mode trunk

SW2(config-if)#switchport trunk native vlan 1001

SW2(config-if)#switchport trunk allowed vlan 10,20,30

SW2(config-if)#exit

WHY VLAN 20 IS INCLUDED HERE:
This trunk's job is different - it connects SW2 to the router, which is
what makes inter-VLAN routing possible. For PCs on VLAN 20 to talk to
PCs on VLAN 10 or VLAN 30 (or vice versa), their traffic has to
physically reach R1 to be routed between subnets. So this link needs
every VLAN that requires routing - which is all three. This matches the
verification output:
   Gig0/2   10,20,30
   Gig0/1   10,30


## STEP 5: CONFIGURE ROUTER-ON-A-STICK ON R1
--------------------------------------------

R1(config)interface gigabitEthernet 0/2
R1(config-if)no shutdown
R1(config-if)exit

R1(config)interface gigabitEthernet 0/2.10
R1(config-subif)encapsulation dot1Q 10
R1(config-subif)ip address 10.0.0.62 255.255.255.192
R1(config-subif)exit

R1(config)interface gigabitEthernet 0/2.20
R1(config-subif)encapsulation dot1Q 20
R1(config-subif)ip address 10.0.0.126 255.255.255.192
R1(config-subif)exit

R1(config)interface gigabitEthernet 0/2.30
R1(config-subif)encapsulation dot1Q 30
R1(config-subif)ip address 10.0.0.190 255.255.255.192
R1(config-subif)exit

NOTE: IPs above are the last usable address in each /26 per the lab
instructions - confirm against actual subnet math for your topology.

## WHAT THE COMMANDS DO:
- "encapsulation dot1Q 10" tells that subinterface "I only handle
  tagged VLAN 10 traffic."
- The physical interface itself stays untagged/admin-up via
  "no shutdown," while each .10/.20/.30 subinterface acts as the
  gateway for its respective VLAN.


## STEP 6: VERIFY
----------------

show vlan brief                  -> confirms VLAN existence + port membership
show interfaces trunk            -> confirms trunk status, allowed VLANs, native VLAN
show ip interface brief (on R1)  -> confirms subinterfaces are up/up with correct IPs
ping <gateway>                   -> from each PC, confirm reachability







<img width="1680" height="1050" alt="Screenshot 2026-06-21 at 3 18 20 PM" src="https://github.com/user-attachments/assets/0942a877-7113-4d99-a63f-1e12d2890193"/>
 On Switch 1 Vlan 10 was assigned to F0/1 and F0/2 and 20 was assigned to  F0/3 and F0/4
 <br><br>
 <img width="1680" height="1050" alt="Screenshot 2026-06-21 at 3 19 12 PM" src="https://github.com/user-attachments/assets/586a6143-5aa9-4ca8-a52e-62111d538d19" />
 do sh trunk int trunk

To show all trunk port on the switch and this displays

Native Vlan

Allowed Vlan

Vlans active on the trunk

 <br><br>

 <img width="1680" height="1050" alt="Screenshot 2026-06-21 at 3 21 25 PM" src="https://github.com/user-attachments/assets/a04e5bb0-ef37-492f-9e20-a49f0e5f079e" />
On Switch 2 Vlan 10 was assigned to F0/3 and F0/2 and 20 was assigned to  F0/1 
 <br><br>
 <br><br>

 <img width="1680" height="1050" alt="Screenshot 2026-06-21 at 3 22 00 PM" src="https://github.com/user-attachments/assets/06a9b78a-d3de-4ebf-884d-72dc41a05c5f" />
do sh trunk int trunk

To show all trunk port on the switch and this displays

Native Vlan

Allowed Vlan

Vlans active on the trunk

<br><br>
<img width="1680" height="1050" alt="Screenshot 2026-06-21 at 4 00 47 PM" src="https://github.com/user-attachments/assets/b755e7a7-c46c-4d3d-b8dc-0aaf1e7aa5ad" />

Pinging 10.0.0.4 from 10.0.0.126/26 and it worked, meaning all configuration was sucessful.





