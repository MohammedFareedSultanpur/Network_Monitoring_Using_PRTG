# Troubleshooting Log

Each entry gives the symptom, how it was diagnosed and the fix.

## 1. SW1 unreachable from the PC
- **Symptom:** the PC could not ping 10.0.99.2. R1 could ping it, and SW1 could ping R1 (10.0.99.1) but nothing beyond it (192.168.10.x).
- **Diagnosis:** `show run | include ip routing|default-gateway|ip route` showed only `ip default-gateway 10.0.99.1`. IOSv L2 has IP routing enabled by default, and with routing on the switch ignores `ip default-gateway`, so it had no route for replies.
- **Fix:** `ip route 0.0.0.0 0.0.0.0 10.0.99.1`. The PC then reached SW1 (TTL 254).
- **Lesson:** replies need a return path. Ping from the near side only proves half of it.

## 2. Alpine host unreachable
- **Symptom:** R1 and the PC could not ping 10.0.99.10.
- **Diagnosis:** `ip addr show eth0` showed no IPv4 address. After the address was added, `ip route` still showed no default route, so replies could not leave the subnet.
- **Fix:** `ip addr add 10.0.99.10/24 dev eth0` and `ip route add default via 10.0.99.1`, run with `sudo` (the console user was not root). Saved as configs/alpine.sh.
- **Note:** these settings are lost on restart.

## 3. Windows blocking ICMP
- **Symptom:** R1 and SW1 could not ping the PC (192.168.10.1), although the PC could reach R1.
- **Fix:** Windows Firewall rule allowing inbound ICMPv4. Also added rules for UDP 162 (traps) and UDP 514 (syslog).

## 4. Command run on the wrong system
- **Symptom:** `route print 10.0.0.0` on the Alpine console printed BusyBox usage text.
- **Cause:** `route print` is a Windows command. On Alpine the equivalent is `ip route`.

## 5. Irrelevant sensors from auto-discovery
- **Symptom:** SW1 showed a red SSL Certificate sensor and a yellow SSL Security Check sensor, which would raise false alarms.
- **Cause:** auto-discovery found an HTTPS port on the switch and added sensors for its self-signed certificate.
- **Fix:** deleted both sensors and set discovery to manual so they would not return.

## 6. Harmless boot message
- **Symptom:** `Hostname "INSERTHOSTNAME-H" is not a legal LAT node name` at boot.
- **Cause:** a placeholder hostname before the startup config is applied. It does not affect operation.