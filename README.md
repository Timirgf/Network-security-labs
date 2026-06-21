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
  
<img width="1680" height="1050" alt="Screenshot 2026-06-21 at 3 18 20 PM" src="https://github.com/user-attachments/assets/0942a877-7113-4d99-a63f-1e12d2890193" 

On SW1 (only needs VLAN 10 and 30 - it has no VLAN 20 devices):

SW1#configure terminal
SW1(config)#vlan 10
SW1(config-vlan)#exit
SW1(config)#vlan 30
SW1(config-vlan)#exit


 <img width="1680" height="1050" alt="Screenshot 2026-06-21 at 3 19 12 PM" src="https://github.com/user-attachments/assets/586a6143-5aa9-4ca8-a52e-62111d538d19" />

 <img width="1680" height="1050" alt="Screenshot 2026-06-21 at 3 21 25 PM" src="https://github.com/user-attachments/assets/a04e5bb0-ef37-492f-9e20-a49f0e5f079e" />

 <img width="1680" height="1050" alt="Screenshot 2026-06-21 at 3 22 00 PM" src="https://github.com/user-attachments/assets/06a9b78a-d3de-4ebf-884d-72dc41a05c5f" />






