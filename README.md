# Nmap-Cheatsheet
## Table of Contents
1. [Host Discovery](#host-discovery)
2. [Port Scanning](#port-scanning)
3. [Service Version & OS Detection](#service-version--os-detection)
4. [Nmap Scripting Engine (NSE)](#nmap-scripting-engine-nse)
5. [Firewall & IDS Evasion](#firewall--ids-evasion)
6. [Scan Optimization](#scan-optimization)
7. [Save Scan Results](#save-scan-results)
## Host Discovery 
When utilizing host discovery it is crucial to use "-sn"; to disable Port scanning
```bash
 nmap -sn <Target-ip>
```
In some cases, you must use this command against Windows machines to disable ping scans
```bash
nmap -Pn <Target-ip>
```
Then you can add other switches along with "-sn":
| Switch   |   Name         | Feature                                        |
|----------|----------------|------------------------------------------------|
| -PA      | TCP ACK Ping   | Good for detecting a firewall                  |  
| -PU      | UDP ping       | Faster & good if host do not respond for TCP   |
| -PS      | TCP SYN Ping   | Quiter                                         |
| -PE      | ICMP echos     |                                                |
For all of these four switches, you can specify a port along with them:
##### Example:
``` bash
nmap -sn -PS21 <Target-ip>
```
### Best two example commands for Host Discovery
#### 1.
```bash
nmap -sn -T4 -PS21,22,25,80,445,3389,8080 -PU137,138 10.0.2.0/24
```
#### 2.
```bash
nmap -sn -T4 10.0.2.0/24
```
## Port Scanning
After we knew the up hosts we must skip the host discovery phase by adding this switch:
```bash
nmap -Pn <target-ip>
```
Then you can add these switches along with "-Pn":
| Switch   |   Name                 | Feature                                | Notes                | 
|----------|------------------------|----------------------------------------|-----------------------
| -sS      | SYN Scan               | Sends a SYN                            | Stealthy and fast    |
| -sT      | TCP Connect Scan       | Utilize full Three-way-handshake       | Loud                 |
| -sU      | UDP Scan               | Sends UDP probes                       | Fast                 |
| -sA      | ACK Scan               | Sends an ACK                           | Detect Firewalls     |
### Ports states
| States   |                                                   Meaning                                                        | 
|----------|------------------------------------------------------------------------------------------------------------------|
| Open     | There is traffic intercepting this port                                                                         | 
| Filtered | Windows firewall is not active or there aren’t any rules for that particular port that is intercepting traffic  | 
| Closed   | Confirms the fact that you are dealing with a Windows firewall                                                   | 
### !!Sometimes Nmap can not really be sure whether the port is filtered or closed!!
## Service Version & OS Detection 
#### To utilize service version detection:
```bash
nmap -sV <target-ip>
```
#### To increase the intensity of the version detection:
```bash
nmap -sV --version-intensity<0-9>
```
The more you increase the intensity; the slower the scan will be
#### To utilize Operating System Detection:
```bash
nmap -O <Target-ip>
```
#### For more accurate Operating System Detection scan (Aggressive OS guesses)
```bash
nmap -O --osscan-guess <Target-ip>
```
### Best command for services & Operating System detection:
```bash
nmap -Pn -sS -p- -sV --version-intensity 8 -O --osscan-guess -T4 <target ip>
```
-p- : to scan all ports
-T<0-5> : Speed up your scan
## Nmap Scripting Engine (NSE)
### You can find all the Nmap scripts in this folder:
```bash
ls -l /usr/share/nmap/scripts | grep <Service>
```
#### Nmap Scripts can have multiple purposes and results
|Category       |         Purpose           |
|---------------|---------------------------| 
| Safe (Default)| Won't harm the target     |    
| unsafe        | Would harm the target     |   
| Enumeration   | For gathering information |
| Auth          | For authentication        |
| Broadcast     | Host discovery            |
| Exploitation  | For exploiting the system |
| Brute         | Brute-Force Attack        |
and a lot more...
### Default Script Scan
```bash
nmap -sC <Target-ip>
```
The default script scan will use a Set of scripts, and provide useful information without exploiting or damaging the target system 
Whenever the NSE detects a service that has a script for it, it will run. But only if this script falls within the DEFAULT category
### Specify a script or multiple scripts
!!Don't include the extension in the command (.NSE)!!
```bash
nmap --script=<Script-Name> <target-ip>
```
#### Set of scripts
```bash
nmap --script=<Script-Name>,<Script2-Name> <target-ip>
```
#### Some scripts might ask for "Scripts arguments"
```bash
nmap --script=<Script-Name> --script-args <Arguments>
```
### To get help in any script:
```bash
nmap --script-help=<script>
```
Or you can look it up in https://nmap.org/book/man-nse.html
#### Other Examples:
This will scan using all scripts related to "ftp"
```bash
nmap -sS -T4 -p20,21 --script=ftp* <target-ip>
```
This will do a full enumeration on "smb"
```bash
nmap -sS -T4 -p445 --script=smb-enum-* --script-args smbusername=<username> smbpassword=<password> <Target-ip>
```
This will scan using all scripts related to "http"
```bash
nmap -sS -T4 -p20,21 --script=http* <target-ip>
```
#### !! It is preferable to add [-sS -sV -Pn] along with both default or specified scripts!!
## Firewall & IDS Evasion
### Detecting the presence of a firewall or a filtering mechanism
You can detect a firewall using the "ACK Scan". Depending on the result
```bash
nmap -sA 
```
| Result   |              Fact               |
|----------|---------------------------------| 
| Open     | There is no filtering mechanism |
| Filtered | There is a filtering mechanism  |
### IDS Evasion 
|          Switch             |                           Result                                | 
|-----------------------------|-----------------------------------------------------------------| 
| -f --mtu<multiplies of 8>   | Fragment the sending packets and you can optimize the data size |
| --ttl<Value>                | Altering the Time To Live value                                 |
| --data-length<Value>        | Altering data length                                            |  
| -D <ip>                     | Altering the destination IP                                     |
| -g<Port>                    | Altering the destination port                                   |
| -T0                         | Slowing down your scan                                          |
| --scan-delay<value>         | Space the duration between each packet being sent               |
## Scan Optimization 
### Time Templates
|  Switch  |              Name               |
|----------|---------------------------------|
|   -T0    | Paranoid. Used for IDS evasion  |
|   -T1    | Sneaky                          |
|   -T2    | Polite                          |
|   -T3    | Default                         |
|   -T4    | Aggresive. Fast and reliable    |
|   -T5    | Insane. Not accurate in results |
### To give up on a non-responding target while the host discovering (You might lose an up host)
#### Example:
```bash
nmap --host-timeout <time> 10.0.2.0/24
```
## Save Scan Results
|  Switch  |     Purpose and extension         |
|----------|-----------------------------------|
|   -oN    | Normal .txt format                |
|   -oX    | .XML format. Used for databases   |                       
|   -oG    | Grepable format. Used for scripts |                         
|   -oA    | A compination of (oX + oN + oG)   |                      
## Other
### Disable DNS resolution to make your scan faster
```bash
nmap -n 
```
### Scan multiple targets
```bash
nmap -iL <targets.txt>
```
### Scanning a range of IP's in a LAN
```bash
nmap --send-ip <Target-ip>
```
### Scanning a range of IP's techiques
#### Example #1
```bash
nmap 10.0.2.0/24
```
#### Example #2
```bash 
nmap 10.0.2.3,4,5,6
```
#### Example #3
```bash
nmap 10.0.2.4, 10.0.2.7
```
### Specify ports techiques
#### Example #1
```bash
nmap -p1-120
```
#### Example #2
```bash
nmap -p443,445,3389
```
#### Example #3
```bash
nmap -p21
```
#### Example #4: Scan all ports (Very important)
```bash
nmap -p-
```
#### To scan the most used 100 ports
```bash
nmap -F 
```

## Credits
### https://nmap.org/
