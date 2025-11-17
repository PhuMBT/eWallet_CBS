# eKYC Integration Guide - Hướng dẫn Tích hợp eKYC

## Tổng quan

Tài liệu này hướng dẫn tích hợp dịch vụ eKYC (electronic Know Your Customer) cho hệ thống Core Banking và eWallet.

---

## ⚠️ LƯU Ý QUAN TRỌNG

### ❌ KHÔNG kết nối trực tiếp với Bộ Công an

**Bank/Ví KHÔNG kết nối trực tiếp** đến database của Bộ Công an Việt Nam.

**Lý do:**

1. **Quy định pháp luật & bảo mật**
   - Database quốc gia chứa thông tin nhạy cảm
   - Cần cấp phép đặc biệt để truy cập
   - Chỉ các đối tác được cấp phép mới có quyền kết nối

2. **Cơ sở hạ tầng & kỹ thuật**
   - API của Bộ Công an có hạn chế về capacity
   - Cần whitelist IP, VPN chuyên dụng
   - Yêu cầu security certification (ISO 27001, etc.)

3. **Chi phí & thời gian triển khai**
   - Thủ tục xin cấp phép phức tạp (6-12 tháng)
   - Chi phí setup cao (hạ tầng, bảo mật, compliance)
   - Maintenance cost cao

4. **Best Practice**
   - Sử dụng eKYC Service Providers đã được cấp phép
   - Providers có SLA và support tốt hơn
   - Easier integration (RESTful API, SDK)

---

## ✅ Mô hình Tích hợp Đúng

### Architecture Pattern

```
┌──────────────────────┐
│   Bank / eWallet     │
│   Core Banking       │
└──────────┬───────────┘
           │
           │ RESTful API / SDK
           ▼
┌──────────────────────┐
│  eKYC Service        │
│  Provider            │
│  (VNPT/FPT/Viettel)  │
└──────────┬───────────┘
           │
           │ Secure Connection
           │ (Leased line / VPN)
           │ (Licensed Partner)
           ▼
┌──────────────────────┐
│  Database Quốc gia   │
│  Bộ Công an          │
│  CCCD/CMND Data      │
└──────────────────────┘
```

### Giải thích Flow

1. **Bank/Wallet** gửi request eKYC verification đến **eKYC Provider**
2. **eKYC Provider** xử lý request:
   - OCR extraction từ ảnh CCCD
   - Facial recognition
   - Liveness detection
3. **eKYC Provider** kết nối đến **Bộ Công an** để verify:
   - Số CCCD có tồn tại không?
   - Thông tin có khớp không? (Họ tên, ngày sinh, địa chỉ)
   - Ảnh CCCD có khớp không?
4. **eKYC Provider** trả về kết quả cho **Bank/Wallet**:
   - Verification status (PASSED/FAILED)
   - Extracted information
   - Confidence score

---

## eKYC Service Providers tại Việt Nam

### 1. VNPT eKYC

**Provider:** VNPT (Vietnam Posts and Telecommunications Group)

**Tính năng:**
- ✅ OCR CCCD/CMND/Passport
- ✅ Facial Recognition
- ✅ Liveness Detection
- ✅ National ID verification (kết nối Bộ Công an)
- ✅ Video call eKYC

**Integration:**
- RESTful API
- SDK (iOS, Android, Web)
- Webhook callbacks

**Website:** https://ekyc.vnpt.vn/

**Pricing:** Contact for pricing

**SLA:** 99.9% uptime

---

### 2. FPT eKYC (FPT.AI)

**Provider:** FPT Corporation

**Tính năng:**
- ✅ OCR CCCD/CMND/Passport/Driver License
- ✅ Face Recognition + Anti-spoofing
- ✅ Liveness Detection (blink, turn head)
- ✅ National ID verification
- ✅ Video KYC

**Integration:**
- RESTful API
- SDK (iOS, Android, React Native, Flutter)
- On-premise deployment option

