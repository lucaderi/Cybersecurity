# Network Segmentation - From Flat to Meshed

**Course:** Network Security & Design

**Topic:** Architecting a Segmented Network for Enhanced Security and Performance

**Objective:** Students will be able to analyze a flat network design, identify its security weaknesses, and redesign it into a segmented, meshed network using VLANs, routers/firewalls, and ACLs. They will then implement security policies to control traffic between segments.

---

## The Exercises (5 Core + 5 Advanced)

We will start with foundational, hands-on exercises and move towards complex, security-focused scenarios.

### Part 1: The Foundation - Understanding the "Flat" Problem & Basic Segmentation

#### Exercise 1: The "Flat Network" Vulnerability Assessment (Diagramming & Analysis)
- **Scenario:** Students are given a network diagram of a small company with a single switch and one router to the internet. All devices (Servers, Admin PCs, HR PCs, Guest Wi-Fi Access Point, IP Phones) are connected to the same switch and are on the same IP subnet (e.g., 192.168.1.0/24).
- **Task:**
    1.  Identify all the security and performance problems with this design. (e.g., broadcast storms, anyone can sniff traffic, a virus on a guest laptop can easily infect the HR server, no access control).
    2.  Explain how a single misconfigured device or an ARP spoofing attack could bring down the entire network.
- **Deliverable:** A short written report or a class discussion.

#### Exercise 2: Introduction to VLANs - Creating the First Segments (Simulation)
- **Scenario:** Using the same flat network from Exercise 1 in a simulator (like Packet Tracer).
- **Task:**
    1.  Create three VLANs: VLAN 10 (Management/Admin), VLAN 20 (HR), and VLAN 30 (Guest).
    2.  Assign the correct switch ports to each VLAN. The server, Admin PCs, HR PCs, and Guest AP should now be in their respective VLANs.
    3.  Configure the switch port connected to the router as a **trunk** to carry traffic for all VLANs.
    4.  **Test:** Verify that PCs in the same VLAN can ping each other, but PCs in different VLANs cannot.
- **Key Learning:** Students see how a single switch can be logically divided to create isolated broadcast domains, immediately containing traffic and limiting the scope of potential issues.

#### Exercise 3: Inter-VLAN Routing - The "Router-on-a-Stick" (Simulation)
- **Scenario:** Building on Exercise 2, we now need to allow limited, controlled communication between VLANs.
- **Task:**
    1.  On the router, create sub-interfaces on the physical interface connected to the switch trunk (e.g., G0/0.10, G0/0.20, G0/0.30).
    2.  Assign an IP address to each sub-interface, which will act as the default gateway for each VLAN (e.g., 192.168.10.1/24 for VLAN 10).
    3.  Ensure the encapsulation matches the VLAN ID (e.g., `encapsulation dot1Q 10`).
    4.  **Test:** Verify that a PC in VLAN 10 can now ping a PC in VLAN 20.
- **Key Learning:** Students understand that traffic *must* pass through a Layer 3 device (router) to move between segments, creating a natural checkpoint for security policies. This is the core concept of a "meshed" network from a routing perspective.

### Part 2: Building the "Meshed" Security Architecture

#### Exercise 4: Implementing a Firewall in the Path (Simulation)
- **Scenario:** The router-on-a-stick is functional, but it provides no security; it routes everything. We need to replace the router with a firewall (e.g., an ASA or pfSense device in the simulator).
- **Task:**
    1.  Replace the router in your topology with a firewall device.
    2.  Configure the firewall's interfaces. This often means assigning each VLAN to a **physical interface** on the firewall for better performance and clarity, rather than a single trunk. (e.g., Firewall interface G0/1 = VLAN 10 network, G0/2 = VLAN 20 network).
    3.  Connect these firewall interfaces to access ports on the switch (one switch port per VLAN).
    4.  Configure the firewall's security zones (e.g., Guest Zone, HR Zone, Corporate Zone).
- **Key Learning:** Students learn the difference between a router and a firewall. The firewall becomes the central hub of the "mesh," with the explicit job of controlling all communication.

