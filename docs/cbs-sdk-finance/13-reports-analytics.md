# XIII. Reports & Analytics - Báo cáo và Phân tích

## Tổng quan

Module Reports & Analytics cung cấp insights về hoạt động kinh doanh, performance metrics, và regulatory reports.

## Dashboard

```typescript
interface Dashboard {
  // Overview metrics
  overview: {
    totalUsers: number;
    activeUsers: number;
    totalTransactions: number;
    totalVolume: number;
    revenue: number;
  };
  
  // Trends
  trends: {
    userGrowth: TimeSeriesData;
    transactionVolume: TimeSeriesData;
    revenue: TimeSeriesData;
  };
  
  // Breakdowns
  breakdowns: {
    byUserType: PieChartData;
    byTransactionType: PieChartData;
    byPaymentMethod: PieChartData;
    byRegion: MapData;
  };
}
```

## Standard Reports

### 1. Transaction Report

```typescript
interface TransactionReport {
  period: { from: string; to: string };
  
  summary: {
    totalCount: number;
    totalAmount: number;
    avgTransactionSize: number;
    successRate: number;
  };
  
  byType: Array<{
    type: string;
    count: number;
    amount: number;
    percentage: number;
  }>;
  
  byMethod: Array<{
    method: string;
    count: number;
    amount: number;
  }>;
  
  byHour: Array<{
    hour: number;
    count: number;
    amount: number;
  }>;
  
  topMerchants: Array<{
    merchantId: string;
    merchantName: string;
    transactionCount: number;
    totalAmount: number;
  }>;
}
```

### 2. Revenue Report

```typescript
interface RevenueReport {
  period: { from: string; to: string };
  
  total: {
    transactionFees: number;
    serviceFees: number;
    interestIncome: number;
    other: number;
    totalRevenue: number;
  };
  
  byProduct: Record<string, number>;
  byCustomerSegment: Record<string, number>;
  
  trends: {
    daily: TimeSeriesData;
    monthly: TimeSeriesData;
  };
  
  forecast: {
    nextMonth: number;
    nextQuarter: number;
  };
}
```

### 3. Customer Report

```typescript
interface CustomerReport {
  totalCustomers: number;
  newCustomers: number;
  activeCustomers: number;
  churnedCustomers: number;
  
  segments: Array<{
    segment: string;
    count: number;
    avgBalance: number;
    avgTransactionCount: number;
    ltv: number;              // Lifetime Value
  }>;
  
  cohortAnalysis: Array<{
    cohort: string;
    retention: Record<string, number>;
  }>;
}
```

### 4. Financial Report

```typescript
interface FinancialReport {
  balanceSheet: {
    assets: {
      cash: number;
      accountsReceivable: number;
      investments: number;
      total: number;
    };
    liabilities: {
      customerDeposits: number;
      accountsPayable: number;
      total: number;
    };
    equity: {
      capital: number;
      retainedEarnings: number;
      total: number;
    };
  };
  
  incomeStatement: {
    revenue: number;
    costOfRevenue: number;
    grossProfit: number;
    operatingExpenses: number;
    operatingIncome: number;
    netIncome: number;
  };
  
  cashFlow: {
    operating: number;
    investing: number;
    financing: number;
    netCashFlow: number;
  };
}
```

## Real-time Analytics

```typescript
interface RealtimeAnalytics {
  // Current metrics (last 5 minutes)
  current: {
    tps: number;                    // Transactions per second
    activeUsers: number;
    pendingTransactions: number;
    systemLoad: number;
  };
  
  // Alerts
  alerts: Array<{
    type: string;
    severity: string;
    message: string;
    timestamp: string;
  }>;
}
```

## Custom Reports

```typescript
interface CustomReport {
  reportId: string;
  name: string;
  description: string;
  
  // Query
  query: {
    dataSource: string;
    filters: Array<{
      field: string;
      operator: string;
      value: any;
    }>;
    groupBy: string[];
    aggregations: Array<{
      field: string;
      function: 'SUM' | 'AVG' | 'COUNT' | 'MIN' | 'MAX';
    }>;
  };
  
  // Visualization
  visualization: {
    type: 'TABLE' | 'LINE' | 'BAR' | 'PIE' | 'MAP';
    options: Record<string, any>;
  };
  
  // Schedule
  schedule?: {
    frequency: 'DAILY' | 'WEEKLY' | 'MONTHLY';
    time: string;
    recipients: string[];
  };
}
```

## Data Export

```typescript
interface ExportRequest {
  reportType: string;
  period: { from: string; to: string };
  format: 'CSV' | 'EXCEL' | 'PDF' | 'JSON';
  filters?: Record<string, any>;
  
  // Delivery
  delivery: {
    method: 'DOWNLOAD' | 'EMAIL' | 'SFTP';
    recipient?: string;
  };
}

async function exportReport(request: ExportRequest): Promise<ExportResult> {
  // Generate report
  const data = await generateReport(request);
  
  // Format data
  let file: Buffer;
  switch (request.format) {
    case 'CSV':
      file = await formatAsCSV(data);
      break;
    case 'EXCEL':
      file = await formatAsExcel(data);
      break;
    case 'PDF':
      file = await formatAsPDF(data);
      break;
  }
  
  // Deliver
  if (request.delivery.method === 'EMAIL') {
    await emailFile(file, request.delivery.recipient);
  }
  
  return {
    fileUrl: await uploadFile(file),
    expiresAt: addDays(new Date(), 7)
  };
}
```

## Regulatory Reports

```typescript
// Báo cáo tuân thủ cho NHNN (Ngân hàng Nhà nước)
interface RegulatoryReport {
  reportId: string;
  reportType: 'MONTHLY_TRANSACTION' | 'QUARTERLY_FINANCIAL' | 'ANNUAL_AUDIT';
  period: string;
  
  data: {
    transactionSummary: {...};
    largeTransactions: Array<{...}>;  // > 400 triệu
    suspiciousActivities: Array<{...}>;
  };
  
  status: 'DRAFT' | 'SUBMITTED' | 'ACCEPTED';
  submittedAt?: string;
}
```

## API Reference

```typescript
GET /api/v1/reports/dashboard
GET /api/v1/reports/transactions
GET /api/v1/reports/revenue
GET /api/v1/reports/customers
GET /api/v1/reports/financial

POST /api/v1/reports/custom
POST /api/v1/reports/export
GET /api/v1/reports/scheduled
```

## Best Practices

- Cache frequently accessed reports
- Pre-aggregate data for better performance
- Implement data retention policies
- Regular backup of analytics data
- Role-based access to sensitive reports

## Kết luận

Reports & Analytics giúp data-driven decision making:
- ✅ Comprehensive dashboards
- ✅ Standard và custom reports
- ✅ Real-time analytics
- ✅ Regulatory compliance
- ✅ Data export flexibility

