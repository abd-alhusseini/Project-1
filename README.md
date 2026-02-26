# Secure Small Office Network: Segmentation, DHCP Relay & ACL Hardening

## 1.  Project Mission
  The objective of this project was to design and deploy a scalable and secure network for a small business using GNS3. The design prioritizes **Traffic Isolation** (VLANs), **Security** (ACLs & SSH), and **Centralized Management** (DHCP Server) to ensure a professional and resilient office operation.

---

## 2. Network Topology

   A Router-on-a-Stick architecture provides inter-VLAN routing across six distinct departments, ensuring efficient use of hardware while maintaining logical segmentation.

![Topology Diagram](https://github.com/abd-alhusseini/Project-1/raw/main/pp/color%20topol.png)

---
## 3. IP Addressing and VLAN Plan
The network is segmented into the following VLANs to logically separate traffic and enforce security boundaries.

| Device / Interface        | Description                | IP Address          | Subnet Mask     | Default Gateway    | Assigned Network    |
|---------------------------|----------------------------|---------------------|-----------------|--------------------|---------------------|
| ISP Router G0/0            | ISP → R1 link              | 203.0.113.2         | 255.255.255.252 | N/A                | 203.0.113.0/30      |
| Router R1 G0/0             | R1 → ISP                   | 203.0.113.1         | 255.255.255.252 | N/A                | 203.0.113.0/30      |
| ISP Router G1/0            | ISP → Simulated Internet   | 192.168.122.64      | 255.255.255.252 | N/A                | 192.168.122.0/30    |
| Internet Host              | Simulated external host    | 192.168.122.1       | 255.255.255.252 | 192.168.122.1      | 192.168.122.0/30   |
| Windows-Server1            | DHCP server (VLAN 10)      | 192.168.10.10       | 255.255.255.0   | 192.168.10.1       | 192.168.10.0/24     |
| Windows-Admin1             | Admin PC (VLAN 10)         | 192.168.10.11       | 255.255.255.0   | 192.168.10.1       | 192.168.10.0/24     |
| PC-Management              | Management PC (VLAN 50)    | DHCP                | 255.255.255.0   | 192.168.50.1       | 192.168.50.0/24     |
| PC2-Finance1               | Finance PC (VLAN 20)       | DHCP                | 255.255.255.0   | 192.168.20.1       | 192.168.20.0/24     |
| PC3-Guest1                 | Guest PC (VLAN 30)         | DHCP                | 255.255.255.0   | 192.168.30.1       | 192.168.30.0/24     |
| PC4-Office1                | Office PC (VLAN 40)        | DHCP                | 255.255.255.0   | 192.168.40.1       | 192.168.40.0/24     |
| PC5-Office1                | Office PC (VLAN 40)        | DHCP                | 255.255.255.0   | 192.168.40.1       | 192.168.40.0/24     |
| Windows-PC1                | Sales workstation (VLAN 60)| DHCP                | 255.255.255.0   | 192.168.60.1       | 192.168.60.0/24     |


####  VLAN Planning
| VLAN ID | Name       | Network      | Gateway      | Devices                  |
| ------- | ---------- | ------------ | ------------ | ------------------------ |
| 10      | Admin      | 192.168.10.0 | 192.168.10.1 | Windows-Server, Admin-PC |
| 20      | Finance    | 192.168.20.0 | 192.168.20.1 | PC2-Finance1             |
| 30      | Guest      | 192.168.30.0 | 192.168.30.1 | PC3-Guest1               |
| 40      | Office     | 192.168.40.0 | 192.168.40.1 | PC4-Office1, PC5-Office1 |
| 50      | Management | 192.168.50.0 | 192.168.50.1 | PC-Management            |
| 60      | Sales      | 192.168.60.0 | 192.168.60.1 | Windows-PC2              |


## 4. Core Features and Configuration

### 4.1  DHCP Relay Agent
   
A centralized **Windows Server 2022** (VLAN 10) manages all IP address assignments. To overcome the limitation of DHCP broadcasts not crossing router boundaries, **DHCP Relay Agents** (`ip helper-address`) were configured on the router's sub-interfaces. This forwards client requests to the server, enabling efficient, centralized IP management.

![](https://github.com/abd-alhusseini/Project-1/raw/main/pp/Screenshot%202026-02-23%20012403.png)

![](https://github.com/abd-alhusseini/Project-1/raw/main/pp/Screenshot%202026-02-23%20012434.png)

![](https://github.com/abd-alhusseini/Project-1/raw/main/pp/Screenshot%202026-02-23%20012622.png)

### 4.2 Secure Management with SSH
  
SSH access is configured on router R1 to ensure secure, encrypted remote management. Administrators can connect using the built-in SSH client in Windows (via CMD ), 
After logging in to R1, perform an interface status overview

![](https://github.com/abd-alhusseini/Project-1/raw/main/pp/Screenshot%202026-02-23%20032936.png)

### 4.3 Access Control Lists (ACLs) for Security

Extended ACLs are implemented on R1 to enforce the security policy. The rules are designed to isolate user VLANs from each other and from the management network, while still allowing access to essential services like the DHCP server and the internet.

ACL Policy Summary:
| VLAN ID / Name / PC         | Access to VLAN 10 (Admin) | Access to Server 10.10 | Inter-VLAN Access       | Echo-reply from VLAN 10 | Echo-reply from VLAN 50 | Internet Access |
|-----------------------------|---------------------------|------------------------|-------------------------|-------------------------|-------------------------|-----------------|
| VLAN 20 PC20 - Finance       | Deny                      | Allow                  | Deny 30,40,50,60         | Deny                    | Deny                    | Allow           |
| VLAN 30 PC30 - Guest        | Deny                      | Allow                  | Deny all                | Deny                    | Deny                    | Allow           |
| VLAN 40 PC40/50 - Offices   | Deny                      | Allow                  | Deny 20,30,50,60         | Deny                    | Deny                    | Allow           |
| VLAN 50 PC1 - Management    | Deny to VLAN 10           | Allow                  | Allow 20,30,40,50,60     | Deny                    | Allow                   | Allow           |
| VLAN 60 PC60 - Sales        | Deny                      | Allow                  | Deny 20,30,40,50         | Deny                    | Deny                    | Allow           |
| VLAN 10 Windows PCs - Admin + Server | Allow              | Allow                  | Allow                   | Allow                   | Allow                   | Allow           |

Verification Tests:
- NAT/PAT Verification: A tracert from an internal PC to an external address confirms that traffic is correctly routed through the NAT gateway.
  
![](https://github.com/abd-alhusseini/Project-1/raw/main/pp/windows-pc%20tracert%20NAT%20.png)

- ACL Verification: A ping from the Management PC (VLAN 50) successfully reaches other VLANs but is blocked from reaching the Admin PC (192.168.10.11), confirming the ACL rule is working as intended.
  ![](https://github.com/abd-alhusseini/Project-1/raw/main/pp/PC-Managment%20ping%20all%20without%20192.168.10.11.png)


## 5. Skills Demonstrated
| Category | Skills & Technologies |
| :--- | :--- |
| **Network Design** | IP Subnetting, VLAN Planning, Router-on-a-Stick Topology |
| **Cisco IOS** | Sub-interfaces, Trunking, DHCP Relay, Extended ACLs, NAT/PAT, SSH |
| **Server Admin** | Windows Server 2022 (DHCP Role, Scope Configuration) |
| **Security** | Traffic Filtering, Secure Management Protocols, Network Segmentation |
| **Tools** | GNS3, Cisco C7200, Cisco IOSv-L2, Windows Server 2022, Windows 10, VPCS |
