# IX. Ledger System - Hệ thống Sổ cái

## Tổng quan

Ledger System là trái tim kế toán của Core Banking, thực hiện ghi chép mọi giao dịch tài chính theo nguyên tắc Double-entry Bookkeeping. Đảm bảo tính chính xác tuyệt đối và khả năng audit đầy đủ.

## Sơ đồ Tổng quan

### 1. Kiến trúc tổng thể Ledger System

```mermaid
graph TB
    subgraph "Transaction Sources"
        TS1[Payment Service]
        TS2[Transfer Service]
        TS3[Merchant Service]
        TS4[Credit Service]
        TS5[Card Service]
        TS6[Fee Service]
    end
    
    subgraph "Ledger Core"
        JE[Journal Entry<br/>Creation]
        VAL[Validation<br/>Engine]
        POST[Posting<br/>Engine]
        
        JE --> VAL
        VAL --> POST
    end
    
    subgraph "Chart of Accounts"
        COA1[Assets<br/>Tài sản]
        COA2[Liabilities<br/>Nợ phải trả]
        COA3[Equity<br/>Vốn CSH]
        COA4[Revenue<br/>Doanh thu]
        COA5[Expense<br/>Chi phí]
    end
    
    subgraph "Account Management"
        GL[General Ledger<br/>Sổ cái tổng hợp]
        SUB[Sub-Ledgers<br/>Sổ cái chi tiết]
        BAL[Balance<br/>Management]
    end
    
    subgraph "Processing"
        DBE[Double-Entry<br/>Bookkeeping]
        ACC[Account<br/>Update]
        HIS[Transaction<br/>History]
    end
    
    subgraph "Period Management"
        DAILY[Daily Close]
        MONTHLY[Month-End Close]
        YEARLY[Year-End Close]
        REV[Revenue Recognition]
    end
    
    subgraph "Reconciliation"
        AUTO[Auto-Reconciliation]
        MANUAL[Manual Reconciliation]
        MATCH[Matching Engine]
        DISC[Discrepancy<br/>Resolution]
    end
    
    subgraph "Reporting"
        TB[Trial Balance<br/>Bảng cân đối số phát sinh]
        BS[Balance Sheet<br/>Bảng cân đối kế toán]
        IS[Income Statement<br/>Báo cáo kết quả KD]
        CF[Cash Flow<br/>Lưu chuyển tiền tệ]
        STMT[Account Statement<br/>Sao kê tài khoản]
        AUDIT[Audit Trail<br/>Dấu vết kiểm toán]
    end
    
    subgraph "Data Storage"
        DB1[(Ledger Entries<br/>Database)]
        DB2[(Journal Entries<br/>Database)]
        DB3[(GL Accounts<br/>Database)]
        DB4[(Period Balances<br/>Archive)]
    end
    
    TS1 --> JE
    TS2 --> JE
    TS3 --> JE
    TS4 --> JE
    TS5 --> JE
    TS6 --> JE
    
    POST --> DBE
    DBE --> ACC
    ACC --> HIS
    
    POST --> COA1
    POST --> COA2
    POST --> COA3
    POST --> COA4
    POST --> COA5
    
    COA1 --> GL
    COA2 --> GL
    COA3 --> GL
    COA4 --> GL
    COA5 --> GL
    
    GL --> SUB
    SUB --> BAL
    
    BAL --> DAILY
    DAILY --> MONTHLY
    MONTHLY --> YEARLY
    MONTHLY --> REV
    
    GL --> AUTO
    AUTO --> MATCH
    MANUAL --> MATCH
    MATCH --> DISC
    
    GL --> TB
    GL --> BS
    GL --> IS
    GL --> CF
    SUB --> STMT
    HIS --> AUDIT
    
    POST --> DB1
    JE --> DB2
    GL --> DB3
    MONTHLY --> DB4
    
    style JE fill:#e1f5ff
    style POST fill:#e1f5ff
    style GL fill:#fff4e6
    style TB fill:#f3e5f5
    style BS fill:#f3e5f5
    style AUTO fill:#e8f5e9
```

### 2. Luồng xử lý Double-Entry Bookkeeping

