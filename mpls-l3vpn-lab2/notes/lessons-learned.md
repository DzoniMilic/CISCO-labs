## Lessons Learned

### OSPF and Routing

- OSPF provides IP reachability across the provider core.
- An OSPF neighbor reaching `FULL` does not automatically guarantee that every expected route is installed in the routing table.
- Verify routing with `show ip route <destination>` rather than relying only on the neighbor state.
- Static routes can hide problems with dynamic routing. Removing an unnecessary static route helped verify that the PE2 loopback was learned through OSPF.
- The OSPF database, routing table, and forwarding table represent different stages of route processing.

### MPLS and LDP

- OSPF provides the IP reachability required for LDP to establish sessions between routers.
- LDP distributes labels used to forward traffic through the MPLS core.
- `mpls ldp router-id Loopback1 force` selects the loopback address as the LDP router ID and forces the change when necessary.
- `mpls ldp autoconfig` under OSPF enables LDP on OSPF-enabled interfaces.
- `show mpls ldp neighbor` verifies LDP session state, while `show mpls forwarding-table` displays label-forwarding entries.
- A route to a remote PE loopback is essential for establishing LDP connectivity to that PE.

### VRF and MP-BGP VPNv4

- A VRF separates customer routing information from the global routing table.
- The Route Distinguisher (RD) makes VPN routes unique in the VPNv4 address space.
- Route Targets (RTs) control which VPN routes a VRF imports and exports.
- MP-BGP VPNv4 carries VPN routes between PE routers; OSPF and LDP serve different purposes in the provider core.
- A route can exist in the VPNv4 BGP table without necessarily being installed in the customer's VRF routing table.
- Verify customer routes in the correct VRF rather than checking only the global routing table.

### Forwarding and Troubleshooting

- A route in the routing table does not by itself prove that traffic will be forwarded successfully.
- CEF and MPLS forwarding entries help verify the actual forwarding path and label operations.
- Successful end-to-end pings in both directions provide practical confirmation of customer connectivity.
- When troubleshooting, isolate the layers in order: IP reachability, OSPF, LDP/MPLS, VRF routes, MP-BGP VPNv4, and end-to-end connectivity.

### Final Verification

- Verify OSPF neighbor relationships and expected core routes.
- Verify LDP sessions and MPLS forwarding entries.
- Verify the customer VRF, its routing table, and VPNv4 route exchange.
- Test CE1-to-CE2 and CE2-to-CE1 connectivity and confirm successful ICMP replies.