#### Exercise 5: Writing Access Control Policies (Simulation)
- **Scenario:** With the firewall in place, traffic between VLANs is now blocked by default.
- **Task:** Write firewall rules (ACLs) to enforce the following company policies:
    1.  **Permit:** Admin PCs (VLAN 10) can access the HR Server (VLAN 20) on TCP port 445 (SMB/CIFS for file sharing).
    2.  **Permit:** HR PCs (VLAN 20) can access the HR Server (VLAN 20) on all ports (they are in the same segment).
    3.  **Permit:** All users can access the internet.
    4.  **Permit:** Guest users (VLAN 30) can only access the internet (HTTP/HTTPS/DNS) and are explicitly denied access to all internal networks (VLANs 10 & 20).
    5.  **Deny:** All other traffic not explicitly permitted.
- **Key Learning:** Students move from theory to practice, translating business security requirements into technical, enforceable rules. This is the essence of a meshed security posture.

### Part 3: Advanced Exercises & Deeper "Meshing"

#### Exercise 6: The Demilitarized Zone (DMZ) (Simulation)
- **Scenario:** The company needs to host a public-facing web server and an email server.
- **Task:** Redesign the network to include a new, isolated segment called a **DMZ**.
    1.  Create a new VLAN (e.g., VLAN 100) for the DMZ.
    2.  Place the web and email servers in this VLAN.
    3.  Configure the firewall with a new interface/zone for the DMZ.
    4.  Write new firewall rules:
        - Allow traffic from the Internet (WAN) to the DMZ web server (port 80/443) and email server (port 25/587).
        - Allow the DMZ servers to initiate connections to the internet for updates (but block inbound internet-initiated connections to other zones).
        - Allow Admin PCs (VLAN 10) to manage the DMZ servers (e.g., SSH/RDP).
- **Key Learning:** Students learn the "3-legged firewall" model and how to create a buffer zone for publicly accessible assets, protecting the internal LAN even if the DMZ servers are compromised.

#### Exercise 7: Intrusion Prevention System (IPS) Integration (Conceptual & Configuration)
- **Scenario:** The firewall is working, but we need deeper inspection of traffic.
- **Task:** Place an **IPS inline** between the firewall and the core switch, or configure the firewall's built-in IPS module.
    1.  Configure the IPS to monitor traffic flowing from the untrusted zones (Guest, Internet) to the trusted zones (Admin, HR).
    2.  Define a rule to drop traffic that matches a signature for a known exploit (e.g., a SQL injection attempt from the guest network towards the HR database server). In a simulation, this might involve creating a custom signature or enabling a pre-defined one.
- **Key Learning:** Students learn about defense-in-depth. The firewall controls access based on policies (who), while the IPS inspects the traffic itself (what) for malicious content.

#### Exercise 8: High Availability and Redundancy (Conceptual Diagramming)
- **Scenario:** The firewall is now a single point of failure.
- **Task:** Redesign the topology to introduce high availability.
    1.  Diagram a solution using a pair of firewalls in an active/passive or active/active cluster.
    2.  Diagram the necessary switch connections and protocols (like VRRP or HSRP) to ensure that if one firewall fails, the other automatically takes over without disrupting traffic between segments.
- **Key Learning:** Students begin to think about network resilience. A truly professional design isn't just secure, but also highly available.

#### Exercise 9: 802.1X for Port-Based Authentication (Conceptual)
- **Scenario:** Anyone plugging a device into a wall jack currently gets access to the VLAN assigned to that port.
- **Task:** Design a plan to implement 802.1X.
    1.  Describe how a user (supplicant) connecting to a switch (authenticator) would need to provide credentials to a RADIUS server.
    2.  Explain how the RADIUS server can then dynamically assign the switch port to the correct VLAN based on the user's group membership (e.g., a student gets the Student VLAN, a professor gets the Faculty VLAN).
- **Key Learning:** Students learn about advanced access control, moving segmentation from a physical-port-based model to a user-identity-based model.

#### Exercise 10: Full "Greenfield" Design Project (Capstone)
- **Scenario:** A company is moving to a new office. You are given a list of requirements:
    - 50 Admin/Engineering staff (need access to development servers).
    - 20 Sales/HR staff (need access to file shares and CRM).
    - A public web server.
    - A "Bring Your Own Device" (BYOD) Wi-Fi network for employees.
    - A separate Guest Wi-Fi network.
    - Strict requirement: The BYOD network can only access the internet and the CRM, but NOT the internal file servers or development servers.
- **Task:** Design the entire network from scratch. Create a detailed diagram showing switches, a firewall, an IPS (optional), and all VLANs. Write the full firewall rule set required to meet the business needs.
- **Deliverable:** A comprehensive network design document and diagram.