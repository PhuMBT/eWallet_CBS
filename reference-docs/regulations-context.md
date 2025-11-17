# Regulations Context for Gen AI

> **Purpose**: This document provides regulatory context for AI assistants working on the Masan TTGT eWallet Core Banking System documentation project. All information here MUST be treated as authoritative and compliance is mandatory.

---

## 📋 Document Information

- **Last Updated**: November 17, 2024
- **Project**: Masan TTGT - eWallet Core Banking System
- **Compliance Status**: Active
- **Primary Regulation**: Thông tư 40/2024/TT-NHNN (Circular 40/2024/TT-NHNN)

---

## ⚖️ PRIMARY REGULATION: Thông tư 40/2024/TT-NHNN

### Basic Information

- **Full Name**: Thông tư 40/2024/TT-NHNN - Hướng dẫn hoạt động cung ứng dịch vụ trung gian thanh toán
- **English**: Circular 40/2024/TT-NHNN - Guidance on provision of payment intermediary services
- **Issued By**: Ngân hàng Nhà nước Việt Nam (State Bank of Vietnam)
- **Issue Date**: September 30, 2024
- **Effective Date**: October 1, 2024
- **Official Link**: https://thuvienphapluat.vn/van-ban/Tien-te-Ngan-hang/Thong-tu-40-2024-TT-NHNN-huong-dan-hoat-dong-cung-ung-dich-vu-trung-gian-thanh-toan-615328.aspx

### Key Articles for eWallet KYC

#### Article 26: Customer Identity Verification (Xác minh danh tính khách hàng)

**Requirements for Individual Customers:**
- Full name (Họ và tên)
- Date of birth (Ngày, tháng, năm sinh)
- Nationality (Quốc tịch)
- ID document number, issue date, issuing authority (Số, ngày cấp, nơi cấp giấy tờ tùy thân)
- Registered phone number (Số điện thoại đăng ký)
- Permanent or temporary address (Địa chỉ thường trú hoặc tạm trú)
- Email address (if available) (Địa chỉ thư điện tử)
- **MANDATORY**: Linked bank account or debit card (Tài khoản thanh toán hoặc thẻ ghi nợ tại ngân hàng liên kết)

**Verification Methods:**
- In-person: Direct comparison with original ID documents
- Online: Electronic verification methods compliant with AML regulations

#### Article 27: Biometric Verification (Xác thực sinh trắc học)

**From January 1, 2026**: Biometric authentication becomes MANDATORY when opening eWallets
- Facial recognition (Nhận diện khuôn mặt)
- Fingerprint (if available) (Vân tay)
- Liveness detection (Phát hiện sống)

---

## 💰 TRANSACTION LIMITS (Hạn mức giao dịch)

### Official Limits by KYC Level

| KYC Level | Description | Monthly Limit (VND) | Legal Basis |
|-----------|-------------|---------------------|-------------|
| **Level 1** | User Account Only | 0 (No wallet) | Internal policy |
| **Level 2** | eKYC Basic | **100,000,000** | TT 40/2024 Article 26 |
| **Level 3** | eKYC Advanced | **500,000,000** | TT 40/2024 Articles 26, 27 |
| **Level 4** | Full Verification | **UNLIMITED*** | TT 40/2024 Articles 26, 27 |

*\*Unlimited but must comply with risk management and AML regulations*

### Level 2: eKYC Basic (100 million VND/month)

**Requirements:**
```
✓ Valid ID document: CCCD/CMND/Passport
✓ Electronic identity verification (eKYC)
✓ Linked bank account (MANDATORY)
✓ Registered phone number
```

**Limits:**
- Monthly transaction limit: 100 million VND
- Balance limit: UNLIMITED
- Daily transaction limit: UNLIMITED
- Coverage: Total of transfers + payments

### Level 3: eKYC Advanced (500 million VND/month)

**Requirements:**
```
✓ All Level 2 requirements
✓ Enhanced biometric authentication (Facial Recognition, Fingerprint)
✓ Enhanced identity verification
✓ Supplementary information as requested
✓ AML Screening - Sanction list check
✓ From 01/01/2026: MANDATORY biometric authentication
```

**Limits:**
- Monthly transaction limit: 500 million VND
- Balance limit: UNLIMITED
- Daily transaction limit: UNLIMITED

### Level 4: Full Verification (UNLIMITED)

**Requirements:**
```
✓ All Level 3 requirements
✓ Full identity verification per AML regulations
✓ Manual review by Compliance Team
✓ Risk management assessment and approval
✓ May require verification at transaction point
```

**Limits:**
- Monthly transaction limit: UNLIMITED
- Balance limit: UNLIMITED
- Daily transaction limit: UNLIMITED
- **NOTE**: Must comply with risk management and AML regulations

---

## 🎯 TRANSACTION LIMIT EXCEPTIONS

**IMPORTANT**: Transaction limits DO NOT APPLY to the following transactions:

