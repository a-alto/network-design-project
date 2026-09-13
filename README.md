# Network Infrastructures — Routing, VPNs & SSH Tunneling

Project for the **Network Infrastructures course at Sapienza University of Rome**, academic year **2025–2026**.

The project focuses on reproducing a multi-router network in **Kathara** and integrating IP addressing, routing, network services, and security controls. Its scope combines **static and dynamic routing, NAT, stateful firewalling, certificate-based VPN connectivity, SSH tunneling, and packet capture** in a single container-based lab.

The original topology and detailed requirements are provided in the [assignment specification](homework_25_26_v2.pdf). This overview follows that specification; execution evidence and implementation details belong to the lab configurations and captures.

## Network Architecture

The prescribed topology contains **16 nodes: 10 routers, 3 servers, and 3 client PCs**, with separate collision domains, routed transit networks, and private LANs.

| Component | Role |
| --- | --- |
| `r1`–`r4` | Statically routed network connecting `pc1` to the core; `r1` provides DHCP, NAT, and private-LAN protection. |
| `r5`–`r8` | OSPF core operating in area `0.0.0.0`, with a static route toward `50.0.0.0/24` through `r5`. |
| `r9` | DHCP, NAT, and firewall gateway for the LAN shared by `pc2` and `s2`. |
| `r10` | NAT and firewall gateway for `192.168.88.0/24`, hosting `pc3` and the VPN server `s3`. |
| `s1` | SSH server and intermediate host for SSH port forwarding. |

The LANs behind `r1` and `r9` both use `192.168.1.0/24` while remaining separate network segments. The exercise therefore combines overlapping private address space with NAT and distinct VPN client addresses.

## Project Scope

### Network Emulation & Routing

- Reproduce the supplied topology through Kathara's `lab.conf`, node directories, and collision-domain assignments.
- Configure static IPv4 addresses through `/etc/network/interfaces`, including the prescribed inter-router addressing scheme.
- Provide DHCP addressing and default gateways on the private LANs behind `r1` and `r9`.
- Combine **OSPF dynamic routing** on the four core routers with **static routing** elsewhere, targeting reachability across all non-private router interfaces.

### NAT & Firewall Policy

- Apply outbound **Network Address Translation (NAT)** on `r1`, `r9`, and `r10`.
- Protect private LANs by blocking unsolicited inbound traffic while allowing return traffic for connections initiated internally.
- Configure `r10` to expose the internal OpenVPN service through its external address.
- Introduce a restrictive forwarding policy on `r4` that permits only SSH traffic on TCP port `22`, creating the final tunneling scenario.

### PKI & OpenVPN

The VPN exercise uses a dedicated **Certificate Authority (CA)**, one server certificate, and two client certificates for certificate-based authentication.

| VPN setting | Assignment configuration |
| --- | --- |
| Server | `s3`, certificate CN `vpn_server` |
| Clients | `pc1` and `pc2`, certificate CNs `vpn_client_1` and `vpn_client_2` |
| Transport | TCP port `7070` |
| Tunnel subnet | `10.10.0.0/24` |
| Assigned client addresses | `pc1`: `10.10.0.100`; `pc2`: `10.10.0.200` |
| Client communication | OpenVPN `client-to-client` directive |
| Pushed route | `192.168.88.0/24` |

This stage connects clients behind different NAT gateways and extends VPN access to the server-side LAN. A return route on `pc3` toward the VPN subnet through `s3` completes the routing required for the client-to-LAN test.

### SSH Port Forwarding

The final scenario combines **SSH public-key authentication** to the `ubuntu` account on `s1` with **local port forwarding** from `pc1`.

Once `r4` permits only forwarded SSH traffic, the direct OpenVPN connection from `pc1` is expected to fail. The exercise then routes that VPN connection through an SSH tunnel using `s1` as the intermediate host, adapting the OpenVPN client configuration accordingly. This illustrates how application-layer tunneling interacts with transport-layer firewall rules.

## Traffic Capture & Validation Scenarios

The assignment specifies three **PCAP captures** using `tcpdump`, with Netcat and ICMP traffic to exercise the network paths:

| Capture | Traffic scenario | Capture location |
| --- | --- | --- |
| `capture_1.pcap` | `pc2` connects to a Netcat listener on `pc1`, TCP port `8080`, over the VPN. | An interface on `r1` |
| `capture_2.pcap` | `pc1` pings `pc3` through the VPN and the server-side LAN. | An interface on `s3` |
| `capture_3.pcap` | Repeat the Netcat test after restoring VPN connectivity through SSH port forwarding. | An interface on `r1` |

These scenarios support inspection of routing behavior, VPN encapsulation, and the effect of adding an SSH tunnel. Captures are specified under `/shared/`, and the SSH forwarding command is recorded in `commands.txt` in the lab directory.

OpenVPN, SSH port forwarding, traffic generation, and packet captures are started manually, as required by the assignment.

## Technologies & Skills

**Technologies:** Kathara, Linux networking, IPv4, DHCP, OSPF, NAT, OpenVPN, PKI, SSH, Netcat, and tcpdump.

The project brings together **network infrastructure configuration, routing troubleshooting, firewall policy enforcement, secure remote connectivity, certificate management, and packet-level diagnostics**, connecting addressing, routing, filtering, and tunneling across the full topology.

All scenarios are part of an authorized university lab and use the topology and addressing prescribed for the exercise.
