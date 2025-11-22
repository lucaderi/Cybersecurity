## Learning Goals
* Translate raw network data into actionable risk intelligence using Zeek
* Understand risk as deviation from established baselines
* Quantify security incident impact through empirical evidence
* Design proactive risk controls using Zeek's scripting capabilities
* Integrate network monitoring into continuous risk assessment frameworks

## Prerequisites
* Virtual Machine with Zeek installed. You can download binary packages from [this repository](https://github.com/zeek/zeek/wiki/Binary-Packages)
* Sample PCAP files provided in course materials
* Basic familiarity with command-line tools (awk, sort, uniq, zeek-cut)

# Exercises
Below you can find a lot of exercises you can perform


## Exercise 1: Baseline Establishment & Anomaly Detection

### Learning Goal
Understand that risk is fundamentally about deviation from established norms and expected behavior patterns.

### Theoretical Context
As ICT Solutions Architects, you must recognize that effective risk assessment requires continuous monitoring and dynamic baselining rather than static compliance checklists.

### PCAP files
From malware-traffic-analysis.net:
- Look for "training exercises" with both normal and infected traffic
- Example: ["2023-05-23-traffic-analysis-exercise.pcap"
](https://www.malware-traffic-analysis.net/2023/05/23/index.html)

### Tasks

#### Process PCAP Files
```bash
# Process baseline traffic (normal business operations)
zeek -r baseline.pcap

# Process suspicious traffic (potential security incident)
zeek -r suspicious.pcap
```

#### Establish DNS Baseline
```
# Identify top 10 most queried domains in baseline
cat dns.log | zeek-cut query | sort | uniq -c | sort -nr | head -10

# Calculate normal query volume per source IP
cat dns.log | zeek-cut id.orig_h | sort | uniq -c | sort -nr > baseline_volume.txt

# Analyze query types and patterns
cat dns.log | zeek-cut qtype_name | sort | uniq -c | sort -nr
```

#### Identify and Quantify Anomalies
```
# Compare DNS activity between baseline and suspicious traffic
cat dns.log | zeek-cut query | sort | uniq -c | sort -nr > suspicious_dns.txt

# Calculate percentage increase for specific host
baseline_queries=$(cat baseline_dns.log | grep "192.168.1.100" | wc -l)
suspicious_queries=$(cat suspicious_dns.log | grep "192.168.1.100" | wc -l)
increase=$(echo "scale=2; ($suspicious_queries - $baseline_queries) / $baseline_queries * 100" | bc)
echo "DNS Query Volume Increase: $increase%"

# Identify new, previously unseen domains
comm -13 <(sort baseline_domains.txt) <(sort suspicious_domains.txt) > new_domains.txt
```

## Exercise 2: Quantifying Security Incident Impact

### Learning Goal

Measure the tangible business impact of realized threats through empirical data analysis.

### Theoretical Context

Risk impact must be quantified in concrete terms—data loss, system compromise, operational disruption—to enable informed business decisions and resource allocation.

### PCAP files
From Stratosphere or CTU datasets:
- Any botnet capture with C2 communication
- Example: [CTU-Malware-Capture-Botnet-48](https://mcfp.felk.cvut.cz/publicDatasets/CTU-Malware-Capture-Botnet-48/)

### Tasks

####  Identify Compromised Infrastructure
```bash
# Process the exfiltration scenario
zeek -r data_exfiltration.pcap

# Identify primary communication patterns
cat conn.log | zeek-cut id.orig_h id.resp_h | sort | uniq -c | sort -nr | head -5

# Analyze connection duration patterns
cat conn.log | zeek-cut duration | awk '{if ($1 > 0) print $1}' | sort -n
```

####  Calculate Data Exfiltration Volume
```bash
# Total data sent to identified C2 server (replace with actual IP)
cat conn.log | zeek-cut id.orig_h id.resp_h resp_bytes | \
  awk '$2 == "C2_IP_ADDRESS" {sum += $3} END {print "Total exfiltrated:", sum/1024/1024, "MB"}'

# Time-series analysis of data transfer
zeek-cut ts resp_bytes < conn.log | awk '/C2_IP_ADDRESS/ {print $1, $2}' > exfiltration_timeline.txt
```


####  Analyze Data Type and Transfer Methods
```bash
# Investigate HTTP data transfers
cat http.log | zeek-cut host uri method user_agent status_code | head -10

# Examine SSL/TLS encryption characteristics
cat ssl.log | zeek-cut server_name version cipher | sort | uniq -c

# Identify extracted files and their properties
cat files.log | zeek-cut conn_uids tx_hosts rx_hosts mime_type filename md5 | head -10```
