# Firewall, DNS and HTTPS Behavior Lab

## Description
This lab documents a controlled experiment to understand how firewall rules
affect DNS resolution and HTTPS connectivity in a home lab environment.

The goal is to observe real network behavior using packet inspection tools
rather than relying on theoretical explanations.

---

## Lab Environment

- Internet Connection: VDSL (lab-only line)
- Router / Modem: TP-LINK VC220-G3u
- Host OS: Windows
- Virtualization: VirtualBox
- Guest OS: Kali Linux
- Network Interfaces:
  - eth0: Bridged (LAN)
  - eth1: NAT (Internet-facing)

---

## Lab Topology

```
Internet (VDSL)
      |
      v
[ISP Router / Modem]
[TP-LINK VC220-G3u ]
      |
      v
[  Host Machine    ]
[   192.168.1.x    ]
      |
      v
[  VirtualBox NAT  ]
[    10.0.2.1      ]
      |
      v
[  Kali Linux VM   ]
[    10.0.2.5      ]
```

---

## Tools Used

- tcpdump
- iptables
- ping
- Web browser (HTTPS testing)

---

## Baseline Observation

Before applying any firewall rules:

- DNS resolution was successful
- HTTPS websites loaded normally
- TCP 3-way handshake observed on port 443
- UDP 443 traffic observed (QUIC / HTTP/3)

This confirmed normal outbound connectivity via `eth1`.

---

## Experiment 1 – Blocking DNS Traffic

### Firewall Rules Applied

```bash
iptables -A OUTPUT -p udp --dport 53 -j DROP
iptables -A OUTPUT -p tcp --dport 53 -j DROP
```
Observed Behavior

- ```ping google.com``` failed with name resolution error

- ```ping 8.8.8.8``` remained successful

- ```tcpdump``` showed DNS requests without responses

Conclusion

Blocking DNS traffic prevents name resolution but does not remove
basic IP connectivity.

---

## Experiment 2 – Blocking HTTPS Traffic

### Firewall Rule Applied

```iptables -A OUTPUT -p tcp --dport 443 -j DROP```

Observed Behavior

- DNS resolution succeeded
- Websites failed to load and timed out
- No TCP 3-way handshake observed
- Only residual packets from existing connections appeared

Conclusion

Successful DNS resolution does not guarantee application-layer connectivity.
Firewall rules can silently prevent session establishment.

Key Takeaways

- DNS and HTTPS are independent but sequential dependencies
- Stateful firewalls can block traffic without visible errors
- Packet inspection is essential to understand real network behavior
- Port forwarding is unrelated to outbound internet access

Security Perspective

Understanding how firewall rules affect traffic flow is a fundamental
skill for network security and incident analysis.

Firewall behavior must be verified through observation, not assumptions.
