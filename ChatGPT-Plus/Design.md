# 📄 File Upload Architecture (Mermaid.js)

## Architecture Description

- Web users request pre-signed URLs through an API Gateway, which forwards the request to a backend Pre-signed URL Service.
- The service validates the file type and user ID before generating the pre-signed URL.
- Upon successful validation, a pre-signed PUT URL is issued to the client.
- Users upload files directly to a designated S3 bucket, following the `uploads/{userId}/{timestamp}` structure.
- Amazon GuardDuty automatically scans new uploads for malware.
- Clean files are tagged as "CLEAN," and optionally a notification is sent to an SNS topic if the caller requested it.
- Malicious files are tagged as "MALICIOUS" and optionally moved to a quarantine bucket.
- Lifecycle rules on the S3 bucket ensure files are automatically deleted after 90 days.
- All pre-signed URL generation events are logged for auditing purposes.

## Architecture Diagram

```mermaid
graph TD

A[Web User] -->|Request pre-signed URL| B(API Gateway)
B --> C[Pre-signed URL Service]
C --> D{Validate file type & user ID}
D -- Valid --> E[Generate pre-signed PUT URL]
D -- Invalid --> F[Return error: Unsupported file type]

E --> G[Client uploads file to S3 Bucket uploads/userId/timestamp]

G --> H[GuardDuty Malware Scan]
H -->|No threats| I[Tag object as CLEAN]
H -->|Threat detected| J[Tag object as MALICIOUS / Move to quarantine bucket]

I --> K{SNS Topic provided?}
K -- Yes --> L[Publish success notification to SNS]
K -- No --> M[No action]

subgraph S3 Bucket Setup
  G
  N[Lifecycle Rule: Delete after 90 days]
end

subgraph Logging
  C --> O[Log pre-signed URL generation: user ID, file name, timestamp]
end
```

## Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Web User
    participant API as API Gateway
    participant Service as Pre-signed URL Service
    participant S3 as S3 Bucket
    participant GD as GuardDuty Malware Scan
    participant SNS as SNS Topic (optional)

    User ->> API: Request pre-signed URL
    API ->> Service: Forward request
    Service ->> Service: Validate file type and user ID
    Service -->> API: Return pre-signed URL
    API -->> User: Deliver pre-signed URL
    User ->> S3: Upload file using pre-signed URL
    S3 ->> GD: Trigger malware scan
    GD -->> S3: Tag as CLEAN or MALICIOUS
    GD -->> SNS: Publish notification (if topic provided)
```