```mermaid
flowchart TD
    A[Transaction Initiated] --> B[Create Journal Entry]
    
    B --> C{Identify Accounts}
    C --> D[Debit Account]
    C --> E[Credit Account]
    
    D --> F[Calculate Amount]
    E --> F
    
    F --> G{Validate Entry}
    
    G -->|Check 1| H{Debit = Credit?}
    H -->|No| I[Reject: Unbalanced]
    H -->|Yes| J{Accounts Exist?}
    
    J -->|No| K[Reject: Invalid Account]
    J -->|Yes| L{Sufficient Balance?}
    
    L -->|No| M[Reject: Insufficient Funds]
    L -->|Yes| N{Period Open?}
    
    N -->|No| O[Reject: Period Closed]
    N -->|Yes| P[Begin Database Transaction]
    
    P --> Q[Insert Journal Entry]
    Q --> R[For Each Entry]
    
    R --> S[Lock Account]
    S --> T[Get Current Balance]
    T --> U[Calculate New Balance]
    U --> V[Insert Ledger Entry]
    V --> W[Update Account Balance]
    W --> X[Release Lock]
    
    X --> Y{More Entries?}
    Y -->|Yes| R
    Y -->|No| Z[Commit Transaction]
    
    Z --> AA[Update GL Balances]
    AA --> AB[Create Audit Log]
    AB --> AC[Notify Reporting]
    AC --> AD[Return Success]
    
    I --> AE[Return Error]
    K --> AE
    M --> AE
    O --> AE
    
    style P fill:#e1f5ff
    style Z fill:#c8e6c9
    style AE fill:#ffcdd2
```

### 3. Cấu trúc Chart of Accounts (COA)

```mermaid
graph LR
    subgraph "1000 - Assets (Tài sản)"
        A1[1000 Assets Root]
        A1 --> A11[1100 Current Assets<br/>Tài sản ngắn hạn]
        A1 --> A12[1200 Fixed Assets<br/>Tài sản cố định]
        
        A11 --> A111[1110 Cash & Equivalents<br/>Tiền và tương đương]
        A11 --> A112[1120 Receivables<br/>Các khoản phải thu]
        A11 --> A113[1130 Inventory<br/>Hàng tồn kho]
        
        A111 --> A1111[1111 Customer Wallets]
        A111 --> A1112[1112 Merchant Accounts]
        A111 --> A1113[1113 Bank Accounts]
        A111 --> A1114[1114 Clearing Accounts]
    end
    
    subgraph "2000 - Liabilities (Nợ phải trả)"
        L1[2000 Liabilities Root]
        L1 --> L11[2100 Current Liabilities<br/>Nợ ngắn hạn]
        L1 --> L12[2200 Long-term Liabilities<br/>Nợ dài hạn]
        
        L11 --> L111[2110 Customer Deposits<br/>Tiền gửi khách hàng]
        L11 --> L112[2120 Payables<br/>Phải trả]
        L11 --> L113[2130 Accrued Expenses<br/>Chi phí trích trước]
        
        L111 --> L1111[2111 Wallet Balances]
        L111 --> L1112[2112 Credit Balances]
    end
    
    subgraph "3000 - Equity (Vốn chủ sở hữu)"
        E1[3000 Equity Root]
        E1 --> E11[3100 Share Capital<br/>Vốn góp]
        E1 --> E12[3200 Retained Earnings<br/>Lợi nhuận giữ lại]
        E1 --> E13[3300 Current Period Profit<br/>Lãi năm nay]
    end
    
    subgraph "4000 - Revenue (Doanh thu)"
        R1[4000 Revenue Root]
        R1 --> R11[4100 Operating Revenue<br/>Doanh thu hoạt động]
        R1 --> R12[4200 Other Revenue<br/>Thu nhập khác]
        
        R11 --> R111[4110 Transaction Fees]
        R11 --> R112[4120 Service Fees]
        R11 --> R113[4130 Interest Income]
        R11 --> R114[4140 FX Income]
    end
    
    subgraph "5000 - Expenses (Chi phí)"
        X1[5000 Expense Root]
        X1 --> X11[5100 Operating Expenses<br/>Chi phí hoạt động]
        X1 --> X12[5200 Financial Expenses<br/>Chi phí tài chính]
        
        X11 --> X111[5110 Processing Costs]
        X11 --> X112[5120 Personnel Costs]
        X11 --> X113[5130 IT Infrastructure]
        X11 --> X114[5140 Marketing]
    end
    
    style A1 fill:#c8e6c9
    style L1 fill:#ffccbc
    style E1 fill:#bbdefb
    style R1 fill:#fff9c4
    style X1 fill:#f8bbd0
```

