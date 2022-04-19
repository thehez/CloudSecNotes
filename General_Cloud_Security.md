- [1. Overview](#1-overview)
- [2. Shared Responsibility Model](#2-shared-responsibility-model)
- [3. Data Protection](#3-data-protection)
  - [3.1. Data Classification](#31-data-classification)
  - [3.2. Tagging resources](#32-tagging-resources)
  - [3.3. Tokenization](#33-tokenization)
  - [3.4. Encryption](#34-encryption)
    - [3.4.1. Why Encrypt](#341-why-encrypt)
    - [3.4.2. Implementing Encryption in the Cloud](#342-implementing-encryption-in-the-cloud)
    - [3.4.3. HSM vs KMS](#343-hsm-vs-kms)
- [IAM](#iam)

# 1. Overview
This page serves as a repository for general security principles that are not tied to one cloud vendor or another. The information is intended to focus instead on security controls and principles that apply to generic cloud native environments or for understanding how traditional on-premise security techniques are altered for cloud.

# 2. Shared Responsibility Model
The shared responsibility model comes up a lot in any cloud security conversation but I think it's fair to summarise it as the difference in what security aspects you (the customer) are responsible for compared to those which the provider are responsible for. These responsibilities also differ according to environment i.e. IaaS, PaaS, SaaS etc. For each service be aware of the controls you are responsible for, generally this will be OS security, patching, network security controls, data access and middleware. 

Misunderstanding shared responsibility and assuming the cloud provider is securing a resource is a root cause of many security issues (looking at you public S3 buckets)

# 3. Data Protection

## 3.1. Data Classification 
Classifying data can help organisations understand the sensitivity of the data that will be stored or transmitted within the environment, and form part of a threat modelling exercise. There are multiple ways of doing this but using a simple classification process can provide a succint but quick way of assessing the value of data:

- Low - Information in this category may or may not be intended for public release, however, if it were exposed publicly the impact to the organization would be very low or negligible.
- Medium - Information should not be disclosed outside of the organization without the proper nondisclosure agreements. In many cases this type of data should be on a need-to-know basis within the organization. In most organizations, the majority of information will fall into this category.
- High - This information is vital to the organization, and disclosure could cause significant harm. Access to this data should be very tightly controlled, with multiple in-depth security controls.

Additionally regulations and legislations (GDPR, PCIDSS etc) may impose other security requirements for classifiying and protecting data that can be placed on top of initial data classification efforts.

## 3.2. Tagging resources
Tagging resources is good practice for a number of reasons, (datatypes, inventory, access) however it is important to come up with a centralised coherent set of tags that are applied consistently (including by any automated deployment processes) when resources are created.

## 3.3. Tokenization
Tokenization is the act of substituting pieces of sensitive data with a token (usually randomly generated). The token generally has the same characteristics as the original data, so underlying systems that are built to take that data shouldn't need to be modified. And the benefit is that only one place (the “token service”) knows the actual sensitive data that the token represents. Tokenization can and also should be used in conjunction with encryption.

## 3.4. Encryption
To support a defence-in-depth approach, encryption should be enabled for all cloud services that support it:
- In transit - i.e. being transmitted accross networks
- At rest - i.e. stored on a persisted disk
- In use - i.e. currently being processed by a CPU/RAM etc.

### 3.4.1. Why Encrypt
Encryption is an interesting topic within cloud environments (due to various implementation and access issues) and it's useful to understand the threats that we are trying to protect against by using encryption:

- **Access to physical media** - Encryption prevents attackers from compromising physical assets to gain access to sensitive data. In a cloud environment it's fair to say this isn't a large risk since a providers datacenter will implement multiple physical and media based security controls to prevent unauthorised access to hardware. 
  
- **Access to platform or hosted system** - An attacker can attempt to compromise legitimate read/write access within the cloud platform (for example attacking the control plane to gain IAM permissions to use a service/key) to compromise data contained within. If encryption is managed by the database/storage system the attacker will likely be able to decieve the system into gaining access. In this case the only solution would be to use client-side encryption to ensure the data is encrypted before it is even stored in the cloud.

- **Access to provider hypervisor** - Typically the resources you use in a cloud environment are running on shared resources i.e. The provider runs a physical server with a hypervisor managing multiple virtual machines. If an attacker can compromise a shared resource they may be able to perform attacks such as dumping memory in an attempt to access data & encryption keys stored there. In the case of highly sensitive data, it may be worth investing in single tenant resources or specialist hardware technology that encrypts data in memory. However data breaches within cloud provider environments are statistically very low so a cost/risk analysis judgement should be made to decide the right approach.

### 3.4.2. Implementing Encryption in the Cloud
Within cloud hosted environments, most services offer in-built encryption features which make use of Key Management Services (KMS) to handle encryption. KMS keys can either by managed by the administrator or the cloud provider, however in both cases the provider handles encryption/decryption of data and data encryption keys which are wrapped with a key encryption key that is stored in kms. This process is typically known as **server-side encryption**.

In sensitive environments, where organisations have strict security requirements server-side encryption may not be appropriate. (As the cloud provider maintains the multi-tenant services and manages the keys it is possible for an attacker to abuse services to decrypt data.) In these environments **client-side encryption** in which the organisation implements the encryption/decryption mechanisms on their own systems using their own libraries and processes, may be required. However this can be difficult to maintain and implement securely so should reserved for especially sensitive data.

### 3.4.3. HSM vs KMS
A Hardware Security Module (HSM) stores encryption keys in a device with significant logical and physical protections against unauthorized access. With most systems, anyone with physical access to the equipment (such as stealing a hard-drive) could potentially get access to the data. Whereas a HSM device has sensors to wipe data as soon as it is tampered with, i.e. someone tries to take it apart, scan it or generally mess about with it. 

KMS uses HSM technology but those HSM clusters are multi-tenanted and distributed over multiple customer environments. Most cloud providers now provide an option to use a dedicated single-tenant hosted HSM for those environments which may have strict security or regulatory requirments in which you need to be able to manage the HSM devices (typically for FIPS etc).

# IAM
Identity and Access Management is probably the most important security control in cloud environments. Misconfigurations with IAM controls can be disasterous, as a malicious actor can leverage APIs to access the control plane to modify the data plane - a malicious user with the right privileges can modify firewall rules and data access on the fly from any location.





