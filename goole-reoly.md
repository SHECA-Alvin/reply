## **Incident Report**

### **Summary**

SHECA received an email notification that the revoked certificates in two recently closed cases had different revocation reason codes.

- [Case 1](https://bugzilla.mozilla.org/show_bug.cgi?id=1902947)
- [Case 2](https://bugzilla.mozilla.org/show_bug.cgi?id=1902592)



### **Impact**

According to the BR guidelines, the CRLReason codes for subscriber certificates must comply with the requirements of TLS BR 4.9.1.1. In the mentioned case, the revoked certificates had multiple CRLReason codes, and the "privilegeWithdrawn" CRLReason code was used incorrectly.

**According to the BR and Mozilla guidelines, the use of `privilegeWithdrawn` should be as follows:**

privilegeWithdrawn

The CRLReason `privilegeWithdrawn` is intended to be used when there has been a subscriber-side infraction that has not resulted in key compromise, such as when the certificate subscriber provided misleading information in their certificate request or failed to uphold their material obligations under the subscriber agreement or terms of use.

Unless the `keyCompromise` CRLReason is being used, the CRLReason `privilegeWithdrawn` MUST be used when:

- The CA operator obtains evidence that the certificate was misused;
- The CA operator becomes aware that the certificate subscriber has violated one or more of its material obligations under the subscriber agreement or terms of use;
- The CA operator becomes aware that a wildcard certificate has been used to authenticate a fraudulently misleading subordinate fully-qualified domain name;
- The CA operator becomes aware of a material change in the information contained in the certificate;
- The CA operator determines or becomes aware that any of the information appearing in the certificate is inaccurate; or
- The CA operator becomes aware that the original certificate request was not authorized, and the subscriber does not retroactively grant authorization.

Otherwise, the `privilegeWithdrawn` CRLReason MUST NOT be used.



SHECA found that the CRLReason code of some revoked certificates does not comply with regulations. These certificates should not use privilegeWithdrawn as the CRLReason code。

SHECA adheres strictly to BR regulations in the use of CRLReason codes. Attached is a screenshot showing some of the reasons for certificate revocation selected by SHECA subscribers, all of which comply with BR regulations.



**Detailed certificate revocation reasons are shown in the table below**

**Case 1:** [Bugzilla Case](https://bugzilla.mozilla.org/show_bug.cgi?id=1902592)

| Link                                            | Code               | Revocation Date |
| ----------------------------------------------- | ------------------ | --------------- |
| [Certificate 1](https://crt.sh/?id=10981773243) | unspecified (0)    | 2024-06-14      |
| [Certificate 2](https://crt.sh/?id=11044872118) | privilegeWithdrawn | 2024-06-14      |
| [Certificate 3](https://crt.sh/?id=11033992247) | unspecified (0)    | 2024-06-14      |
| [Certificate 4](https://crt.sh/?id=11032757670) | unspecified (0)    | 2024-06-19      |
| [Certificate 5](https://crt.sh/?id=11031824086) | privilegeWithdrawn | 2023-11-08      |
| [Certificate 6](https://crt.sh/?id=11031824053) | privilegeWithdrawn | 2023-11-08      |
| [Certificate 7](https://crt.sh/?id=10810579104) | privilegeWithdrawn | 2023-10-17      |
| [Certificate 8](https://crt.sh/?id=10810577035) | unspecified (0)    | 2023-10-17      |

**Case 2:** [Bugzilla Case](https://bugzilla.mozilla.org/show_bug.cgi?id=1902947)

| Link                                            | Code               | Revocation Date |
| ----------------------------------------------- | ------------------ | --------------- |
| [Certificate 1](https://crt.sh/?id=12421772198) | Expired            |                 |
| [Certificate 2](https://crt.sh/?id=12573150826) | privilegeWithdrawn | 2024-04-02      |
| [Certificate 3](https://crt.sh/?id=12421770452) | privilegeWithdrawn | 2024-04-02      |
| [Certificate 4](https://crt.sh/?id=12421770586) | superseded         | 2024-06-17      |
| [Certificate 5](https://crt.sh/?id=11917833535) | superseded         | 2024-06-17      |
| [Certificate 6](https://crt.sh/?id=11044872118) | privilegeWithdrawn | 2024-06-14      |

### **Timeline**

All timestamps are in Beijing Time (UTC+8):

- **2023-08-22 00:00** Received notification email.
- **2023-08-22 09:00** Investigation began to determine the reasons behind the different revocation codes.
- **2023-08-22 12:47** Issue confirmed, and it was discovered that the system incorrectly assigned the wrong CRLReason codes.
- **2024-08-22 15:30** Reason for the issue:

  - **unspecified (0):**  When SHECA's certificates encounter compliance issues, as a commercial CA, SHECA will do its utmost within the allowable regulations to ensure uninterrupted service for subscribers. We will promptly notify the subscribers to apply for a new replacement certificate through SHECA's issuance system. The subscriber will then request the revocation of their original certificate and choose the default CRLReason code "unspecified (0)," as most users do not understand the purposes of the other CRLReason code options.

  - **privilegeWithdrawn:** SHECA uses two RA systems, one of which has a defect. Even if the subscriber selects "unspecified (0)" the value ultimately transmitted to the CA system is "privilegeWithdrawn"

  - **superseded:** For subscribers who do not proactively request revocation, SHECA uses a script to batch revoke certificates, assigning the CRLReason code “superseded,” which is compliant with TLS BR 4.9.1.1 (12).

- **2024-08-22 15:31** Suspended all certificate revocation operations.

### **Root Cause Analysis**

- **Why were different CRLReason codes used for the revoked certificates?**

  - **unspecified (0):** When SHECA's certificates encounter compliance issues, as a commercial CA, SHECA will do its utmost within the allowable regulations to ensure uninterrupted service for subscribers. We will promptly notify the subscribers to apply for a new replacement certificate through SHECA's issuance system. The subscriber will then request the revocation of their original certificate and choose the default CRLReason code "unspecified (0)," as most users do not understand the purposes of the other CRLReason code options.

   - **privilegeWithdrawn:** SHECA uses two RA systems, one of which has a defect. Even if the subscriber selects "unspecified (0)" the value ultimately transmitted to the CA system is "privilegeWithdrawn"

  - **superseded:** For subscribers who do not proactively request revocation, SHECA uses a script to batch revoke certificates, assigning the CRLReason code “superseded,” which is compliant with TLS BR 4.9.1.1 (12).


- **Why wasn't this issue detected in time?**

  SHECA conducted a linting process on the signed CRL, but this process could not detect the issue. Although the value of the CRLReason code was correct, its usage did not strictly adhere to the BR guidelines, and SHECA's RA system imposed no special restrictions on the use of the "privilegeWithdrawn" CRLReason code.

### **Lessons Learned**

#### **What went well**

SHECA strictly adheres to BR guidelines for using CRLReason codes and has established a comprehensive revocation process, allowing us to quickly identify the issue.

#### **What didn't go well**

This issue could not be detected using linting tools because the CRLReason codes were correct, but their usage context was incorrect.

### **Action Items**

| **Action Item**                                              | **Type**     | **Due Date** |
| ------------------------------------------------------------ | ------------ | ------------ |
| Fix the incorrect CRLReason code value transmission issue in the affected RA system. | *Prevention* | 2024-08-27   |
| Review all CRLReason code usage scenarios to confirm no further issues exist. | *Prevention* | 2024-08-30   |
| In the revocation process, restrictions on the use of the "privilegeWithdrawn" CRLReason code will be enhanced. If an operator chooses to revoke a certificate using the "privilegeWithdrawn" CRLReason code, they must upload relevant evidence of the subscriber's violation certificate. The certificate can only be officially revoked after the compliance department reviews and approves the evidence. | *Prevention* | 2024-08-30   |

### **Appendix**

#### **Detailed Information on Affected Certificates**

| Link                                      | CRLReason Code     | Revocation Date |
| ----------------------------------------- | ------------------ | --------------- |
| https://crt.sh/?id=10981773243            | unspecified (0）   | 2024-06-14      |
| https://crt.sh/?id=11044872118            | privilegeWithdrawn | 2024-06-14      |
| https://crt.sh/?id=11033992247            | unspecified (0）   | 2024-06-14      |
| https://crt.sh/?id=11032757670            | unspecified (0）   | 2024-06-19      |
| https://crt.sh/?id=11031824086            | privilegeWithdrawn | 2023-11-08      |
| https://crt.sh/?id=11031824053            | privilegeWithdrawn | 2023-11-08      |
| https://crt.sh/?id=10810579104            | privilegeWithdrawn | 2023-10-17      |
| https://crt.sh/?id=10810577035            | unspecified (0）   | 2023-10-17      |
| https://crt.sh/?id=12421772198（Expired） |                    |                 |
| https://crt.sh/?id=12573150826            | privilegeWithdrawn | 2024-04-02      |
| https://crt.sh/?id=12421770452            | privilegeWithdrawn | 2024-04-02      |
| https://crt.sh/?id=12421770586            | superseded         | 2024-06-17      |
| https://crt.sh/?id=11917833535            | superseded         | 2024-06-17      |
| https://crt.sh/?id=11044872118            | privilegeWithdrawn | 2024-06-14      |