1. ✅ **Online payments on National Public Service Portal** (Thanh toán trực tuyến trên Cổng Dịch vụ công quốc gia)
2. ✅ **Utility payments**: Electricity, water, telecommunications (Thanh toán tiền điện, nước, viễn thông)
3. ✅ **Transportation fees and charges** (Thanh toán phí, giá dịch vụ liên quan đến giao thông đường bộ)
4. ✅ **Education and healthcare fees**: Tuition, hospital fees (Thanh toán học phí, viện phí)
5. ✅ **Insurance premiums**: Social insurance, health insurance, commercial insurance (Đóng bảo hiểm xã hội, bảo hiểm y tế, phí bảo hiểm)
6. ✅ **Debt payments to banks**: Principal, interest, and related fees (Chi trả nợ và lãi cho ngân hàng)
7. ✅ **Merchant eWallets**: Wallets of payment acceptance units (Ví điện tử của merchant - đơn vị chấp nhận thanh toán)

---

## 🔐 MANDATORY REQUIREMENTS

### Bank Account Linking (CRITICAL)

**For Level 2 and above**: Bank account linking is MANDATORY
- Must link to a valid Vietnamese bank account or debit card
- This is a legal requirement, not optional
- Without bank linking, customer cannot progress beyond Level 1

### Biometric Authentication Timeline

**Current (2024)**: Optional but recommended
**From January 1, 2026**: MANDATORY for all new eWallet registrations

---

## 🏗️ SYSTEM IMPLEMENTATION GUIDELINES

### For Level 1 (User Account)

```typescript
// DO NOT create CIF at Level 1
cifCreated: false

// Only allow app browsing
features: ['view_app', 'browse_products', 'view_promotions']

// No wallet, no transactions
limits: {
  maxBalance: 0,
  dailyTransaction: 0,
  monthlyTransaction: 0
}
```

**Rationale**: Prevent "junk" CIFs from users who only want to browse

### For Level 2 (eKYC Basic - 100M/month)

```typescript
// CREATE CIF when reaching Level 2
cifCreated: true

// MANDATORY bank linking
requirements: [
  'Valid ID: CCCD/CMND/Passport',
  'eKYC verification',
  'Bank account linking (MANDATORY)', // ⚠️ Critical
  'Phone number registration'
]

// Limits per TT 40/2024
limits: {
  maxBalance: Infinity,           // No balance limit
  dailyTransaction: Infinity,     // No daily limit
  monthlyTransaction: 100_000_000 // ⚖️ 100M VND/month (LAW)
}
```

### For Level 3 (eKYC Advanced - 500M/month)

```typescript
requirements: [
  'All Level 2 requirements',
  'Enhanced biometric authentication',
  'Enhanced identity verification',
  'AML Screening',
  'From 01/01/2026: MANDATORY biometric auth'
]

limits: {
  maxBalance: Infinity,
  dailyTransaction: Infinity,
  monthlyTransaction: 500_000_000 // ⚖️ 500M VND/month (LAW)
}
```

### For Level 4 (Full Verification - UNLIMITED)

```typescript
requirements: [
  'All Level 3 requirements',
  'Full identity verification per AML',
  'Manual compliance review',
  'Risk management approval',
  'May require in-person verification'
]

limits: {
  maxBalance: Infinity,
  dailyTransaction: Infinity,
  monthlyTransaction: Infinity // ⚖️ UNLIMITED (LAW)
}

note: 'Must comply with risk management and AML regulations'
```

---

## 🎯 MASAN-SPECIFIC EXTENSIONS

### Level 5: Enhanced Merchant (Extension - NOT in TT 40/2024)

**Purpose**: Support for offline merchants (NPP, NBL) needing high transaction limits

```typescript
// This is a Masan extension, not part of TT 40/2024
level: 5,
description: 'Enhanced Merchant Verification',
applicableTo: [
  'Nhà phân phối Masan (NPP)',
  'Nhà bán lẻ (NBL)',
  'SME businesses'
]

requirements: [
  'All Level 4 requirements',
  'Address verification OR workplace verification OR income source verification',
  'Business license',
  'Business registration certificate'
]

limits: {
  monthlyTransaction: Infinity // Subject to Risk Management approval
}
```

**Important**: Level 5 is an internal extension and should be clearly marked as such in documentation.

---

## 📝 COMPLIANCE CHECKLIST FOR AI

When working on KYC/CIF documentation, always verify:

- [ ] Are transaction limits exactly as specified in TT 40/2024?
  - Level 2: 100M VND/month ✓
  - Level 3: 500M VND/month ✓
  - Level 4: UNLIMITED ✓

- [ ] Is bank account linking marked as MANDATORY for Level 2+?

- [ ] Is biometric authentication timeline correct?
  - Current: Optional
  - From 01/01/2026: MANDATORY

- [ ] Are transaction limit exceptions properly documented?

- [ ] Is Level 5 clearly marked as "Extension - Not in TT 40/2024"?

- [ ] Does documentation cite legal basis (Điều 26, Điều 27)?

- [ ] Is the official link to TT 40/2024 included?

---

## 🚨 CRITICAL ERRORS TO AVOID

### ❌ WRONG: Outdated limits

```typescript
monthlyTransaction: 20_000_000  // ❌ Old regulation
```

