# XI. Risk Management - Quản lý Rủi ro

## Tổng quan

Risk Management Module bảo vệ hệ thống khỏi gian lận, rủi ro tín dụng, và các hoạt động đáng ngờ thông qua monitoring real-time, rule engine, và machine learning.

## Fraud Detection

### Rule-based Detection

```typescript
interface FraudRule {
  ruleId: string;
  name: string;
  description: string;
  
  conditions: {
    // Transaction patterns
    amountThreshold?: number;
    velocityCheck?: {
      maxTransactions: number;
      timeWindow: number;        // minutes
    };
    
    // Behavioral
    unusualTime?: boolean;       // Transaction at unusual hours
    unusualLocation?: boolean;   // Transaction from new location
    deviceChange?: boolean;      // New device
    
    // Account
    newAccount?: {
      ageThreshold: number;      // days
    };
  };
  
  action: 'BLOCK' | 'REVIEW' | 'ALERT';
  severity: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  
  active: boolean;
}

// Example rules
const FRAUD_RULES: FraudRule[] = [
  {
    ruleId: 'FR001',
    name: 'Large transaction from new account',
    conditions: {
      amountThreshold: 10_000_000,
      newAccount: { ageThreshold: 7 }
    },
    action: 'REVIEW',
    severity: 'HIGH'
  },
  {
    ruleId: 'FR002',
    name: 'Velocity - too many transactions',
    conditions: {
      velocityCheck: {
        maxTransactions: 10,
        timeWindow: 60
      }
    },
    action: 'BLOCK',
    severity: 'CRITICAL'
  }
];
```

### Real-time Fraud Scoring

```typescript
async function calculateFraudScore(
  transaction: Transaction
): Promise<FraudAssessment> {
  let score = 0;
  const flags = [];
  
  // 1. Amount analysis
  const avgTransaction = await getAvgTransactionAmount(transaction.userId);
  if (transaction.amount > avgTransaction * 3) {
    score += 20;
    flags.push('UNUSUAL_AMOUNT');
  }
  
  // 2. Velocity check
  const recentTxnCount = await getTransactionCount(
    transaction.userId,
    'last_hour'
  );
  if (recentTxnCount > 10) {
    score += 30;
    flags.push('HIGH_VELOCITY');
  }
  
  // 3. Device fingerprint
  const knownDevice = await isKnownDevice(
    transaction.userId,
    transaction.deviceId
  );
  if (!knownDevice) {
    score += 15;
    flags.push('NEW_DEVICE');
  }
  
  // 4. Location check
  const knownLocation = await isKnownLocation(
    transaction.userId,
    transaction.ipAddress
  );
  if (!knownLocation) {
    score += 10;
    flags.push('NEW_LOCATION');
  }
  
  // 5. Time pattern
  const isUnusualTime = await checkTimePattern(
    transaction.userId,
    transaction.timestamp
  );
  if (isUnusualTime) {
    score += 10;
    flags.push('UNUSUAL_TIME');
  }
  
  // 6. Account age
  const accountAge = await getAccountAge(transaction.userId);
  if (accountAge < 7) {
    score += 15;
    flags.push('NEW_ACCOUNT');
  }
  
  return {
    score,
    level: score >= 70 ? 'HIGH' : score >= 40 ? 'MEDIUM' : 'LOW',
    flags,
    action: score >= 70 ? 'BLOCK' : score >= 40 ? 'REVIEW' : 'ALLOW'
  };
}
```

### Machine Learning Models

```typescript
interface MLFraudModel {
  modelId: string;
  modelType: 'RANDOM_FOREST' | 'NEURAL_NETWORK' | 'GRADIENT_BOOSTING';
  
  features: string[];        // Input features
  version: string;
  accuracy: number;
  precision: number;
  recall: number;
  
  deployedAt: string;
  lastRetrainedAt: string;
}

async function predictFraud(
  transaction: Transaction
): Promise<MLPrediction> {
  // Extract features
  const features = await extractFeatures(transaction);
  
  // Get model
  const model = await getActiveModel('fraud_detection');
  
  // Make prediction
  const prediction = await model.predict(features);
  
  return {
    isFraud: prediction.probability > 0.7,
    probability: prediction.probability,
    confidence: prediction.confidence,
    features: features,
    modelVersion: model.version
  };
}
```

