# Design Note: Network Monitoring Lab (PRTG + Cisco CML)

## 1. Objective
Build a small enterprise-style network in Cisco Modeling Labs and monitor it with PRTG, to practise what a NOC engineer does daily: monitor availability and performance, receive alerts, and detect and diagnose failures.

## 2. Environment
| Component | Detail |
|---|---|
| Monitoring server | PRTG (free edition, up to 100 sensors) on a Windows PC |
| Emulation | Cisco CML Free (5-node limit) in VMware Workstation |
| Nodes | 2 x IOSv routers, 1 x IOSv L2 switch, 1 x Alpine host, 1 External Connector |
| Link to PC | External Connector in bridge mode on VMnet8 (192.168.10.0/24) |

## 3. Topology and addressing
See the README for the diagram and address table.

## 4. Design decisions
- **R1 is the only device on the PC's network.** The others sit behind it, as at a real site with one edge device toward the monitoring network. It also fits the 5-node limit.
- **Monitor loopbacks, not physical interfaces.** A loopback stays up while the device has any working path, so one link failure does not make the device look dead. Traps and syslog use the loopback as source address, so PRTG can match messages to the device it has on record.
- **Static route on the PC** (10.0.0.0/8 via R1) keeps the lab simple and avoids a routing protocol on the PC.
- **OSPF area 0 between R1 and R2** distributes loopbacks and links. A link failure drops the adjacency and logs a message, which gives a realistic event to detect. R1 originates a default route so R2 can send replies back to the PC.
- **Static default route on SW1.** IP routing is enabled on the switch, so `ip default-gateway` is ignored.
- **SNMPv2c with an ACL.** The community string works only from the PRTG PC (192.168.10.1).
- **Traps and syslog alongside polling.** Polling shows state at intervals. Traps and syslog show events as they happen and explain the cause.
- **Manual sensor selection.** Auto-discovery added irrelevant SSL sensors on SW1, so they were removed and discovery set to manual.

## 5. Monitoring scope
| Layer | Monitored | Sensor |
|---|---|---|
| Availability | Reachability, latency, loss | Ping |
| Device health | CPU, memory, uptime | CPU Load, System Health, SNMP Uptime |
| Performance | Interface throughput | SNMP Traffic |
| Events | Link, config and OSPF changes | SNMP Trap Receiver, Syslog Receiver |
| Alerting | Down for 60 s, and recovery | State triggers and a notification template |

Alpine has no SNMP agent, so it has a Ping sensor only.

## 6. Limitations
- SNMPv2c sends the community string in clear text. Use SNMPv3 in production.
- No NTP, so device timestamps differ from PRTG's clock.
- IOSv exposes limited MIBs, so some health sensors report less than real hardware.
- Alpine loses its IP configuration on restart.
- No NetFlow, so no per-application traffic view.
- Everything runs on one PC, so latencies do not reflect a real network.

## 7. Possible improvements
SNMPv3, NTP, NetFlow, email alerting through SMTP, and a Zabbix comparison.