### ✅ CORRECT: TT 40/2024 limits

```typescript
monthlyTransaction: 100_000_000 // ✅ TT 40/2024
```

### ❌ WRONG: Bank linking optional

```typescript
requirements: [
  'Bank account linking (optional)' // ❌ WRONG
]
```

### ✅ CORRECT: Bank linking mandatory

```typescript
requirements: [
  'Bank account linking (MANDATORY)' // ✅ CORRECT
]
```

### ❌ WRONG: Daily/balance limits on Level 2

```typescript
limits: {
  maxBalance: 10_000_000,      // ❌ No such limit in TT 40/2024
  dailyTransaction: 10_000_000 // ❌ No such limit in TT 40/2024
}
```

### ✅ CORRECT: Only monthly limit

```typescript
limits: {
  maxBalance: Infinity,              // ✅ No balance limit
  dailyTransaction: Infinity,        // ✅ No daily limit
  monthlyTransaction: 100_000_000    // ✅ Only monthly limit
}
```

---

## 🔄 ONBOARDING FLOW REQUIREMENTS

### Correct CIF Creation Timing

```
User registers → Level 1 (NO CIF) → User wants wallet → 
eKYC + Bank linking → Level 2 (CREATE CIF) ✓
```

**Rationale**: 
- Level 1: Only user account, no CIF (prevents junk CIFs)
- Level 2: CIF created only when user completes eKYC and banks linking

### AML Screening Placement

```
Level 2 (100M) → User needs higher limit → 
Level 3 eKYC Advanced → AML Screening → 
  - Low/Medium Risk → Level 3 (500M) ✓
  - High Risk → Manual Review → Level 4 (UNLIMITED) ✓
```

---

## 📚 REFERENCE DOCUMENTS

### Primary Sources

1. **Thông tư 40/2024/TT-NHNN**
   - Link: https://thuvienphapluat.vn/van-ban/Tien-te-Ngan-hang/Thong-tu-40-2024-TT-NHNN-huong-dan-hoat-dong-cung-ung-dich-vu-trung-gian-thanh-toan-615328.aspx
   - Status: Active (Effective from October 1, 2024)

2. **Luật Phòng, chống rửa tiền 2022** (Law on Anti-Money Laundering 2022)
   - Relevant for AML screening requirements

3. **Nghị định 52/2024/NĐ-CP** (Decree 52/2024/ND-CP)
   - Supporting regulations for payment services

### Secondary Sources

- Ngân hàng Nhà nước Việt Nam official guidance
- Financial sector compliance bulletins

---

## 💡 TIPS FOR AI ASSISTANTS

### When Updating KYC Documentation:

1. **Always cross-reference** with this document
2. **Cite legal basis** (e.g., "TT 40/2024 Điều 26")
3. **Include official links** to regulations
4. **Mark extensions clearly** (Level 5 is Masan extension)
5. **Use correct Vietnamese terms** alongside English
6. **Verify transaction limits** exactly match TT 40/2024
7. **Emphasize mandatory requirements** (bank linking, biometric from 2026)

### When Writing Code Examples:

```typescript
// ✅ GOOD: Include legal context
limits: {
  monthlyTransaction: 100_000_000 // ⚖️ TT 40/2024 Article 26
}

// ✅ GOOD: Mark mandatory requirements
requirements: [
  'Bank account linking (MANDATORY)', // ⚖️ TT 40/2024
]
```

### When Creating Diagrams:

- Annotate with "TT 40/2024" where applicable
- Use ⚖️ emoji to highlight legal limits
- Clearly separate regulated levels (2-4) from extensions (5)

---

## 🔔 CHANGE LOG

### November 17, 2024
- Initial creation
- Added complete TT 40/2024 context
- Documented all KYC levels (1-5)
- Included transaction limit exceptions
- Added compliance checklist
- Created guidelines for AI assistants

---

## ❓ FAQ FOR AI ASSISTANTS

**Q: Can Level 2 customers have unlimited balance?**
A: Yes. TT 40/2024 only limits monthly transactions (100M), not balance.

**Q: Is bank linking optional?**
A: No. Bank linking is MANDATORY for Level 2 and above per TT 40/2024.

**Q: When does biometric authentication become mandatory?**
A: From January 1, 2026 for all new eWallet registrations.

**Q: Can merchant wallets exceed 100M/month?**
A: Yes. Merchant wallets (payment acceptance units) are exempt from limits.

**Q: Is Level 5 part of TT 40/2024?**
A: No. Level 5 is a Masan extension for offline merchants (NPP, NBL).

**Q: What is the daily transaction limit for Level 2?**
A: There is NO daily limit. Only monthly limit of 100M VND.

**Q: Can a customer skip from Level 2 to Level 4?**
A: Yes, if they complete full verification and pass compliance review.

---

## 📧 Contact for Clarification

For questions about regulatory compliance:
- Check official Ngân hàng Nhà nước website
- Reference Thông tư 40/2024/TT-NHNN official document
- Consult with project legal/compliance team

---

**End of Regulations Context Document**

*This document should be updated whenever new regulations are issued or existing ones are amended.*

