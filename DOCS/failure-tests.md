# Failure Tests

All tests were run after the PRTG alert notification and state triggers (Down for 60 s, and Up) were in place.

## Test 1: R1-R2 link failure
**Action:** on R1, `interface GigabitEthernet0/1` then `shutdown`; later `no shutdown`.

| Stage | What was observed | Screenshot |
|---|---|---|
| Before | All sensors green | failure-tests/20 |
| Trigger | CLI shows OSPF neighbour 2.2.2.2 going FULL to DOWN, then the link and line protocol going down | failure-tests/21 |
| Detection | R2's Ping v2 sensor Down, message "Device 10.255.0.2 is unreachable", Down for 128 s when captured | failure-tests/22, 23 |
| Syslog | From 10.255.0.1: OSPF-5-ADJCHG neighbour down, LINK-5-CHANGED, LINEPROTO-5-UPDOWN for Gi0/1 | failure-tests/24 |
| Trap | SNMP IF-MIB::linkDown for GigabitEthernet0/1 ("administratively down") | failure-tests/25 |
| Recovery | After `no shutdown`, sensors returned to green | failure-tests/26 |

**Why it matters:** the same event is seen three ways. Polling (Ping) shows the symptom, syslog shows the cause (OSPF adjacency lost), and the trap shows which interface changed.

**Note on timing:** the PRTG alert fires after the sensor has been Down for the 60 s trigger delay plus up to one scan interval (60 s). Device timestamps in syslog do not match PRTG's clock because the lab has no NTP, so detection time is read from PRTG's "Down for" value.

## Test 2: Host link failure
**Action:** on SW1, shut `GigabitEthernet0/1` (the port to Alpine), then re-enable it.

| Stage | Observed | Screenshot |
|---|---|---|
| Failure | Alpine Ping sensor goes Down | failure-tests/27 |
| Recovery | Sensor returns to Up after the port is re-enabled | failure-tests/28 |

Only the host's Ping sensor is affected, because R1, R2 and SW1 are still reachable. This shows the failure was isolated to the host link.

## Test 3: Load
**Action:** on Alpine, `ping -s 1400 -i 0.2 10.255.0.2`.

**Result:** a visible spike on the interface traffic graph (failure-tests/29), compared with the idle baseline in prtg-setup/16.

## Conclusions
- Ping sensors detect failures within about two scan intervals.
- Syslog and traps explain the cause. Sensors alone only show that something is down.
- Monitoring loopbacks means a single link failure does not hide a device if another path exists. In this lab R2 has only one path, so its loopback goes down with the link.