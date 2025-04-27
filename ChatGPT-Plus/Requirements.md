# 📄 File Upload Service – Requirements Document

### 1. Purpose
Design a secure, scalable file upload system where web users can upload files directly to S3 using pre-signed URLs. Files must be scanned for malware using AWS-native services before downstream processing.

---

### 2. Upload Characteristics
- **Supported file types:** `txt`, `pdf`, `xls`, image formats (e.g., `jpg`, `png`).
- **Max file size:** 100MB per upload.
- **File validation:** Both MIME type and file extension validated **upfront** when generating the pre-signed URL.
- **Unauthorized file types:** Return a clear error message ("Unsupported file type").

---

### 3. User Identity
- **Authentication:**  
  Users are not directly authenticated with our service but must provide their **user ID**.
- **Object organization in S3:**  
  Files are stored under the key structure:  
  ```
  uploads/{userId}/{timestamp}
  ```
- **Metadata:**  
  User ID will also be included in the file’s metadata.

---

### 4. S3 Bucket Setup
- **Bucket(s):** Dedicated or shared, but uploads by web users go into a specific structure for scanning.
- **Lifecycle policy:** Files automatically deleted **90 days** after upload.
- **Encryption:** Use **AWS S3 Default Encryption** (SSE-S3).
- **Object overwriting:**  
  Not allowed — keys must be unique.

---

### 5. Pre-signed URL Generation
- **Validity duration:**  
  Configurable by caller (up to 60 minutes maximum).
- **Access control:**  
  Service is protected behind an API Gateway, accessible only by trusted clients.
- **Audit logging:**  
  Log each pre-signed URL generation event, including:
  - User ID
  - File name
  - Timestamp
  - Requester IP (if available)

---

### 6. Malware Scanning
- **Service:**  
  Use **Amazon GuardDuty Malware Protection for S3**.
- **Scope:**  
  Only scan files uploaded via the web user flow (under `uploads/` prefix).
- **Post-scan action:**  
  - **Clean files:** Optionally send an SNS notification (if the caller requested an SNS topic when creating the pre-signed URL).
  - **Malicious files:** Tag or move to a quarantine bucket (separate cleanup workflow, not auto-deletion).

---

### 7. Optional Features
- **SNS Notification:**  
  If the caller specifies an SNS topic when requesting a pre-signed URL, post-scan success will publish a message to the given topic.
- **No forced object versioning.**

---

### 8. Failure Handling
- If a file is uploaded but never scanned (e.g., scan errors), lifecycle rules will eventually clean it up after 90 days.
- No retries or manual intervention unless explicitly designed later.
