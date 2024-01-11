# CloudConcerns: Data Leakage through AMI Sharing

## Table of Contents
- [CloudConcerns: Data Leakage through AMI Sharing](#cloudconcerns-data-leakage-through-ami-sharing)
  - [Table of Contents](#table-of-contents)
  - [Table](#table)
  - [Prerequisites](#prerequisites)
  - [Modus Operandi](#modus-operandi)
    - [Scenario Diagram](#scenario-diagram)
    - [Scenario Script](#scenario-script)
  - [Scenario Analysis](#scenario-analysis)
    - [Technical Complexity](#technical-complexity)
    - [Data Volume Leaked](#data-volume-leaked)
  - [Detection](#detection)
    - [Vulnerability Detection](#vulnerability-detection)
    - [Leakage Detection](#leakage-detection)
  - [Mitigations](#mitigations)
  - [Exploitation Tooling](#exploitation-tooling)

## Table

| Parameter                | Description                                                         |
|--------------------------|---------------------------------------------------------------------|
| Name of the technique    | Data Leakage through AMI Sharing                                    |
| MITRE ATT&CK technique   | [T1537](https://attack.mitre.org/techniques/T1537/)                 |
| Impact type              | Data Leakage                                                        |
| Technical Complexity     | Low                                                                 |
| Data Volume Leaked       | High                                                                |
| Remediation Complexity   | High                                                                |

![MitreAtt&ckTechnique](/img/t1537.png)

## Prerequisites

- The attacker has sufficient permissions to modify AMI permissions.
- The attacker has sufficient permissions to create an EC2 instance and an AMI.

## Modus Operandi

### Scenario Diagram

![CloudConcerns Leakage AMI Sharing Diagram](cloudconcerns-leakage-ami.svg)

Refer to the [Risk](/risk-scale.md) scale page, for risk evaluation explanation.

### Scenario Script

An insider with access to data within an AWS Account has rights to create EC2 images or volumes. They transfer all the data they wish to leak to an external account.

After storing the desired data in the EC2, the insider creates an AMI from the EC2 and shares it with external parties. The AMI can be shared in different ways:
- Publicly: Making it accessible to all AWS accounts.
- With specific account IDs (12 digits): Sharing exclusively with individual accounts.
- With an AWS Organization ID: Making the AMI available to all accounts within the organization.

The potential volume of data leaked is nearly unlimited, making this scenario particularly efficient. Moreover, the AMI can include both technical and business data. Technical data leakage might include secrets/hashes, security configurations like hardening, vulnerabilities, etc. Leakage in the attacker's account allows them to analyze the entire system/application stack without leaving traces, as it's executed in the attacker's account.

## Scenario Analysis

### Technical Complexity

This scenario is relatively simple to execute, provided all prerequisites (i.e., necessary IAM permissions) are met.

### Data Volume Leaked

This technique is suited for a very high volume of data leakage. With one AMI, up to 64 TB can be exfiltrated ([AMI size limits](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ComponentsAMIs.html#storage-for-the-root-device)).

## Detection
### Vulnerability Detection

- Evaluate IAM permissions to detect if sensitive users can modify AMI permissions.
- Ensure that "Block public access for AMIs" is configured in the EC2 "Data protection and security" options.
- Modifications to AMI permissions will generate log records in CloudTrail. You should monitor such events.

### Leakage Detection

If an AMI is shared with another account, the only trace is the sharing configuration. We **cannot know** if an adversary **has effectively accessed or downloaded the AMI**. No log is generated in CloudTrail or any other logging system.

The only log record generated is from the configuration of the sharing, which would look something like this:

```json
{
    "eventVersion": "1.09",
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "AROAXXXXXX:anonymized",
        "arn": "arn:aws:sts::123456789012:assumed-role/AnonymizedRole/username",
        "accountId": "123456789012",
        "accessKeyId": "ASIAXXXXXX",
        "sessionContext": {
            "sessionIssuer": {
                "type": "Role",
                "principalId": "AROAXXXXXX",
                "arn": "arn:aws:iam::123456789012:role/anonymized-role",
                "accountId": "123456789012",
                "userName": "AnonymizedUserName"
            },
            "attributes": {
                "creationDate": "2024-01-09T07:40:47Z",
                "mfaAuthenticated": "false"
            }
        }
    },
    "eventTime": "2024-01-09T08:25:47Z",
    "eventSource": "ec2.amazonaws.com",
    "eventName": "ModifyImageAttribute",
    "awsRegion": "eu-west-1",
    "sourceIPAddress": "1.2.3.4",
    "userAgent": "AWS Internal",
    "requestParameters": {
        "imageId": "ami-XXXXXXXX",
        "launchPermission": {
            "add": {
                "items": [
                    {
                        "userId": "999999999999"
                    }
                ]
            }
        },
        "attributeType": "launchPermission"
    },
    "responseElements": {
        "requestId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
        "_return": true
    },
    "requestID": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
    "eventID": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
    "readOnly": false,
    "eventType": "AwsApiCall",
    "managementEvent": true,
    "recipientAccountId": "123456789012",
    "eventCategory": "Management",
    "sessionCredentialFromConsole": "true"
}
```
Notice the `userId` field that identify the AWS Account with which the AMI is shared. Detection scenarios should identify account IDs external to your AWS Organization.

## Mitigations

- Apply least privilege to users to deny them the ability to modify permissions on AMIs.
- Use SCPs on accounts to also protect against most admins. It will deny the ability to share any AMI with external accounts (be aware of operational impacts).
- Encrypt the AMI with a KMS Managed Key. To access the AMI, the attacker would need decryption rights on the KMS Key (adding complexity to the attack). Ensure all AMIs are encrypted (as enforceable rules are only detective, not protective).
- Configure "Block public access for AMIs" in the EC2 "Data protection and security" options and prevent any changes to this configuration. Deny all the following permissions: `ec2:EnableImageBlockPublicAccess`, `ec2:DisableImageBlockPublicAccess` (mandatory permission), `ec2:GetImageBlockPublicAccessState` (see [Block Public Access to AMIs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/sharingamis-intro.html#block-public-access-to-amis)).


## Exploitation Tooling

To create an AMI from an existing EC2 (replace i-xxxxxxxxxxxx with your instance ID):
```bash
aws ec2 create-image --instance-id i-xxxxxxxxxxxx --name "MyAMIName" --description "My AMI description" --no-reboot
```


Using the AWS CLI on Linux, to share an AMI (e.g., ami-777777777777) with the account 999999999999:

```bash
aws ec2 describe-image-attribute --image-id ami-777777777777 --launch-permission "Add=[{UserId=999999999999}]"

```







