# XV. Configuration & Settings - Cấu hình Hệ thống

## Tổng quan

Module Configuration & Settings quản lý tất cả các cấu hình hệ thống, business rules, và parameters có thể điều chỉnh mà không cần deploy code mới.

## System Parameters

```typescript
interface SystemParameter {
  key: string;
  value: any;
  dataType: 'STRING' | 'NUMBER' | 'BOOLEAN' | 'JSON' | 'ARRAY';
  category: string;
  description: string;
  
  // Validation
  validation?: {
    min?: number;
    max?: number;
    regex?: string;
    allowedValues?: any[];
  };
  
  // Metadata
  sensitive: boolean;        // Hide value in UI
  requiresRestart: boolean;  // Need app restart after change
  canOverride: boolean;      // Can be overridden per tenant
  
  // Audit
  lastModifiedBy: string;
  lastModifiedAt: string;
}

// Example parameters
const SYSTEM_PARAMS = {
  // Transaction limits
  'transaction.max_single_amount': 50_000_000,
  'transaction.max_daily_amount': 100_000_000,
  'transaction.max_daily_count': 50,
  
  // Fees
  'fee.transaction.percentage': 0.5,
  'fee.transaction.min': 1_000,
  'fee.transaction.max': 50_000,
  'fee.withdrawal.fixed': 5_000,
  
  // KYC
  'kyc.level1.max_balance': 10_000_000,
  'kyc.level2.max_balance': 100_000_000,
  'kyc.verification.timeout_days': 7,
  
  // Security
  'security.max_login_attempts': 5,
  'security.password_expiry_days': 90,
  'security.session_timeout_minutes': 30,
  'security.2fa_required_for_large_txn': true,
  'security.2fa_threshold': 10_000_000,
  
  // System
  'system.maintenance_mode': false,
  'system.api_rate_limit': 100,
  'system.support_email': 'support@masan.com',
  'system.support_phone': '1900xxxx'
};
```

## Fee Configuration

```typescript
interface FeeConfig {
  feeId: string;
  name: string;
  transactionType: string;
  
  // Fee structure
  structure: {
    type: 'FIXED' | 'PERCENTAGE' | 'TIERED' | 'COMBINED';
    
    fixed?: {
      amount: number;
      currency: string;
    };
    
    percentage?: {
      rate: number;          // 0.5 = 0.5%
      min?: number;
      max?: number;
    };
    
    tiered?: Array<{
      from: number;
      to: number;
      fee: number;
    }>;
  };
  
  // Fee bearer
  paidBy: 'SENDER' | 'RECEIVER' | 'SHARED';
  
  // Exceptions
  exemptions?: Array<{
    userType: string;
    reason: string;
  }>;
  
  // Effective period
  effectiveFrom: string;
  effectiveTo?: string;
  
  active: boolean;
}
```

## Currency Management

```typescript
interface Currency {
  code: string;              // ISO 4217 (VND, USD, EUR)
  name: string;
  symbol: string;
  
  // Display
  decimalPlaces: number;
  thousandSeparator: string;
  decimalSeparator: string;
  
  // Exchange rates
  exchangeRates: Array<{
    toCurrency: string;
    rate: number;
    effectiveDate: string;
  }>;
  
  // Status
  active: boolean;
  isBaseCurrency: boolean;
}

async function convertCurrency(
  amount: number,
  fromCurrency: string,
  toCurrency: string
): Promise<number> {
  if (fromCurrency === toCurrency) {
    return amount;
  }
  
  const rate = await getExchangeRate(fromCurrency, toCurrency);
  return amount * rate;
}
```

## Business Rules

```typescript
interface BusinessRule {
  ruleId: string;
  name: string;
  description: string;
  
  // Rule definition
  conditions: {
    field: string;
    operator: 'EQ' | 'NEQ' | 'GT' | 'GTE' | 'LT' | 'LTE' | 'IN' | 'NOT_IN';
    value: any;
  }[];
  
  // Action
  action: {
    type: 'ALLOW' | 'DENY' | 'REQUIRE_APPROVAL' | 'SET_VALUE' | 'TRIGGER_WEBHOOK';
    parameters?: Record<string, any>;
  };
  
  // Priority (higher = evaluated first)
  priority: number;
  
  // Effective period
  effectiveFrom: string;
  effectiveTo?: string;
  
  active: boolean;
}

// Example: Require approval for large transactions
const LARGE_TXN_RULE: BusinessRule = {
  ruleId: 'RULE_001',
  name: 'Large Transaction Approval',
  description: 'Transactions over 50M require approval',
  conditions: [
    {
      field: 'transaction.amount',
      operator: 'GT',
      value: 50_000_000
    }
  ],
  action: {
    type: 'REQUIRE_APPROVAL',
    parameters: {
      approverRole: 'TRANSACTION_APPROVER',
      timeoutHours: 24
    }
  },
  priority: 100,
  effectiveFrom: '2024-01-01',
  active: true
};
```

