---
name: transitgateway
description: >-
  Configures AWS Transit Gateway: creating a hub and attaching VPCs, segmenting traffic with route
  tables, centralizing egress and inspection through a hub (appliances or a Gateway Load Balancer
  endpoint), forcing east-west traffic between VPCs through AWS Network Firewall, connecting
  on-premises networks over the transit-gateway side of a Site-to-Site VPN or Direct Connect
  attachment (including ECMP to aggregate bandwidth across multiple VPN tunnels), peering transit
  gateways across Regions, migrating from a VPC peering mesh, and routing IP multicast. Applicable
  when connecting many VPCs through one router, isolating environments, forcing VPC-to-VPC traffic
  through a central Network Firewall, reaching on-premises over the hub, linking Regions, or moving
  off a peering mesh. Not applicable for single-VPC routing, VPC peering between two VPCs (vpcpeering
  skill), Direct Connect gateway or virtual interface setup (directconnect skill), or Route 53 DNS
  work.
---