# Securing Azure Storage with Virtual Network Service Endpoints

## Lab Overview
This lab focuses on securing Azure Storage using Virtual Network Service Endpoints and Network Security Groups (NSGs) within Microsoft Azure.
The primary goal was to enforce network-level access control by ensuring:
Traffic to Azure Storage remains within the Microsoft Azure backbone network.
Only resources located in a designated private subnet are permitted access to the storage account.
Resources outside the approved subnet are explicitly denied access.
All resources were deployed in the West US region.


## Lab Objectives

Securing Azure Storage with Service Endpoints

During this lab, I successfully completed the following:
1. Provisioned a Virtual Network (VNet).
2. Configured separate Public and Private subnets.
3. Enabled a Service Endpoint for Azure Storage on the private subnet.
4. Implemented Network Security Groups (NSGs) to control subnet traffic.
5. Deployed Azure Virtual Machines for validation testing.
6. Created a Storage Account with an Azure File Share.
7. Verified successful access from the authorized private subnet.
8. Confirmed access denial from the unauthorized public subnet.

## Architecture Summary

### Virtual Network Configuration
- **VNet Name:** myVirtualNetwork
- **Address Space:** 10.0.0.0/16
- **Region:** West US


### Subnets
| Subnet Name | Address Range   | Purpose |
|-------------|-----------------|----------|
| Public      | 10.0.0.0/24     | Internet-access VM |
| Private     | 10.0.1.0/24     | Restricted storage access |


<p align="center"><strong>Figure 1: Virtual Network Architecture with Public and Private Subnets</strong></p>

<p align="center">
  <img src="images/ASS1.png" width="700" height="400">
</p>


## Network Security Group Configuration

### NSG for Private Subnet (`myNsgPrivate`)

**Outbound Rules**
- Allow traffic to Azure Storage (Service Tag: Storage)
- Deny outbound traffic to Internet

<p align="center"><strong>Figure 2: NSG Outbound Rules for the Private Subnet</strong></p>

<p align="center">
  <img src="images/ASS2.png" width="700" height="400">
</p>


**Inbound Rules**
- Allow RDP (Port 3389) for testing

<p align="center"><strong>Figure 3: NSG Inbound Rules for the Private Subnet</strong></p>

<p align="center">
  <img src="images/ASS3.png" width="700" height="400">
</p>

<p align="center"><strong>Figure 4: Associating the myNSGPrivate to the Private Subnet</strong></p>

<p align="center">
  <img src="images/ASS4.png" width="700" height="400">
</p>

**Result:**
- Private VM can access Storage.
- Private VM cannot access the Internet.

   
### NSG for Public Subnet (myNsgPublic)

**Inbound Rules**
- Allow RDP (Port 3389)

<p align="center"><strong>Figure 5: NSG Inbound Rules for the Public Subnet</strong></p>

<p align="center">
  <img src="images/ASS5.png" width="700" height="400">
</p>

**Outbound Rules**
- Default rules apply (Internet allowed).

<p align="center"><strong>Figure 6: Associating the myNSGPublic to the Public Subnet</strong></p>

<p align="center">
  <img src="images/ASS6.png" width="700" height="400">
</p>

**Result:**
- Public VM has Internet access.
- Public VM cannot access Storage (denied).



## Storage Account Configuration

- **Performance:** Standard (General-purpose v2)
- **Redundancy:** Locally Redundant Storage (LRS)
- **File Share Name:** my-file-share
- **Public Network Access:** Enabled from selected networks only
- **Allowed Network:** Private subnet only

<p align="center"><strong>Figure 7: Storage Account Configuration</strong></p>

<p align="center">
  <img src="images/ASS7.png" width="700" height="400">
</p>

<p align="center"><strong>Figure 8: File Share Configuration for Storage Account</strong></p>

<p align="center">
  <img src="images/ASS8.png" width="700" height="400">
</p>

<p align="center"><strong>Figure 9: Enabling Service Endpoint for the Private Subnet</strong></p>

<p align="center">
  <img src="images/ASS9.png" width="700" height="400">
</p>

## Virtual Machines Deployed