### 4. Quy trình Reconciliation (Đối soát)

```mermaid
sequenceDiagram
    participant SYS as Internal System<br/>(Core Banking)
    participant REC as Reconciliation<br/>Engine
    participant EXT as External System<br/>(Bank/Partner)
    participant DISC as Discrepancy<br/>Handler
    participant RPT as Reporting
    
    Note over SYS,RPT: Daily Reconciliation Process
    
    SYS->>REC: Export internal ledger entries<br/>(T-Day transactions)
    EXT->>REC: Import external statements<br/>(Bank/partner data)
    
    REC->>REC: Load reconciliation rules
    REC->>REC: Parse and normalize data
    
    loop For each internal entry
        REC->>REC: Search matching external entry
        REC->>REC: Apply matching rules:<br/>- Amount match<br/>- Reference match<br/>- Date match (+/- tolerance)
        
        alt Perfect Match
            REC->>REC: Mark as MATCHED
            REC->>REC: Update status in both systems
        else Partial Match
            REC->>REC: Mark as SUSPECTED_MATCH
            REC->>DISC: Queue for manual review
        else No Match
            REC->>REC: Mark as UNMATCHED
            REC->>DISC: Create discrepancy record
        end
    end
    
    REC->>REC: Identify unmatched external entries
    REC->>DISC: Create external discrepancy records
    
    DISC->>DISC: Analyze discrepancies
    
    alt Auto-resolvable
        DISC->>DISC: Apply auto-correction rules
        DISC->>SYS: Create adjustment entries
        SYS->>SYS: Post adjustments
        DISC->>REC: Re-run matching
    else Manual review needed
        DISC->>RPT: Generate discrepancy report
        RPT->>RPT: Notify finance team
    end
    
    REC->>REC: Calculate reconciliation metrics:<br/>- Match rate<br/>- Unmatched amount<br/>- Discrepancy count
    
    REC->>RPT: Generate reconciliation report
    RPT->>RPT: Store report
    RPT->>RPT: Send notifications
    
    alt Match rate >= 99%
        RPT->>RPT: Mark as RECONCILED
    else Match rate < 99%
        RPT->>RPT: Mark as PENDING_REVIEW
        RPT->>RPT: Escalate to management
    end
```

### 5. Vòng đời Period Close (Đóng sổ kỳ kế toán)

```mermaid
stateDiagram-v2
    [*] --> Open: Period starts
    
    Open --> PreClose: Month-end approaching
    
    PreClose --> Validation: Initiate close
    
    Validation --> ValidationChecks
    state ValidationChecks {
        [*] --> CheckUnposted
        CheckUnposted --> CheckBalance: All posted
        CheckBalance --> CheckReconcile: Balanced
        CheckReconcile --> [*]: Reconciled
        
        CheckUnposted --> ValidationFailed: Has unposted
        CheckBalance --> ValidationFailed: Not balanced
        CheckReconcile --> ValidationFailed: Not reconciled
    }
    
    ValidationChecks --> Processing: All checks passed
    ValidationFailed --> PreClose: Fix issues
    
    Processing --> ProcessingSteps
    state ProcessingSteps {
        [*] --> GenTrialBalance
        GenTrialBalance --> CalcPeriodBalance: Trial balance OK
        CalcPeriodBalance --> AccrueRevenue: Balances calculated
        AccrueRevenue --> GenFinStatements: Revenue recognized
        GenFinStatements --> ArchiveData: Statements generated
        ArchiveData --> [*]: Data archived
    }
    
    ProcessingSteps --> Closed: Close successful
    ProcessingSteps --> ProcessingError: Error occurred
    ProcessingError --> PreClose: Rollback & fix
    
    Closed --> NextPeriod: Open next period
    NextPeriod --> Open
    
    Closed --> Reopened: Reopen requested
    Reopened --> Open: With restrictions
    
    note right of Closed
        Period locked
        No new transactions
        Only reversals allowed
        with special permission
    end note
```