**Website:** https://ekyc.fpt.ai/

**Pricing:** Pay-per-use or monthly package

**SLA:** 99.9% uptime, response time < 3s

---

### 3. Viettel eKYC

**Provider:** Viettel Group

**Tính năng:**
- ✅ OCR giấy tờ tùy thân
- ✅ Face matching
- ✅ Liveness detection
- ✅ National database verification
- ✅ Background check

**Integration:**
- RESTful API
- SDK available
- VPN connection option

**Website:** Contact Viettel for details

**Pricing:** Enterprise pricing

**SLA:** 99.9% uptime

---

### 4. Các Providers Khác

**CMC eKYC:**
- OCR + Face recognition
- National ID verification
- https://cmctelecom.vn

**MISA eKYC:**
- Accounting software với tích hợp eKYC
- Suitable for SME

**VinCSS (VinGroup):**
- Cyber security + eKYC
- Enterprise solution

---

## API Integration Example

### 1. Request eKYC Verification

**Endpoint:** `POST https://api.ekyc-provider.com/v1/verify`

**Headers:**
```http
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

**Request Body:**
```json
{
  "requestId": "REQ_20250117_001234",
  "idType": "CCCD",
  "frontImage": "base64_encoded_front_image",
  "backImage": "base64_encoded_back_image",
  "faceImage": "base64_encoded_selfie",
  "livenessVideo": "base64_encoded_video_optional",
  "metadata": {
    "ipAddress": "1.2.3.4",
    "deviceId": "DEVICE_12345",
    "timestamp": "2025-01-17T10:30:00Z"
  }
}
```

**Response (Success):**
```json
{
  "status": "SUCCESS",
  "requestId": "REQ_20250117_001234",
  "verificationId": "VER_20250117_001234",
  "result": {
    "idVerification": "PASSED",
    "faceMatch": "PASSED",
    "livenessCheck": "PASSED",
    "nationalDbCheck": "PASSED"
  },
  "extractedData": {
    "idNumber": "001234567890",
    "fullName": "NGUYEN VAN A",
    "dateOfBirth": "01/01/1990",
    "gender": "Nam",
    "nationality": "Việt Nam",
    "placeOfOrigin": "Hà Nội",
    "placeOfResidence": "123 Đường ABC, Quận XYZ, Hà Nội",
    "issueDate": "01/01/2020",
    "expiryDate": "01/01/2030"
  },
  "confidenceScores": {
    "ocrConfidence": 0.98,
    "faceMatchScore": 0.95,
    "livenessScore": 0.97,
    "overallScore": 0.96
  },
  "timestamp": "2025-01-17T10:30:05Z"
}
```

**Response (Failed):**
```json
{
  "status": "FAILED",
  "requestId": "REQ_20250117_001234",
  "verificationId": "VER_20250117_001234",
  "result": {
    "idVerification": "FAILED",
    "faceMatch": "PASSED",
    "livenessCheck": "PASSED",
    "nationalDbCheck": "FAILED"
  },
  "failureReason": {
    "code": "ID_NOT_FOUND",
    "message": "ID number not found in national database",
    "details": "The provided ID number (001234567890) does not exist in the national citizen database"
  },
  "timestamp": "2025-01-17T10:30:05Z"
}
```

---

### 2. Check Verification Status

**Endpoint:** `GET https://api.ekyc-provider.com/v1/verify/{verificationId}`

**Response:**
```json
{
  "verificationId": "VER_20250117_001234",
  "status": "COMPLETED",
  "result": "PASSED",
  "timestamp": "2025-01-17T10:30:05Z"
}
```

---

## Implementation Best Practices

### 1. ✅ Multi-Provider Strategy

**Khuyến nghị:** Tích hợp 2-3 providers để đảm bảo availability

**Lý do:**
- Nếu provider chính down, có thể failover
- So sánh kết quả từ nhiều provider cho độ chính xác cao
- Negotiate better pricing

