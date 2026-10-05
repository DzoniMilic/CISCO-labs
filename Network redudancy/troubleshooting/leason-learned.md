# Lessons Learned

## 1. Default routing alone does not guarantee full connectivity
A default route is sufficient to forward traffic toward unknown destinations, but it does not automatically guarantee that return traffic from upstream devices will follow the correct path.

During this lab, the upstream INET router required explicit static routes for the branch LAN and WAN subnets. Without proper return-path routing, packets could reach the destination but responses could not return successfully.

This highlights the importance of considering **both forward and return paths** when designing and validating WAN connectivity.

---

## 2. Router-originated traffic and host-originated traffic behave differently
A network can appear healthy for end users while router-originated tests behave differently.

Host traffic uses the host’s default gateway and follows the network forwarding path, while router-originated traffic uses the router’s own routing table and source interface selection. Because of this difference, router-based tests may reveal issues that are not immediately visible from end-host connectivity tests.

Understanding this distinction is important when performing network troubleshooting.

---

## 3. Floating static routes provide simple WAN failover
Floating static routes offer a simple and predictable method for implementing WAN redundancy.

By configuring a secondary default route with a higher administrative distance, the router automatically prefers the primary ISP path while keeping the backup path available if the primary link fails.

For small environments or lab scenarios, this approach provides a clear and easy-to-understand failover mechanism without introducing the complexity of dynamic routing protocols.

---

## 4. Static routing has limitations in failure detection
Static routes do not automatically detect failures beyond the directly connected interface.

In this project, upstream routers were unaware of downstream failures unless the interface they were directly connected to went down. This limitation required careful design of return routes to prevent asymmetric routing during failover testing.

In production environments, mechanisms such as:

- dynamic routing protocols
- IP SLA with object tracking
- provider-side routing policies

are often used to detect upstream reachability and make more intelligent failover decisions.

---

## 5. Effective validation requires multiple verification methods
Proper network validation should not rely on a single command or test.

A structured validation approach should include:
- interface status verification
- routing table inspection
- end-to-end ping testing
- traceroute path verification
- failure simulation and recovery testing

Using multiple validation methods helps confirm both connectivity and correct traffic forwarding behavior.

---

## 6. Keeping the lab scope focused improves clarity
The LAN portion of this project was intentionally kept simple so the focus remained on WAN behavior.

This allowed the redundancy design, routing behavior, failover events, and troubleshooting process to be easier to observe and document. Keeping the scope focused helps produce clearer technical explanations and more effective validation documentation.

---

# Lessons Learned good to know

1. Floating static routes react only to local reachability

A floating static route is removed from the routing table only when its own next-hop or exit interface becomes unavailable. R1 considered ISP1 healthy as soon as Gi0/3 was up, without knowing that the link between ISP1 and INET was still down. This is exactly what was observed when R1 selected ISP1 while the ISP1-INET segment was broken.

2. Failover and failback must be checked on both sides of the path

The forward path (R1 default routes) and the return path (INET static routes) are controlled independently. A failback is only complete when both ends agree on the primary ISP. Validating only from R1 can hide a half-restored state.

3. Restore order matters during failback testing

Bringing links back one at a time can produce temporary states that do not exist in the normal or failed design. When testing failback, it is important to know which side has already converged back to the primary path and which has not.

4. Interface-based failover is not Internet-aware

The test showed that a primary link can be "up" while the upstream path is useless. In production, mechanisms such as:

IP SLA with object tracking
dynamic routing protocols
provider-side routing policies

are used to detect real upstream reachability instead of only local link state.

5. Next step: IP SLA with tracking

Planned follow-up test: keep R1 Gi0/3 and ISP1 up, but simulate loss of Internet connectivity beyond ISP1. Then use an IP SLA probe to 8.8.8.8, attach a track object to the primary default route, and confirm that the route is withdrawn and traffic moves to ISP2 automatically.

IP SLA (ping 8.8.8.8) -> track -> primary static route
   probe fails -> primary route removed -> floating static (ISP2) takes over