### 6. Balance Calculation & Update Flow

```mermaid
graph TB
    A[Transaction Posted] --> B{Get Account Type}
    
    B -->|Asset/Expense| C[Debit Normal Balance]
    B -->|Liability/Equity/Revenue| D[Credit Normal Balance]
    
    C --> C1{Entry Type}
    D --> D1{Entry Type}
    
    C1 -->|Debit| C2[Balance = Balance + Amount<br/>Increase]
    C1 -->|Credit| C3[Balance = Balance - Amount<br/>Decrease]
    
    D1 -->|Credit| D2[Balance = Balance + Amount<br/>Increase]
    D1 -->|Debit| D3[Balance = Balance - Amount<br/>Decrease]
    
    C2 --> E[Update Account Balance]
    C3 --> E
    D2 --> E
    D3 --> E
    
    E --> F[Lock Account Record]
    F --> G[Read Current Balance]
    G --> H[Apply Change]
    H --> I[Write New Balance]
    I --> J[Update Last Transaction Time]
    J --> K[Release Lock]
    
    K --> L[Update Parent GL Balance]
    L --> M[Propagate to Chart of Accounts]
    
    M --> N{Update Period Balance}
    N --> O[Current Day Balance]
    N --> P[Month-to-Date Balance]
    N --> Q[Year-to-Date Balance]
    
    O --> R[Update Cache]
    P --> R
    Q --> R
    
    R --> S[Trigger Balance Alerts]
    S --> T{Check Thresholds}
    
    T -->|Below minimum| U[Alert: Low Balance]
    T -->|Above maximum| V[Alert: High Balance]
    T -->|Within limits| W[No Alert]
    
    U --> X[End]
    V --> X
    W --> X
    
    style E fill:#e1f5ff
    style F fill:#fff4e6
    style R fill:#c8e6c9
```

## Double-Entry Bookkeeping

### Nguyên tắc cơ bản

Mỗi giao dịch tạo ra ít nhất 2 bút toán (entries) với tổng Debit = tổng Credit:

```
Debit (Nợ) = Credit (Có)
```

### Ví dụ Chuyển tiền

```typescript
// User A chuyển 1,000,000 VND cho User B
const entries = [
  {
    account: 'WALLET_A',
    type: 'DEBIT',
    amount: 1_000_000,
    description: 'Chuyen tien cho B'
  },
  {
    account: 'WALLET_B',
    type: 'CREDIT',
    amount: 1_000_000,
    description: 'Nhan tien tu A'
  }
];
```

## Cấu trúc Ledger

### Ledger Entry

```typescript
interface LedgerEntry {
  entryId: string;
  transactionId: string;
  
  // Account info
  accountId: string;
  accountNumber: string;
  accountType: string;
  
  // Entry type
  type: 'DEBIT' | 'CREDIT';
  amount: number;
  currency: string;
  
  // Balance after this entry
  balanceAfter: number;
  
  // Description
  description: string;
  referenceNumber?: string;
  
  // Timestamps
  valueDate: string;        // Effective date
  bookingDate: string;      // Actual recording date
  
  // Metadata
  metadata: Record<string, any>;
  
  // Reversal
  reversed: boolean;
  reversalEntryId?: string;
}
```

### Chart of Accounts (COA)

```typescript
interface ChartOfAccounts {
  accountCode: string;      // e.g., '1000', '2000'
  accountName: string;
  accountType: 'ASSET' | 'LIABILITY' | 'EQUITY' | 'REVENUE' | 'EXPENSE';
  
  // Hierarchy
  parentAccountCode?: string;
  level: number;
  
  // Normal balance
  normalBalance: 'DEBIT' | 'CREDIT';
  
  // Status
  active: boolean;
  
  // Metadata
  description: string;
  currency?: string;
}

// Example COA structure
const COA = {
  // Assets (Tài sản)
  '1000': { name: 'Cash and Cash Equivalents', type: 'ASSET' },
  '1001': { name: 'Customer Wallets', type: 'ASSET', parent: '1000' },
  '1002': { name: 'Merchant Accounts', type: 'ASSET', parent: '1000' },
  
  // Liabilities (Nợ phải trả)
  '2000': { name: 'Customer Deposits', type: 'LIABILITY' },
  '2001': { name: 'Pending Settlements', type: 'LIABILITY' },
  
  // Equity (Vốn chủ sở hữu)
  '3000': { name: 'Share Capital', type: 'EQUITY' },
  
  // Revenue (Doanh thu)
  '4000': { name: 'Transaction Fees', type: 'REVENUE' },
  '4001': { name: 'Service Fees', type: 'REVENUE' },
  
  // Expenses (Chi phí)
  '5000': { name: 'Operating Expenses', type: 'EXPENSE' },
  '5001': { name: 'Payment Processing Costs', type: 'EXPENSE' }
};
```

