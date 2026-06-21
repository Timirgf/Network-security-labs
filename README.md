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

  ##  What I configured
  
- Created and named VLANs 10, 20, and 30 on SW1
- Assigned access ports to the correct VLAN per PC
- Configured a trunk link between SW1, SW25 and R1 carrying all three VLANs
- Configured three subinterfaces on R1 (one per VLAN), each set as the 
  default gateway for its subnet
- Set static IP addressing on each PC with the correct gateway

## Verification
 Vlan assignment