**Implementation:**
```typescript
interface EKYCProvider {
  name: string;
  priority: number;
  verify(request: EKYCRequest): Promise<EKYCResult>;
}

const providers: EKYCProvider[] = [
  { name: 'VNPT', priority: 1, verify: vnptVerify },
  { name: 'FPT', priority: 2, verify: fptVerify },
  { name: 'Viettel', priority: 3, verify: viettelVerify }
];

async function verifyWithFallback(request: EKYCRequest): Promise<EKYCResult> {
  for (const provider of providers.sort((a, b) => a.priority - b.priority)) {
    try {
      const result = await provider.verify(request);
      if (result.status === 'SUCCESS') {
        return result;
      }
    } catch (error) {
      console.error(`Provider ${provider.name} failed:`, error);
      // Continue to next provider
    }
  }
  throw new Error('All eKYC providers failed');
}
```

---

### 2. ✅ Caching & Rate Limiting

**Cache verified IDs:**
```typescript
// Cache ID verification result for 30 days
const cacheKey = `ekyc:${idNumber}`;
const cachedResult = await redis.get(cacheKey);

if (cachedResult) {
  return JSON.parse(cachedResult);
}

const result = await eKYCProvider.verify(request);

if (result.status === 'SUCCESS') {
  await redis.setex(cacheKey, 30 * 24 * 3600, JSON.stringify(result));
}

return result;
```

**Rate limiting:**
```typescript
// Limit to 100 eKYC requests per user per day
const rateLimitKey = `ekyc:ratelimit:${userId}`;
const count = await redis.incr(rateLimitKey);

if (count === 1) {
  await redis.expire(rateLimitKey, 24 * 3600);
}

if (count > 100) {
  throw new Error('Daily eKYC limit exceeded');
}
```

---

### 3. ✅ Error Handling & Retry

**Retry transient errors:**
```typescript
async function verifyWithRetry(
  request: EKYCRequest,
  maxRetries: number = 3
): Promise<EKYCResult> {
  let lastError: Error;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await eKYCProvider.verify(request);
    } catch (error) {
      lastError = error;
      
      if (isTransientError(error)) {
        // Network timeout, temporary unavailable
        await sleep(Math.pow(2, attempt) * 1000); // Exponential backoff
        continue;
      } else {
        // Permanent error (invalid API key, bad request)
        throw error;
      }
    }
  }
  
  throw new Error(`eKYC verification failed after ${maxRetries} retries: ${lastError.message}`);
}

function isTransientError(error: any): boolean {
  return error.code === 'TIMEOUT' 
    || error.code === 'NETWORK_ERROR'
    || error.statusCode === 503; // Service Unavailable
}
```

---

### 4. ✅ Security Best Practices

**Encrypt sensitive data:**
```typescript
// Encrypt ID images before sending to provider
const encryptedFrontImage = await encryptAES256(frontImage);
const encryptedBackImage = await encryptAES256(backImage);

const result = await eKYCProvider.verify({
  frontImage: encryptedFrontImage,
  backImage: encryptedBackImage,
  encryption: 'AES-256-GCM'
});
```

**Log audit trail:**
```typescript
await auditLog.create({
  userId: user.id,
  action: 'EKYC_VERIFICATION',
  provider: 'VNPT',
  requestId: request.id,
  result: result.status,
  timestamp: new Date(),
  ipAddress: request.ipAddress,
  metadata: {
    idNumber: maskIDNumber(result.extractedData.idNumber), // 001234***890
    confidenceScore: result.confidenceScores.overallScore
  }
});
```

**Mask sensitive data in logs:**
```typescript
function maskIDNumber(idNumber: string): string {
  if (idNumber.length <= 6) return '***';
  return idNumber.substring(0, 6) + '***' + idNumber.substring(idNumber.length - 3);
}

// Example: 001234567890 → 001234***890
```

