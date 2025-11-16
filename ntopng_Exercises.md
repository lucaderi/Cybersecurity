# ntopng Cybersecurity Lab Exercises

## Network Baseline Establishment

### Scenario

You're the new security analyst at TechCorp Inc. (500 employees, mixed BYOD environment) and need to establish a network performance baseline during peak business hours.

#### Dashboard Analysis
- Monitor main dashboard for 8 continuous hours (9 AM - 5 PM)
- Record metrics:
  * Peak throughput times and values
  * Top 5 applications by bandwidth consumption  
  * Packet rate anomalies
  * Protocol distribution changes
 
#### Host Inventory & Classification
- Identify and document:
  * Top 10 talkers (outbound traffic)
  * Top 10 listeners (inbound traffic) 
  * Host roles (servers, workstations, network devices)
  * Unknown/Misconfigured hosts
 
#### Protocol Analysis
  - Analyze protocol distribution:
  * Business-essential protocols (HTTP/HTTPS, DNS, SMTP)
  * Unauthorized protocols (BitTorrent, gaming, P2P)
  * Encrypted vs unencrypted traffic ratios
  * Port usage patterns and anomalies

### Configuration Steps
```
# 1. Create baseline monitoring interface
ntopng -i eth0 -w 8080 -W 0.0.0.0:3000

# 2. Configure data retention
Settings -> Preferences -> Time Series -> 30 days retention

# 3. Set up host pools
Settings -> Users -> Host Pools -> Create by department
```


## Data Exfiltration Detection
#### Scenario

Suspected insider threat at FinancialData Corp. HR reports employee downloading large files before resignation.

#### Flow Analysis
1. Identify large outbound transfers:
   - Filter: Bytes Sent > 100MB
   - Time range: Last 7 days
   - Sort by: Client->Server bytes descending

2. Temporal analysis:
   - After-hours transfers (6 PM - 7 AM)
   - Weekend activity patterns
   - Unusual transfer durations

3. Geographic analysis:
   - Destination countries risk assessment
   - Unknown geographic locations
  
  #### Behavioral Analysis
  1. Compare to baseline:
   - Normal vs current transfer patterns
   - Protocol usage anomalies
   - Transfer frequency changes

2. Data analysis:
   - File type analysis from flow metadata
   - Transfer completion rates
   - Multiple destination patterns
  
#### Investigation Steps
```
# Load investigation data
ntopng -i /course_data/data-exfiltration.pcap

# Analysis workflow:
1. Flows -> Sort by "Bytes (Client->Server)"
2. Identify suspicious internal host
3. Host Details -> Historical -> Traffic Graphs
4. Check "Countries" for destination analysis
5. Export flow data for forensic analysis
```

### Botnet C2 Communication

#### Scenario

Multiple users report slow performance and unusual network activity. Suspicion of botnet infection.

#### Threat Intelligence Integration
```
1. Enable threat feeds:
   - Settings -> Blacklists
   - Enable: AlienVault OTX, Abuse.ch, FireHOL
   - Update frequency: Every 4 hours

2. Configure scoring:
   - Malicious IP threshold: 70/100
   - Suspicious threshold: 40/100
   - Automatic host blocking: Enabled
```

#### Beaconing Detection
```
1. Periodic communication analysis:
   - Flows -> Sort by "Duration"
   - Look for regular timing patterns (every 5, 10, 15 minutes)
   - Check flow consistency (similar packet sizes)

2. DNS analysis:
   - Protocols -> DNS -> Suspicious queries
   - Domain generation algorithm (DGA) patterns
   - Fast-flux domain indicators
```
#### Lateral Movement Analysis
```
1. Internal scanning detection:
   - Multiple failed connection attempts
   - Port scanning patterns
   - Service enumeration attempts

2. Exploitation monitoring:
   - SMB/RDP brute force attempts
   - Successful lateral movement
   - New service connections
```


### Ransomware Outbreak Response
#### Scenario
Active ransomware incident. Multiple workstations reporting encrypted files. CEO demands immediate containment.

#### Patient Zero Identification
```
1. Initial compromise analysis:
   - First malicious flow detection
   - Phishing email indicators (malicious attachments)
   - Exploit kit traffic patterns

2. Timeline reconstruction:
   - Initial infection time
   - First encryption activity
   - Lateral movement start time
```

#### Lateral Movement Tracking
```
1. SMB analysis:
   - Mass file access patterns
   - Encryption in progress indicators
   - Network share enumeration

2. RDP analysis:
   - Credential brute force attempts
   - Successful administrative logins
   - Unusual RDP session patterns
```
#### C2 Communication Analysis
```
1. Command channel identification:
   - HTTP/HTTPS to suspicious domains
   - DNS queries for C2 domains
   - Tor network connections

2. Data exfiltration:
   - Pre-encryption data theft
   - Ransom note delivery
   - Payment communication
```