## Transaction Monitoring

### Suspicious Activity Detection

```typescript
interface SuspiciousActivity {
  activityId: string;
  userId: string;
  
  type: 'STRUCTURING' | 'RAPID_MOVEMENT' | 'UNUSUAL_PATTERN' | 'HIGH_RISK_COUNTRY';
  
  details: {
    transactions: Transaction[];
    pattern: string;
    timeframe: string;
  };
  
  riskScore: number;
  
  status: 'DETECTED' | 'UNDER_REVIEW' | 'CLEARED' | 'REPORTED';
  detectedAt: string;
  reviewedBy?: string;
  reviewNotes?: string;
}

// Detect structuring (breaking large transaction into smaller ones)
async function detectStructuring(userId: string): Promise<boolean> {
  const last24h = await getTransactions(userId, 'last_24h');
  
  // Check for multiple transactions just below reporting threshold
  const threshold = 15_000_000;  // 15 triệu
  const nearThreshold = last24h.filter(
    t => t.amount > threshold * 0.8 && t.amount < threshold
  );
  
  if (nearThreshold.length >= 3) {
    await createSuspiciousActivity({
      userId,
      type: 'STRUCTURING',
      details: {
        transactions: nearThreshold,
        pattern: 'Multiple transactions just below threshold',
        timeframe: '24 hours'
      }
    });
    return true;
  }
  
  return false;
}
```

## Credit Risk Management

### Credit Risk Assessment

```typescript
interface CreditRiskAssessment {
  userId: string;
  
  // Score
  creditScore: number;
  riskRating: 'AAA' | 'AA' | 'A' | 'BBB' | 'BB' | 'B' | 'CCC';
  pd: number;                // Probability of Default
  lgd: number;               // Loss Given Default
  ead: number;               // Exposure at Default
  expectedLoss: number;      // PD * LGD * EAD
  
  // Factors
  factors: {
    paymentHistory: number;
    creditUtilization: number;
    debtToIncome: number;
    accountAge: number;
    recentInquiries: number;
  };
  
  // Decision
  recommendation: {
    approved: boolean;
    maxCreditLimit: number;
    interestRate: number;
    terms: string[];
  };
}
```

### Portfolio Risk

```typescript
interface PortfolioRisk {
  totalExposure: number;
  
  // By risk rating
  byRating: Record<string, {
    count: number;
    exposure: number;
    percentage: number;
  }>;
  
  // By product
  byProduct: Record<string, {
    exposure: number;
    npl: number;              // Non-Performing Loans
    nplRatio: number;
  }>;
  
  // Concentrations
  concentrations: {
    topBorrowers: Array<{
      userId: string;
      exposure: number;
      percentage: number;
    }>;
    
    byIndustry: Record<string, number>;
    byRegion: Record<string, number>;
  };
  
  // Metrics
  metrics: {
    avgCreditScore: number;
    avgPD: number;
    totalExpectedLoss: number;
    capitalRequired: number;
  };
}
```

## Limits & Controls

### Transaction Limits

```typescript
interface TransactionLimits {
  userId: string;
  
  limits: {
    // Per transaction
    single: {
      min: number;
      max: number;
    };
    
    // Daily limits
    daily: {
      amount: number;
      count: number;
      remaining: {
        amount: number;
        count: number;
      };
    };
    
    // Monthly limits
    monthly: {
      amount: number;
      count: number;
      remaining: {
        amount: number;
        count: number;
      };
    };
  };
  
  // By transaction type
  byType: Record<TransactionType, {
    daily: number;
    monthly: number;
  }>;
}

async function checkTransactionLimit(
  userId: string,
  amount: number,
  type: TransactionType
): Promise<LimitCheckResult> {
  const limits = await getTransactionLimits(userId);
  const usage = await getTodayUsage(userId);
  
  // Check single transaction limit
  if (amount > limits.single.max) {
    return {
      allowed: false,
      reason: 'EXCEEDS_SINGLE_LIMIT',
      limit: limits.single.max
    };
  }
  
  // Check daily limit
  if (usage.today.amount + amount > limits.daily.amount) {
    return {
      allowed: false,
      reason: 'EXCEEDS_DAILY_LIMIT',
      limit: limits.daily.amount,
      used: usage.today.amount
    };
  }
  
  // Check daily count
  if (usage.today.count >= limits.daily.count) {
    return {
      allowed: false,
      reason: 'EXCEEDS_DAILY_COUNT',
      limit: limits.daily.count
    };
  }
  
  return { allowed: true };
}
```