## Transaction Posting

### Posting Process

```mermaid
sequenceDiagram
    participant T as Transaction
    participant L as Ledger
    participant V as Validator
    participant DB as Database
    
    T->>L: Create journal entry
    L->>V: Validate entries
    V->>V: Check debit = credit
    V->>V: Check account exists
    V->>V: Check balance
    
    alt Valid
        V-->>L: Validation passed
        L->>DB: Begin transaction
        L->>DB: Insert entries
        L->>DB: Update balances
        L->>DB: Commit
        DB-->>L: Success
    else Invalid
        V-->>L: Validation failed
        L-->>T: Reject transaction
    end
```

### Journal Entry

```typescript
interface JournalEntry {
  journalId: string;
  transactionId: string;
  
  // Entries (must balance)
  entries: LedgerEntry[];
  
  // Validation
  totalDebit: number;
  totalCredit: number;
  balanced: boolean;
  
  // Status
  status: 'DRAFT' | 'POSTED' | 'REVERSED';
  postedAt?: string;
  postedBy?: string;
  
  // Reversal
  reversalJournalId?: string;
  reversedAt?: string;
}

async function postJournalEntry(
  journal: JournalEntry
): Promise<void> {
  // 1. Validate entries balance
  const totalDebit = journal.entries
    .filter(e => e.type === 'DEBIT')
    .reduce((sum, e) => sum + e.amount, 0);
  
  const totalCredit = journal.entries
    .filter(e => e.type === 'CREDIT')
    .reduce((sum, e) => sum + e.amount, 0);
  
  if (totalDebit !== totalCredit) {
    throw new Error('Journal entries do not balance');
  }
  
  // 2. Start database transaction
  await db.transaction(async (trx) => {
    // 3. Insert journal entry
    await trx('journal_entries').insert({
      ...journal,
      status: 'POSTED',
      postedAt: new Date()
    });
    
    // 4. Insert ledger entries
    for (const entry of journal.entries) {
      // Get current balance
      const currentBalance = await trx('accounts')
        .where({ accountId: entry.accountId })
        .first()
        .then(a => a.balance);
      
      // Calculate new balance
      const newBalance = entry.type === 'DEBIT'
        ? currentBalance - entry.amount
        : currentBalance + entry.amount;
      
      // Insert entry with new balance
      await trx('ledger_entries').insert({
        ...entry,
        balanceAfter: newBalance
      });
      
      // Update account balance
      await trx('accounts')
        .where({ accountId: entry.accountId })
        .update({ balance: newBalance });
    }
  });
}
```

## General Ledger (GL)

### GL Account

```typescript
interface GLAccount {
  glAccountId: string;
  accountCode: string;
  accountName: string;
  accountType: AccountType;
  
  // Balances
  balance: {
    debit: number;
    credit: number;
    net: number;
  };
  
  // Currency
  currency: string;
  
  // Period balances
  periodBalances: Array<{
    period: string;          // YYYY-MM
    openingBalance: number;
    debit: number;
    credit: number;
    closingBalance: number;
  }>;
  
  // Status
  active: boolean;
  lastActivityDate: string;
}
```

### GL Report (Trial Balance)