---

### 5. ✅ Monitoring & Alerting

**Track provider performance:**
```typescript
// Metrics to track
const metrics = {
  ekyc_requests_total: counter,
  ekyc_success_rate: gauge,
  ekyc_response_time: histogram,
  ekyc_failure_by_provider: counter,
  ekyc_cost_by_provider: counter
};

// Alert conditions
if (successRate < 0.95) {
  alert('eKYC success rate below 95%');
}

if (avgResponseTime > 5000) {
  alert('eKYC response time above 5 seconds');
}
```

**Dashboard example:**
```
eKYC Provider Performance (Last 24h)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Provider   | Requests | Success | Avg Time | Cost
─────────────────────────────────────────────
VNPT       | 1,234    | 98.2%   | 2.3s     | $123
FPT        | 567      | 99.1%   | 1.8s     | $85
Viettel    | 89       | 97.8%   | 3.1s     | $18
─────────────────────────────────────────────
TOTAL      | 1,890    | 98.5%   | 2.2s     | $226
```

---

## Cost Optimization

### Pricing Comparison (Example)

| Provider | Price per Request | Volume Discount | Annual Contract |
|----------|------------------|----------------|-----------------|
| VNPT     | $0.10            | 10% (>10K)     | 20% off         |
| FPT      | $0.15            | 15% (>20K)     | 25% off         |
| Viettel  | $0.20            | 20% (>50K)     | 30% off         |

### Optimization Strategies

1. **Batch Processing**
   - Group requests to get volume discount
   - Negotiate monthly/annual packages

2. **Smart Routing**
   - Route based on cost and performance
   - Use cheaper provider for low-risk customers

3. **Caching**
   - Cache verified results
   - Avoid duplicate verification

4. **Conditional Verification**
   - Level 1: No eKYC (free)
   - Level 2: Basic eKYC ($0.10)
   - Level 3+: Enhanced eKYC ($0.20)

---

## Compliance & Legal

### 1. Data Protection (PDPA)

**Lưu trữ dữ liệu cá nhân:**
- ✅ Chỉ lưu dữ liệu cần thiết
- ✅ Encrypt at rest và in transit
- ✅ Retention policy (xóa sau 7 năm theo quy định ngân hàng)
- ✅ Customer consent required

**Ví dụ consent:**
```
"Tôi đồng ý cho [Tên Ngân hàng/Ví] sử dụng thông tin CCCD 
của tôi để xác thực danh tính thông qua dịch vụ eKYC của 
[VNPT/FPT/Viettel]. Thông tin sẽ được bảo mật và chỉ sử 
dụng cho mục đích KYC theo quy định pháp luật."
```

---

### 2. Regulatory Compliance

**Thông tư 40/2024/TT-NHNN:**
- Bắt buộc eKYC cho ví điện tử Level 2+
- Sinh trắc học bắt buộc từ 01/01/2026

**Luật An ninh mạng:**
- Dữ liệu công dân phải lưu tại Việt Nam
- eKYC provider phải có server tại VN

**Quy định AML/CFT:**
- Lưu trữ hồ sơ KYC tối thiểu 5 năm
- Báo cáo giao dịch đáng ngờ

---

## Testing & QA

### 1. Test Cases

**Positive Cases:**
- ✅ Valid CCCD → Should pass
- ✅ Valid Passport → Should pass
- ✅ Good quality images → High confidence

**Negative Cases:**
- ❌ Blurry images → Should fail OCR
- ❌ Fake ID → Should fail verification
- ❌ Face mismatch → Should fail face matching
- ❌ Expired ID → Should flag warning

**Edge Cases:**
- 🟡 Low quality but readable → Lower confidence
- 🟡 Similar but not exact match → Risk score
- 🟡 Provider timeout → Retry mechanism

---

### 2. Test Environment