| VM Name       | Subnet  | Purpose |
|--------------|----------|----------|
| myVmPrivate  | Private  | Storage access test |
| myVmPublic   | Public   | Access denial validation |


<p align="center"><strong>Figure 10: Creating a Virtual Machine in the Private Subnet</strong></p>

<p align="center">
  <img src="images/ASS12.png" width="700" height="400">
</p>


<p align="center"><strong>Figure 11: Creating a Virtual Machine in the Public Subnet</strong></p>

<p align="center">
  <img src="images/ASS11.png" width="700" height="400">
</p>


Both VMs:
- Windows Server 2022 Datacenter: Azure Edition
- Standard HDD disk
- Deployed in West US



## Validation Tests

### Test 1: Private VM Access to Storage

<p align="center"><strong>Figure 12: RDP connection to the Private VM</strong></p>

<p align="center">
  <img src="images/ASS16.png" width="700" height="400">
</p>


Established a network drive connection to Azure File Storage through PowerShell

<p align="center"><strong>Figure 13: Mapping drive to Azure File Storage through PowerShell on the Private VM</strong></p>

<p align="center">
  <img src="images/ASS13.png" width="700" height="400">
</p>


<p align="center"><strong>Figure 14: Successfully Mapping drive to Azure File Storage</strong></p>

<p align="center">
  <img src="images/ASS14.png" width="700" height="400">
</p>


**Result:**
- Z: drive successfully mapped.
- Storage access confirmed.

## Internet connectivity test:

<p align="center"><strong>Figure 15: Denied Internet Connectivity from the Private VM </strong></p>

<p align="center">
  <img src="images/ASS15.png" width="700" height="400">
</p>



**Result:** Failed (Internet blocked)


### Test 2: Public VM Access to Storage

<p align="center"><strong>Figure 16: RDP connection to the Public VM</strong></p>

<p align="center">
  <img src="images/ASS17.png" width="700" height="400">
</p>

Attempted same PowerShell mapping script.

<p align="center"><strong>Figure 17: Failed Drive Mapping to Azure File Storage through PowerShell on the Public VM </strong></p>

<p align="center">
  <img src="images/ASS18.png" width="700" height="400">
</p>

**Result:**

Access is denied


## Internet connectivity test:

<p align="center"><strong>Figure 18: Successful Internet Connectivity from the Public VM</strong></p>

<p align="center">
  <img src="images/ASS19.png" width="700" height="400">
</p>

**Result:** Successful (Internet allowed)


## Key Security Concepts Demonstrated
- Azure Virtual Network (VNet) segmentation
- Service Endpoint configuration for secure backbone routing
- Network Security Group (NSG) rule design and prioritization
- Storage account network access controls
- Least privilege network architecture
- Defense-in-depth implementation
- Outbound traffic restriction strategies
- Subnet-level access enforcement


## Real-World Significance

This lab reflects enterprise-grade cloud security practices where:

- Sensitive data (such as healthcare or financial records) must be protected from public internet exposure.
- Storage access is restricted to trusted application tiers within a private network boundary.
- Backend systems operate without unrestricted internet connectivity.
- Network segmentation reduces overall attack surface.
- Lateral movement within cloud environments is minimized through controlled subnet design.
- By leveraging Service Endpoints, traffic to Azure Storage remains within Microsoft’s secure backbone infrastructure rather than traversing the public internet, significantly improving data protection posture.


## Lessons Learned
- Service Endpoints are configured at the subnet level, not at the resource level.
- Storage firewall policies must align precisely with subnet configurations.
- NSG outbound rules can override default internet access behavior.
- Secure subnet design is foundational to resilient cloud architecture.
- Connectivity validation is a critical step in confirming security controls.


## Skills Demonstrated
- Azure networking and subnet configuration
- Storage security implementation and hardening
- NSG rule creation, ordering, and troubleshooting
- Secure virtual machine deployment
- Connectivity testing and validation
- Cloud security diagnostics
- Implementation of Zero Trust network principles


## Conclusion
This lab successfully demonstrated how to:
- Secure Azure Storage using Service Endpoints
- Enforce subnet-restricted access controls
- Prevent unauthorized subnet connectivity
- Implement controlled outbound internet access through NSGs