```typescript
interface TrialBalance {
  asOfDate: string;
  accounts: Array<{
    accountCode: string;
    accountName: string;
    accountType: string;
    debit: number;
    credit: number;
  }>;
  
  totals: {
    totalDebit: number;
    totalCredit: number;
    balanced: boolean;
  };
}

async function generateTrialBalance(
  asOfDate: string
): Promise<TrialBalance> {
  const accounts = await db('ledger_entries')
    .select('accountCode', 'accountName', 'accountType')
    .sum('amount as total')
    .where('bookingDate', '<=', asOfDate)
    .groupBy('accountCode')
    .orderBy('accountCode');
  
  let totalDebit = 0;
  let totalCredit = 0;
  
  const formatted = accounts.map(acc => {
    const isDebitAccount = ['ASSET', 'EXPENSE'].includes(acc.accountType);
    const debit = isDebitAccount ? acc.total : 0;
    const credit = isDebitAccount ? 0 : acc.total;
    
    totalDebit += debit;
    totalCredit += credit;
    
    return {
      accountCode: acc.accountCode,
      accountName: acc.accountName,
      accountType: acc.accountType,
      debit,
      credit
    };
  });
  
  return {
    asOfDate,
    accounts: formatted,
    totals: {
      totalDebit,
      totalCredit,
      balanced: Math.abs(totalDebit - totalCredit) < 0.01
    }
  };
}
```

## Account Statement

### Statement Generation

```typescript
interface AccountStatement {
  accountId: string;
  accountNumber: string;
  accountHolder: string;
  
  // Period
  statementPeriod: {
    from: string;
    to: string;
  };
  
  // Balances
  openingBalance: number;
  closingBalance: number;
  
  // Transactions
  transactions: Array<{
    date: string;
    description: string;
    reference: string;
    debit: number;
    credit: number;
    balance: number;
  }>;
  
  // Summary
  summary: {
    totalDebits: number;
    totalCredits: number;
    transactionCount: number;
  };
}

async function generateStatement(
  accountId: string,
  from: string,
  to: string
): Promise<AccountStatement> {
  // Get opening balance
  const openingBalance = await getBalanceAsOf(accountId, from);
  
  // Get transactions
  const entries = await db('ledger_entries')
    .where({ accountId })
    .whereBetween('valueDate', [from, to])
    .orderBy('valueDate');
  
  const transactions = entries.map(entry => ({
    date: entry.valueDate,
    description: entry.description,
    reference: entry.referenceNumber,
    debit: entry.type === 'DEBIT' ? entry.amount : 0,
    credit: entry.type === 'CREDIT' ? entry.amount : 0,
    balance: entry.balanceAfter
  }));
  
  return {
    accountId,
    accountNumber: await getAccountNumber(accountId),
    accountHolder: await getAccountHolder(accountId),
    statementPeriod: { from, to },
    openingBalance,
    closingBalance: entries[entries.length - 1]?.balanceAfter || openingBalance,
    transactions,
    summary: {
      totalDebits: transactions.reduce((sum, t) => sum + t.debit, 0),
      totalCredits: transactions.reduce((sum, t) => sum + t.credit, 0),
      transactionCount: transactions.length
    }
  };
}
```

## Period Closing

### Month-End Close

```typescript
async function performMonthEndClose(
  year: number,
  month: number
): Promise<void> {
  const period = `${year}-${String(month).padStart(2, '0')}`;
  
  // 1. Verify all transactions are posted
  const unpostedCount = await db('journal_entries')
    .where('status', 'DRAFT')
    .whereRaw('DATE_FORMAT(createdAt, "%Y-%m") = ?', [period])
    .count();
  
  if (unpostedCount > 0) {
    throw new Error('Cannot close period with unposted transactions');
  }
  
  // 2. Generate trial balance
  const trialBalance = await generateTrialBalance(`${period}-31`);
  
  if (!trialBalance.totals.balanced) {
    throw new Error('Trial balance does not balance');
  }
  
  // 3. Calculate period balances for all accounts
  const accounts = await db('gl_accounts').where({ active: true });
  
  for (const account of accounts) {
    const periodBalance = await calculatePeriodBalance(account.glAccountId, period);
    
    await db('gl_period_balances').insert({
      glAccountId: account.glAccountId,
      period,
      openingBalance: periodBalance.openingBalance,
      debit: periodBalance.debit,
      credit: periodBalance.credit,
      closingBalance: periodBalance.closingBalance
    });
  }
  
  // 4. Mark period as closed
  await db('accounting_periods').insert({
    period,
    status: 'CLOSED',
    closedAt: new Date()
  });
  
  // 5. Generate reports
  await generateMonthEndReports(period);
}
```