**Sandbox Credentials:**
```typescript
const TEST_CONFIG = {
  vnpt: {
    apiKey: 'test_vnpt_key_12345',
    endpoint: 'https://sandbox.ekyc.vnpt.vn/v1'
  },
  fpt: {
    apiKey: 'test_fpt_key_67890',
    endpoint: 'https://sandbox.ekyc.fpt.ai/v1'
  }
};
```

**Test Data:**
- Providers thường cung cấp test CCCD numbers
- Example: `000000000000` → Always pass
- Example: `999999999999` → Always fail

---

## FAQ

### Q1: Có thể tự xây dựng eKYC system không?

**A:** Về mặt kỹ thuật là có thể, nhưng KHÔNG KHUYẾN NGHỊ vì:
- ❌ Không thể kết nối trực tiếp Bộ Công an (cần cấp phép)
- ❌ Chi phí AI model training rất cao (Face recognition, OCR)
- ❌ Maintenance và update khó khăn
- ❌ Không có SLA và support
- ✅ Use existing providers (faster, cheaper, more reliable)

---

### Q2: Provider nào tốt nhất?

**A:** Phụ thuộc vào use case:
- **VNPT**: Government connections, reliable, good pricing
- **FPT**: Advanced AI, fast response, good SDK
- **Viettel**: Enterprise solution, on-premise option

**Khuyến nghị:** Tích hợp 2-3 providers cho redundancy

---

### Q3: Làm sao để giảm chi phí eKYC?

**A:** 
1. ✅ Cache kết quả verification (30 days)
2. ✅ Batch processing để được volume discount
3. ✅ Smart routing (dùng provider rẻ hơn cho low-risk)
4. ✅ Annual contract với discount
5. ✅ Chỉ verify khi thực sự cần (Level 2+)

---

### Q4: eKYC có thể bị fake không?

**A:** Có, nhưng eKYC providers có nhiều lớp bảo vệ:
- ✅ Liveness detection (chống ảnh/video giả)
- ✅ Anti-spoofing (chống deepfake)
- ✅ Verify với database Bộ Công an
- ✅ Risk scoring

**Best practice:** 
- Kết hợp eKYC + AML screening
- Manual review cho high-risk cases
- Transaction monitoring

---

### Q5: Thời gian xử lý eKYC bao lâu?

**A:** 
- OCR + Face recognition: 1-3 seconds
- National database verification: 2-5 seconds
- Total: 3-8 seconds average

**Optimization:**
- Async processing (không block user)
- Webhook callbacks
- Progress indicator

---

## Kết luận

### ✅ Best Practices Summary

1. **KHÔNG kết nối trực tiếp Bộ Công an** → Use eKYC Providers
2. **Multi-provider strategy** → VNPT + FPT + Viettel
3. **Caching & rate limiting** → Optimize cost và performance
4. **Error handling & retry** → Improve success rate
5. **Security & compliance** → PDPA, AML/CFT
6. **Monitoring & alerting** → Track provider performance
7. **Testing thoroughly** → Positive, negative, edge cases

---

### 📚 Resources

**Official Documentation:**
- VNPT eKYC: https://ekyc.vnpt.vn/docs
- FPT eKYC: https://ekyc.fpt.ai/docs
- Viettel: Contact for documentation

**Regulations:**
- Thông tư 40/2024/TT-NHNN: https://thuvienphapluat.vn/van-ban/Tien-te-Ngan-hang/Thong-tu-40-2024-TT-NHNN-huong-dan-hoat-dong-cung-ung-dich-vu-trung-gian-thanh-toan-615328.aspx
- Nghị định 52/2024/NĐ-CP: Thanh toán không dùng tiền mặt

**Support:**
- VNPT: support@ekyc.vnpt.vn
- FPT: support@fpt.ai
- Viettel: Contact through sales

---

**Ngày tạo:** 2025-01-17  
**Version:** 1.0  
**Tác giả:** AI Architecture Assistant  
**Dự án:** Masan eWallet CBS Documentation

