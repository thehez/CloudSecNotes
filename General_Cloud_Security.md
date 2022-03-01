- [1. Overview](#1-overview)
- [2. Security within cloud environments](#2-security-within-cloud-environments)
  - [2.1. Shared Responsibility Model](#21-shared-responsibility-model)
  - [2.2. Data Identification & Classification](#22-data-identification--classification)
    - [2.2.1. Tagging resources](#221-tagging-resources)
  - [Data Protection](#data-protection)
    - [Tokenization](#tokenization)
    - [Encryption](#encryption)

# 1. Overview

This page serves as a repository for general security principles that are not tied to one cloud vendor or another. The information is intended to focus instead on security controls and principles that apply to generic cloud native environments or for understanding how traditional on-premise security techniques are altered for cloud.

# 2. Security within cloud environments

## 2.1. Shared Responsibility Model
The shared responsibility model comes up a lot in any cloud security conversation but I think it's fair to summarise it as the difference in what security aspects you (the customer) are responsible for compared to those which the provider is responsible for. These responsibilities also differ according to environment i.e. IaaS, PaaS, SaaS etc. For each service be aware of the controls you are responsible for or share responsibility for, generally this could be OS security, patching, network security controls, data access and middleware. 

Misunderstanding shared responsibility and assuming the cloud provider is securing a resource is a root cause of many security issues (looking at you public S3 buckets)

## 2.2. Data Identification & Classification
Classifying data can help organisations understand the sensitivity of data stored or transmitted within the environment and the threats that may target such data. There are multiple ways of doing this but keeping it simple can provide a quick way of assessing the value of data:

- Low - Information in this category may or may not be intended for public release, however, if it were exposed publicly the impact to the organization would be very low or negligible.
- Medium - Information should not be disclosed outside of the organization without the proper nondisclosure agreements. In many cases this type of data should be on a need-to-know basis within the organization. In most organizations, the majority of information will fall into this category.
- High - This information is vital to the organization, and disclosure could cause significant harm. Access to this data should be very tightly controlled, with multiple in-depth security controls.

Additionally regulations and legislations (GDPR, PCIDSS etc) may impose other security requirements for classifiying and protecting data that can be placed on top of initial data classification efforts.

### 2.2.1. Tagging resources
Tagging resources is good practice for a number of reasons, (datatypes, inventory, access) however it is important to come up with a centralised coherent set of tags that are applied consistently (including by any automated deployment processes) when resources are created.

## Data Protection
The majority of cloud services provide standardized options for enabling data protection:

### Tokenization
Tokenization replaces pieces of sensitive data with a token (usually randomly generated). It has the benefit that the token generally has the same characteristics as the original data, so underlying systems that are built to take that data shouldn't need to be modified. Only one place (a “token service”) knows the actual sensitive data that the token represents. Tokenization can also be used in conjunction with encryption.

### Encryption
Where possible data encryption should be enabled for all cloud services:
- In transit - i.e. being transmitted accross networks
- At rest - i.e. stored on a persisted disk
- In use - i.e. currently being processed by a CPU/RAM etc.
  
Within cloud hosted environments, most services offer in-built encryption features which make use of Key Management Services to take the burden off administrators. When using cloud based KMS, management and handling of the encryption keys is provided by the cloud provider (the provider encrypts the data and data encryption keys which are encrypted with a key encryption key that is stored in kms) which is typically known as **server-side encryption**.

In very sensitive environments, where organisations have strict security requirements server-side encryption may not be appropriate. As the cloud provider maintains the multi-tenant services and manages the keys it is possible for an attacker to abuse services to decrypt data. In these environments **client-side encryption** in which the organisation implements the encryption/decryption mechanisms on their own systems using their own libraries and processes. However this can be difficult to maintain and implement securely so should only be implemented as a necessity.

Another alternative to using KMS server-side encryption is using a dedicated Hardware Security Module (HSM). HSMs can store encryption keys and have significant logical and physical protections against unauthorized access. With most systems, anyone with physical access can easily get access, but an HSM has sensors to
wipe out the data as soon as someone tries to take it apart, scan it with X-rays, or generally mess about with it. 