## Reconciliation

### Auto-reconciliation

```typescript
interface ReconciliationRule {
  ruleId: string;
  name: string;
  
  // Match criteria
  matchOn: ('AMOUNT' | 'REFERENCE' | 'DATE')[];
  tolerance?: {
    amountDiff: number;
    dateDiff: number;      // days
  };
  
  // Account mapping
  sourceAccount: string;
  targetAccount: string;
}

async function autoReconcile(
  internalEntries: LedgerEntry[],
  externalTransactions: ExternalTransaction[]
): Promise<ReconciliationResult> {
  const matched = [];
  const unmatched = { internal: [], external: [] };
  
  for (const internal of internalEntries) {
    let found = false;
    
    for (const external of externalTransactions) {
      if (Math.abs(internal.amount - external.amount) < 0.01 &&
          internal.referenceNumber === external.reference) {
        matched.push({
          internal,
          external,
          status: 'MATCHED'
        });
        found = true;
        break;
      }
    }
    
    if (!found) {
      unmatched.internal.push(internal);
    }
  }
  
  // Find unmatched external
  externalTransactions.forEach(ext => {
    if (!matched.find(m => m.external === ext)) {
      unmatched.external.push(ext);
    }
  });
  
  return {
    matched,
    unmatched,
    matchRate: matched.length / (matched.length + unmatched.internal.length)
  };
}
```

## API Reference

```typescript
// Journal entries
POST /api/v1/ledger/journal-entries
GET /api/v1/ledger/journal-entries/{journalId}
POST /api/v1/ledger/journal-entries/{journalId}/reverse

// Ledger entries
GET /api/v1/ledger/entries
GET /api/v1/ledger/entries/{entryId}

// GL accounts
GET /api/v1/ledger/gl-accounts
GET /api/v1/ledger/gl-accounts/{accountCode}

// Statements
GET /api/v1/ledger/accounts/{accountId}/statement

// Reports
GET /api/v1/ledger/trial-balance
GET /api/v1/ledger/balance-sheet
GET /api/v1/ledger/income-statement

// Period closing
POST /api/v1/ledger/period-close
GET /api/v1/ledger/periods/{period}

// Reconciliation
POST /api/v1/ledger/reconcile
GET /api/v1/ledger/reconciliations
```

## Use Cases trong hệ thống Masan

### 1. Ghi nhận thanh toán của NBL

```typescript
// Customer pays at retailer
const journalEntry = {
  transactionId: 'TXN_001',
  entries: [
    {
      accountCode: '1001',  // Customer Wallet (Asset)
      type: 'CREDIT',
      amount: 100_000,
      description: 'Payment at retailer'
    },
    {
      accountCode: '1002',  // Merchant Account (Asset)
      type: 'DEBIT',
      amount: 100_000,
      description: 'Receive payment from customer'
    }
  ]
};
```

### 2. Ghi nhận phí giao dịch

```typescript
// Record transaction fee
const feeEntry = {
  transactionId: 'TXN_001',
  entries: [
    {
      accountCode: '1002',  // Merchant Account
      type: 'CREDIT',
      amount: 1_000,
      description: 'Transaction fee'
    },
    {
      accountCode: '4000',  // Fee Revenue
      type: 'DEBIT',
      amount: 1_000,
      description: 'Transaction fee revenue'
    }
  ]
};
```

## Best Practices

1. **Integrity**
   - Always balance debits and credits
   - Use database transactions
   - Implement audit trails
   - Never delete entries (reverse instead)

2. **Performance**
   - Index on accountId, valueDate
   - Partition large tables by period
   - Cache account balances
   - Use materialized views for reports

3. **Compliance**
   - Retain records per regulations
   - Implement maker-checker for critical operations
   - Regular reconciliation
   - Audit trail for all changes

## Kết luận

Ledger System đảm bảo tính chính xác và minh bạch của mọi giao dịch tài chính:

- ✅ Double-entry bookkeeping chuẩn
- ✅ Real-time balance updates
- ✅ Comprehensive audit trail
- ✅ Automated reconciliation
- ✅ Regulatory compliance

