# AWS SDK — Developer Guide (Python/Boto3, Node.js, Java, .NET)

> A step‑by‑step guide for installing and using the AWS Software Development Kits (SDKs). Focused examples for **Python (Boto3)** with cross‑platform installation instructions (Linux, macOS, Windows). Brief guidance for other popular SDKs (Node.js, Java, and .NET) and complete sample scripts for creating an EC2 key pair and a Security Group.

---

## Table of Contents

1. [Overview](#overview)
2. [Supported languages & SDKs](#supported-languages--sdks)
3. [Prerequisites](#prerequisites)
4. [Install & verify (by OS)](#install--verify-by-os)
   - [Python (Boto3)](#python-boto3)
   - [Node.js (aws-sdk v3)](#nodejs-awssdk-v3)
   - [Java / .NET / Go (notes)](#java--net--go-notes)
5. [Configure credentials (all OS)](#configure-credentials)
6. [Python examples (Boto3) — list regions, create key pair, create security group](#Python-examples-(Boto3)-list-regions-create-keypair-create-security-group)
   - [list_regions.py](#list_regionspy)
   - [create_keypair_and_sg.py](#create_keypair_and_sgpy)
7. [Windows-specific notes](#windows-specific-notes)
8. [macOS & Linux-specific notes](#macos--linux-specific-notes)
9. [Error handling & troubleshooting](#error-handling--troubleshooting)
10. [Best practices](#best-practices)
11. [Appendix: example outputs & file paths](#appendix)

---

## 1. Overview

AWS SDKs provide native libraries that simplify calling AWS APIs from your applications. This guide emphasizes **Boto3 (Python)** because it is commonly used for automation and scripting, and provides full cross‑platform instructions and examples. Equivalent instructions for Node.js (aws-sdk v3), Java, and .NET are summarized.

---

## 2. Supported languages & SDKs

- Python — **boto3** (recommended latest stable)
- JavaScript/TypeScript — **@aws-sdk/client-*** (v3 modular packages)
- Java — AWS SDK for Java v2
- .NET — AWS SDK for .NET
- Go, Ruby, PHP, C++, and more — see official AWS SDK pages for platform specifics

---

## 3. Prerequisites

- An AWS account and an IAM user/role with programmatic access (Access Key ID & Secret Access Key), or SSO/STS access.
- For system installs: admin or sudo privileges.
- For Python: Python 3.8+ recommended.
- For Node.js: Node 16+ recommended.
- Network access to AWS endpoints (or configured proxies for corporate networks).

---

## 4. Install & verify (by OS)

### Python (Boto3)

**Linux (Ubuntu/Debian)**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-venv python3-pip unzip curl

# Create & activate a virtualenv (recommended)
python3 -m venv ~/.venvs/aws-sdk
source ~/.venvs/aws-sdk/bin/activate
pip install --upgrade pip
pip install boto3
python3 -c "import boto3; print('boto3', boto3.__version__)"
```

**RHEL / CentOS / Amazon Linux**

```bash
sudo yum update -y
sudo yum install -y python3 python3-venv python3-pip unzip curl
python3 -m venv ~/.venvs/aws-sdk
source ~/.venvs/aws-sdk/bin/activate
pip install --upgrade pip
pip install boto3
```

**macOS (Homebrew)**

```bash
# If Python is missing
brew update
brew install python
python3 -m venv ~/.venvs/aws-sdk
source ~/.venvs/aws-sdk/bin/activate
pip install --upgrade pip
pip install boto3
python3 -c "import boto3; print('boto3', boto3.__version__)"
```

**Windows (PowerShell)**

```powershell
# Open PowerShell (recommended: run as Administrator for system installs)
python -m venv $env:USERPROFILE\.venvs\aws-sdk
$env:USERPROFILE\.venvs\aws-sdk\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install boto3
python -c "import boto3; print('boto3', boto3.__version__)"
```

> Using a virtual environment isolates dependencies per project — strongly recommended.

---

### Node.js (aws-sdk v3) — quick install

**Install Node.js (if needed)**
- macOS: `brew install node`
- Linux: use the distro package manager or nvm
- Windows: install from https://nodejs.org or use nvm-windows

**Create project & install modular v3 clients**

```bash
mkdir aws-node && cd aws-node
npm init -y
npm install @aws-sdk/client-ec2
```

**Verify** (simple script)

```js
// list-regions.js
const { DescribeRegionsCommand, EC2Client } = require("@aws-sdk/client-ec2");
async function main(){
  const client = new EC2Client({});
  const cmd = new DescribeRegionsCommand({});
  const res = await client.send(cmd);
  console.log(res.Regions.map(r=>r.RegionName));
}
main();
```

Run: `node list-regions.js`

---

### Java / .NET / Go (notes)

- Java: Use AWS SDK for Java v2 via Maven/Gradle. Example artifact: `software.amazon.awssdk:ec2`.
- .NET: Use NuGet package `AWSSDK.EC2` or AWS SDK for .NET v3 as appropriate.
- Go: `aws-sdk-go-v2` modules are preferred.

Refer to the official AWS SDK docs for language-specific project setup and examples.

---

## 5. Configure credentials (all OS)

### Option A — AWS CLI interactive (recommended)

```bash
aws configure
```

This prompts for Access Key ID, Secret Access Key, default region and output format and writes them to `~/.aws/credentials` and `~/.aws/config` (Windows: `%UserProfile%\.aws\`).

### Option B — Environment variables (session only)

**Linux/macOS (bash)**

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_DEFAULT_REGION="us-east-1"
```

**Windows PowerShell**

```powershell
$Env:AWS_ACCESS_KEY_ID = "AKIA..."
$Env:AWS_SECRET_ACCESS_KEY = "..."
$Env:AWS_DEFAULT_REGION = "us-east-1"
```

### Option C — Shared credentials & named profiles

`~/.aws/credentials` (example)

```ini
[default]
aws_access_key_id = AKIA...
aws_secret_access_key = ...

[dev]
aws_access_key_id = AKIADEV...
aws_secret_access_key = ...
```

Use a profile in Boto3 by creating a session:

```python
import boto3
session = boto3.Session(profile_name='dev')
ec2 = session.client('ec2')
```

---

## 6. Python examples (Boto3)

Below are two ready-to-run Python scripts: `list_regions.py` and `create_keypair_and_sg.py`.

> Save them into your project directory and run from an activated virtualenv.

### list_regions.py

```python
#!/usr/bin/env python3
"""List available EC2 regions using boto3"""
import boto3

def main():
    ec2 = boto3.client('ec2')
    resp = ec2.describe_regions()
    regions = [r['RegionName'] for r in resp['Regions']]
    print('Available AWS Regions:')
    for r in regions:
        print('-', r)

if __name__ == '__main__':
    main()
```

Run: `python3 list_regions.py`

---

### create_keypair_and_sg.py

This script does the following:
1. Determines the default VPC
2. Creates an EC2 key pair and saves the private key to `key.pem` (with secure permissions)
3. Creates a Security Group in the default VPC
4. Adds ingress rules for SSH (restricted to your public IP) and HTTP/HTTPS (0.0.0.0/0)

```python
#!/usr/bin/env python3
"""Create an EC2 key pair and security group using boto3.

Caveats:
- Requires AWS credentials configured (aws configure or env vars).
- Uses an external service (https://api.ipify.org) to detect your public IP for SSH restriction.
"""
import os
import stat
import sys
import urllib.request

import boto3
from botocore.exceptions import ClientError

KEY_NAME = 'MySDKKey'
KEY_FILE = 'MySDKKey.pem'
SG_NAME = 'MySDKSecurityGroup'
SG_DESC = 'Created by create_keypair_and_sg.py'


def get_public_ip():
    try:
        with urllib.request.urlopen('https://api.ipify.org') as r:
            ip = r.read().decode('utf-8').strip()
            return ip
    except Exception as e:
        print('Warning: could not determine public IP automatically:', e)
        return None


def save_key_material(key_material, filename=KEY_FILE):
    with open(filename, 'w', encoding='utf-8') as f:
        f.write(key_material)
    # Set permissions to 400 (owner read only)
    os.chmod(filename, stat.S_IRUSR)


def main():
    ec2 = boto3.client('ec2')

    # 1) Create or get key pair
    try:
        print(f'Creating key pair {KEY_NAME}...')
        resp = ec2.create_key_pair(KeyName=KEY_NAME, KeyType='rsa', KeyFormat='pem')
        save_key_material(resp['KeyMaterial'], KEY_FILE)
        print(f'Private key saved to {KEY_FILE}')
    except ClientError as e:
        if e.response['Error']['Code'] == 'InvalidKeyPair.Duplicate':
            print(f'Key pair {KEY_NAME} already exists. Skipping creation.')
        else:
            print('Error creating key pair:', e)
            sys.exit(1)

    # 2) Find default VPC
    try:
        vpcs = ec2.describe_vpcs(Filters=[{'Name': 'isDefault', 'Values': ['true']}])
        if not vpcs['Vpcs']:
            print('No default VPC found. Exiting.')
            sys.exit(1)
        vpc_id = vpcs['Vpcs'][0]['VpcId']
        print('Default VPC:', vpc_id)
    except ClientError as e:
        print('Error describing VPCs:', e)
        sys.exit(1)

    # 3) Create security group
    try:
        print(f'Creating security group {SG_NAME}...')
        sg = ec2.create_security_group(GroupName=SG_NAME, Description=SG_DESC, VpcId=vpc_id)
        sg_id = sg['GroupId']
        print('Created security group:', sg_id)
    except ClientError as e:
        if e.response['Error']['Code'] == 'InvalidGroup.Duplicate':
            print(f'Security group {SG_NAME} already exists. Finding its ID...')
            groups = ec2.describe_security_groups(Filters=[{'Name': 'group-name', 'Values': [SG_NAME]}])
            sg_id = groups['SecurityGroups'][0]['GroupId']
            print('Found security group:', sg_id)
        else:
            print('Error creating security group:', e)
            sys.exit(1)

    # 4) Add ingress rules
    try:
        public_ip = get_public_ip()
        ssh_cidr = f'{public_ip}/32' if public_ip else '0.0.0.0/0'
        print('Authorizing ingress rules:')
        print(' - SSH (port 22) from', ssh_cidr)
        print(' - HTTP (port 80) from 0.0.0.0/0')
        print(' - HTTPS (port 443) from 0.0.0.0/0')

        ec2.authorize_security_group_ingress(
            GroupId=sg_id,
            IpPermissions=[
                {'IpProtocol': 'tcp', 'FromPort': 22, 'ToPort': 22, 'IpRanges': [{'CidrIp': ssh_cidr, 'Description': 'SSH access'}]},
                {'IpProtocol': 'tcp', 'FromPort': 80, 'ToPort': 80, 'IpRanges': [{'CidrIp': '0.0.0.0/0', 'Description': 'HTTP'}]},
                {'IpProtocol': 'tcp', 'FromPort': 443, 'ToPort': 443, 'IpRanges': [{'CidrIp': '0.0.0.0/0', 'Description': 'HTTPS'}]},
            ]
        )
        print('Ingress rules added successfully.')
    except ClientError as e:
        if e.response['Error']['Code'] == 'InvalidPermission.Duplicate':
            print('Some or all ingress rules already exist. Continuing.')
        else:
            print('Error authorizing ingress:', e)
            sys.exit(1)

    print('\nDone. Key file:', KEY_FILE, 'Security Group ID:', sg_id)

if __name__ == '__main__':
    main()
```

Run: `python3 create_keypair_and_sg.py`

Notes:
- If you do not want the script to call an external service to detect your public IP, manually set `ssh_cidr` to `"x.x.x.x/32"`.
- On Windows, file permission modification uses different semantics — PowerShell `icacls` can be used when necessary.

---

## 7. Windows-specific notes

- When creating virtual environments, prefer PowerShell and ensure execution policy allows script activation: `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` (run as Administrator if required).
- If Python installer did not add Python to PATH, select "Modify" from the installer or re-run the installer and check "Add Python to PATH".
- Use `Activate.ps1` to activate venv in PowerShell, or `Activate.bat` in cmd.exe.

---

## 8. macOS & Linux-specific notes

- Prefer virtualenv or `pyenv` to avoid system Python conflicts.
- On macOS Apple Silicon, use homebrew-installed Python (ARM-native) or run pip in the desired architecture.
- Install `jq` if you need to parse JSON from CLI outputs in scripts.

---

## 9. Error handling & troubleshooting

- **`ModuleNotFoundError: No module named 'boto3'`**
  - Ensure virtualenv is activated and `pip install boto3` succeeded.

- **AccessDenied / InvalidClientTokenId**
  - Confirm credentials, check that keys are active and not rotated. Use `aws sts get-caller-identity` to verify identity.

- **Network errors when calling `api.ipify.org`**
  - If your network blocks external calls, skip automatic public IP detection and supply SSH CIDR manually.

- **Permission errors when writing key file**
  - Ensure script runs with a user that can write to the working directory; set secure permissions after creating the file.

---

## 10. Best practices

- Use IAM roles (EC2 IAM role or IAM role assumption) instead of long‑lived access keys where possible.
- Use named profiles for multiple accounts/environments.
- Never commit credentials or private keys into source control.
- Rotate credentials regularly and delete unused keys.
- Restrict security group SSH access to known IPs, not `0.0.0.0/0`.

---

## 11. Appendix: example outputs & file paths

- Credentials/config paths:
  - Linux/macOS: `~/.aws/credentials`, `~/.aws/config`
  - Windows: `%UserProfile%\\.aws\\credentials`, `%UserProfile%\\.aws\\config`

- Example: `python3 -c "import boto3; print(boto3.__version__)"` prints the installed boto3 version.

---