## Blacklist Management

```typescript
interface Blacklist {
  type: 'USER' | 'DEVICE' | 'IP' | 'CARD' | 'MERCHANT';
  
  entries: Array<{
    entryId: string;
    value: string;
    reason: string;
    addedAt: string;
    addedBy: string;
    expiresAt?: string;
    active: boolean;
  }>;
}

async function checkBlacklist(
  type: BlacklistType,
  value: string
): Promise<boolean> {
  const entry = await db('blacklist')
    .where({ type, value, active: true })
    .where('expiresAt', '>', new Date())
    .orWhereNull('expiresAt')
    .first();
  
  return !!entry;
}
```

## Alerts & Notifications

### Risk Alert

```typescript
interface RiskAlert {
  alertId: string;
  severity: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  type: string;
  
  subject: {
    type: 'USER' | 'TRANSACTION' | 'ACCOUNT';
    id: string;
  };
  
  message: string;
  details: Record<string, any>;
  
  status: 'OPEN' | 'ACKNOWLEDGED' | 'INVESTIGATING' | 'RESOLVED' | 'FALSE_POSITIVE';
  
  assignedTo?: string;
  createdAt: string;
  resolvedAt?: string;
}

// Auto-escalate high severity alerts
async function escalateAlert(alert: RiskAlert): Promise<void> {
  if (alert.severity === 'CRITICAL') {
    // Immediate notification to risk team
    await sendUrgentNotification({
      recipients: ['risk-team@company.com'],
      subject: `CRITICAL ALERT: ${alert.type}`,
      body: alert.message
    });
    
    // SMS to on-call person
    await sendSMS({
      to: await getOnCallPerson(),
      message: `Critical risk alert: ${alert.alertId}`
    });
  }
}
```

## API Reference

```typescript
// Fraud detection
POST /api/v1/risk/fraud/check
GET /api/v1/risk/fraud/score/{transactionId}

// Transaction monitoring
GET /api/v1/risk/monitoring/suspicious-activities
POST /api/v1/risk/monitoring/report

// Credit risk
POST /api/v1/risk/credit/assess
GET /api/v1/risk/credit/portfolio

// Limits
GET /api/v1/risk/limits/{userId}
POST /api/v1/risk/limits/{userId}/update

// Blacklist
POST /api/v1/risk/blacklist/add
DELETE /api/v1/risk/blacklist/remove
GET /api/v1/risk/blacklist/check

// Alerts
GET /api/v1/risk/alerts
POST /api/v1/risk/alerts/{alertId}/acknowledge
POST /api/v1/risk/alerts/{alertId}/resolve
```

## Use Cases trong hệ thống Masan

### 1. Phát hiện gian lận tại NBL

```typescript
// Detect if customer card is compromised
if (fraudScore > 70) {
  await blockCard(cardId);
  await notifyCustomer({
    type: 'FRAUD_ALERT',
    message: 'Suspicious activity detected. Card temporarily blocked.'
  });
}
```

### 2. Credit risk cho NPP

```typescript
const creditAssessment = await assessCreditRisk('NPP_123');
if (creditAssessment.riskRating >= 'BBB') {
  // Approve credit limit
  await approveCreditLimit({
    userId: 'NPP_123',
    amount: 100_000_000,
    terms: 30  // days
  });
}
```

## Best Practices

1. **Detection**
   - Multi-layered approach (rules + ML)
   - Real-time monitoring
   - Regular model retraining
   - False positive minimization

2. **Response**
   - Automated blocking for critical cases
   - Human review for medium risk
   - Fast escalation process
   - Customer communication

3. **Compliance**
   - SAR (Suspicious Activity Report) filing
   - Record retention
   - Regular audits
   - Staff training

## Kết luận

Risk Management bảo vệ hệ thống Masan:

- ✅ Real-time fraud detection
- ✅ Comprehensive transaction monitoring
- ✅ Credit risk management
- ✅ Proactive controls
- ✅ Fast incident response

