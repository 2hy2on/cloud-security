# flaws.cloud Level 1

**Platform:** http://flaws.cloud  
**Category:** S3 Misconfiguration

## Vulnerability
S3 bucket was publicly accessible without authentication,
exposing all files inside the bucket.

## Steps
1. Checked that flaws.cloud is hosted on S3 via nslookup
2. Accessed bucket contents without credentials using AWS CLI

\```bash
aws s3 ls s3://flaws.cloud --no-sign-request
\```

3. Found hidden file `secret-dd02c7c.html` and accessed it directly

## Key Takeaway
Misconfigured S3 bucket permissions can expose sensitive files to anyone.

## Real-world Example
2019 Capital One Data Breach
- Attacker exploited misconfigured AWS WAF role
- 100 million+ customer records exposed
- Vulnerability was discovered via Responsible Disclosure Program
  on July 17, 2019
- Source: Capital One official statement

## How to Fix
- Set S3 bucket ACL to private
- Enable "Block Public Access" settings in AWS console
- Regularly audit bucket permissions using AWS Config