## Feature Flags

```typescript
interface FeatureFlag {
  flagId: string;
  name: string;
  description: string;
  
  // Flag value
  enabled: boolean;
  
  // Rollout strategy
  rollout?: {
    type: 'PERCENTAGE' | 'USER_LIST' | 'USER_ATTRIBUTE';
    percentage?: number;
    userIds?: string[];
    attribute?: {
      key: string;
      values: any[];
    };
  };
  
  // Environments
  environments: {
    dev: boolean;
    staging: boolean;
    production: boolean;
  };
}

async function isFeatureEnabled(
  flagId: string,
  userId?: string
): Promise<boolean> {
  const flag = await getFeatureFlag(flagId);
  
  if (!flag.enabled) {
    return false;
  }
  
  if (!flag.rollout) {
    return true;
  }
  
  // Check rollout strategy
  switch (flag.rollout.type) {
    case 'PERCENTAGE':
      return hashUserId(userId) < flag.rollout.percentage;
    
    case 'USER_LIST':
      return flag.rollout.userIds.includes(userId);
    
    case 'USER_ATTRIBUTE':
      const user = await getUser(userId);
      return flag.rollout.attribute.values.includes(
        user[flag.rollout.attribute.key]
      );
  }
}
```

## Localization

```typescript
interface Localization {
  locale: string;            // vi-VN, en-US
  translations: Record<string, string>;
  
  // Date/Time format
  dateFormat: string;
  timeFormat: string;
  timezone: string;
  
  // Number format
  numberFormat: {
    decimalSeparator: string;
    thousandSeparator: string;
    currencyPosition: 'PREFIX' | 'SUFFIX';
  };
}

// Example translations
const TRANSLATIONS = {
  'vi-VN': {
    'transaction.success': 'Giao dịch thành công',
    'transaction.failed': 'Giao dịch thất bại',
    'error.insufficient_balance': 'Số dư không đủ'
  },
  'en-US': {
    'transaction.success': 'Transaction successful',
    'transaction.failed': 'Transaction failed',
    'error.insufficient_balance': 'Insufficient balance'
  }
};
```

## Workflow Configuration

```typescript
interface Workflow {
  workflowId: string;
  name: string;
  entityType: string;        // 'TRANSACTION', 'LOAN_APPLICATION', etc.
  
  steps: Array<{
    stepId: string;
    name: string;
    type: 'APPROVAL' | 'NOTIFICATION' | 'API_CALL' | 'SCRIPT';
    
    // Approval step
    approvers?: {
      type: 'ROLE' | 'USER' | 'DYNAMIC';
      roleId?: string;
      userId?: string;
      expression?: string;   // For dynamic approver selection
    };
    
    // Conditions
    conditions?: Array<{
      field: string;
      operator: string;
      value: any;
    }>;
    
    // Timeout
    timeoutHours?: number;
    onTimeout?: 'AUTO_APPROVE' | 'AUTO_REJECT' | 'ESCALATE';
  }>;
  
  active: boolean;
}
```

## API Reference

```typescript
// System parameters
GET /api/v1/config/parameters
GET /api/v1/config/parameters/{key}
PUT /api/v1/config/parameters/{key}

// Fees
GET /api/v1/config/fees
POST /api/v1/config/fees
PUT /api/v1/config/fees/{feeId}

// Currency
GET /api/v1/config/currencies
POST /api/v1/config/currencies
PUT /api/v1/config/currencies/{code}
GET /api/v1/config/exchange-rates

// Business rules
GET /api/v1/config/rules
POST /api/v1/config/rules
PUT /api/v1/config/rules/{ruleId}

// Feature flags
GET /api/v1/config/features
PUT /api/v1/config/features/{flagId}
```

## Best Practices

1. **Change Management**
   - Always test in staging first
   - Document all changes
   - Require approval for critical params
   - Rollback plan ready

2. **Versioning**
   - Track configuration versions
   - Audit trail for all changes
   - Ability to rollback

3. **Security**
   - Encrypt sensitive parameters
   - Access control for config changes
   - Regular security reviews

## Kết luận

Configuration & Settings cho phép linh hoạt vận hành:
- ✅ Dynamic configuration
- ✅ Business rule engine
- ✅ Feature flags for safe rollouts
- ✅ Multi-currency support
- ✅ Localization ready

