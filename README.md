# Network Security Monitoring Home Lab (Suricata + Zeek)

A home lab that copies live network traffic off a managed switch into an Ubuntu sensor running Suricata (IDS) and Zeek (network security monitoring), then tests whether the sensor sees and logs it.

## Architecture
- Host: Windows PC running VirtualBox
- Sensor: Ubuntu 26.04 VM
- Switch: TP-Link managed switch with port mirroring enabled
- Capture path: mirrored port > ASIX AX88179 USB 3.0 Gigabit Ethernet adapter > VM interface enp0s8

## Security stack
- Suricata: monitors enp0s8 and writes signature-based alerts to fast.log
- Zeek: installed from the Zeek repository; turns traffic into structured connection and protocol logs
- Zeek's live logs sit in a spool directory, reached through the symlink /opt/zeek/logs/current/

## Testing and analysis
Generated known test traffic with a plain-HTTP request:

    curl http://testmyids.com

Confirmed Zeek produced both health logs (capture_loss.log, weird.log, known_hosts.log) and protocol logs (software.log, http.log).

Parsed http.log with zeek-cut to show the sensor captured the source, destination and request details:

    cat /opt/zeek/logs/current/http.log | zeek-cut id.orig_h id.resp_h method host uri

## In progress: attack simulation
- Installed Nmap on the Windows host with winget, plus the Npcap capture driver
- Next: run a fast port scan against the sensor and check whether Suricata alerts

      nmap -F <sensor-ip>

## What I learned
- How port mirroring gets switch traffic to a sensor without sitting inline
- The difference between alert-based detection (Suricata) and passive logging (Zeek)
- Where Zeek keeps live logs and how to pull useful fields with zeek-cut
