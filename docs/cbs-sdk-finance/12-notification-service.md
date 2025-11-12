# XII. Notification Service - Dịch vụ Thông báo

## Tổng quan

Notification Service gửi thông báo đến khách hàng qua nhiều kênh (Push, SMS, Email, In-app) để cập nhật về giao dịch, cảnh báo bảo mật, và thông tin marketing.

## Notification Channels

```typescript
enum NotificationChannel {
  PUSH = 'PUSH',               // Push notification
  SMS = 'SMS',                 // Tin nhắn SMS
  EMAIL = 'EMAIL',             // Email
  IN_APP = 'IN_APP',           // In-app notification
  WEBHOOK = 'WEBHOOK'          // Webhook to external system
}

interface Notification {
  notificationId: string;
  userId: string;
  
  // Content
  type: NotificationType;
  title: string;
  body: string;
  data: Record<string, any>;
  
  // Channel
  channels: NotificationChannel[];
  
  // Priority
  priority: 'LOW' | 'NORMAL' | 'HIGH' | 'URGENT';
  
  // Status
  status: 'PENDING' | 'SENT' | 'DELIVERED' | 'FAILED' | 'READ';
  sentAt?: string;
  deliveredAt?: string;
  readAt?: string;
  
  // Retry
  retryCount: number;
  maxRetries: number;
}
```

## Notification Types

```typescript
enum NotificationType {
  // Transaction
  TRANSACTION_SUCCESS = 'TRANSACTION_SUCCESS',
  TRANSACTION_FAILED = 'TRANSACTION_FAILED',
  PAYMENT_RECEIVED = 'PAYMENT_RECEIVED',
  
  // Account
  ACCOUNT_CREATED = 'ACCOUNT_CREATED',
  ACCOUNT_VERIFIED = 'ACCOUNT_VERIFIED',
  BALANCE_LOW = 'BALANCE_LOW',
  
  // Security
  LOGIN_SUCCESS = 'LOGIN_SUCCESS',
  LOGIN_FAILED = 'LOGIN_FAILED',
  PASSWORD_CHANGED = 'PASSWORD_CHANGED',
  SUSPICIOUS_ACTIVITY = 'SUSPICIOUS_ACTIVITY',
  
  // Card
  CARD_ISSUED = 'CARD_ISSUED',
  CARD_ACTIVATED = 'CARD_ACTIVATED',
  CARD_BLOCKED = 'CARD_BLOCKED',
  
  // Marketing
  PROMOTION = 'PROMOTION',
  PRODUCT_ANNOUNCEMENT = 'PRODUCT_ANNOUNCEMENT',
  
  // System
  MAINTENANCE = 'MAINTENANCE',
  SYSTEM_UPDATE = 'SYSTEM_UPDATE'
}
```

## Push Notifications

```typescript
async function sendPushNotification(notification: Notification): Promise<void> {
  // Get user's devices
  const devices = await getUserDevices(notification.userId);
  
  for (const device of devices) {
    try {
      if (device.platform === 'IOS') {
        await sendAPNS({
          deviceToken: device.token,
          notification: {
            title: notification.title,
            body: notification.body,
            data: notification.data,
            badge: await getUnreadCount(notification.userId),
            sound: notification.priority === 'URGENT' ? 'urgent.mp3' : 'default'
          }
        });
      } else if (device.platform === 'ANDROID') {
        await sendFCM({
          deviceToken: device.token,
          notification: {
            title: notification.title,
            body: notification.body
          },
          data: notification.data,
          priority: notification.priority === 'URGENT' ? 'high' : 'normal'
        });
      }
    } catch (error) {
      console.error('Failed to send push:', error);
    }
  }
}
```

## SMS Notifications

```typescript
async function sendSMS(notification: Notification): Promise<void> {
  const user = await getUser(notification.userId);
  
  // Format message
  const message = formatSMSMessage(notification);
  
  // Send via SMS provider
  await smsProvider.send({
    to: user.phoneNumber,
    from: SMS_SENDER_NAME,
    message,
    priority: notification.priority
  });
}

function formatSMSMessage(notification: Notification): string {
  const templates = {
    TRANSACTION_SUCCESS: `[Masan] Giao dich ${notification.data.amount}đ thanh cong. So du: ${notification.data.balance}đ`,
    LOGIN_SUCCESS: `[Masan] Dang nhap thanh cong tai ${notification.data.location} luc ${notification.data.time}`,
    SUSPICIOUS_ACTIVITY: `[Masan] CANH BAO: Phat hien hoat dong bat thuong. Vui long lien he hotline ngay.`
  };
  
  return templates[notification.type] || notification.body;
}
```

## Email Notifications

```typescript
async function sendEmail(notification: Notification): Promise<void> {
  const user = await getUser(notification.userId);
  
  // Get email template
  const template = await getEmailTemplate(notification.type);
  
  // Render template with data
  const html = await renderTemplate(template, {
    userName: user.fullName,
    ...notification.data
  });
  
  // Send email
  await emailProvider.send({
    to: user.email,
    from: EMAIL_FROM_ADDRESS,
    subject: notification.title,
    html,
    priority: notification.priority
  });
}
```

## Templates

```typescript
interface NotificationTemplate {
  templateId: string;
  type: NotificationType;
  channel: NotificationChannel;
  
  // Multi-language support
  translations: {
    [lang: string]: {
      title: string;
      body: string;
      htmlTemplate?: string;
    };
  };
  
  // Variables
  variables: string[];
  
  active: boolean;
}

// Example template
const TRANSACTION_SUCCESS_TEMPLATE = {
  templateId: 'TXN_SUCCESS_001',
  type: 'TRANSACTION_SUCCESS',
  channel: 'PUSH',
  translations: {
    vi: {
      title: 'Giao dịch thành công',
      body: 'Bạn đã chuyển {{amount}} đến {{recipient}}. Số dư: {{balance}}'
    },
    en: {
      title: 'Transaction Successful',
      body: 'You transferred {{amount}} to {{recipient}}. Balance: {{balance}}'
    }
  },
  variables: ['amount', 'recipient', 'balance']
};
```

## User Preferences

```typescript
interface NotificationPreferences {
  userId: string;
  
  // Channel preferences
  channels: {
    push: boolean;
    sms: boolean;
    email: boolean;
    inApp: boolean;
  };
  
  // Type preferences
  types: {
    transactional: boolean;     // Cannot disable
    security: boolean;           // Cannot disable
    marketing: boolean;
    promotional: boolean;
    news: boolean;
  };
  
  // Quiet hours
  quietHours: {
    enabled: boolean;
    start: string;              // e.g., '22:00'
    end: string;                // e.g., '08:00'
    channels: NotificationChannel[]; // Affected channels
  };
  
  // Language
  language: 'vi' | 'en';
}
```

## Batch Notifications

```typescript
async function sendBatchNotifications(
  userIds: string[],
  notification: Omit<Notification, 'userId' | 'notificationId'>
): Promise<void> {
  // Split into batches
  const batchSize = 1000;
  const batches = chunk(userIds, batchSize);
  
  for (const batch of batches) {
    await Promise.all(
      batch.map(userId => 
        sendNotification({
          ...notification,
          userId,
          notificationId: generateId()
        })
      )
    );
  }
}
```

## API Reference

```typescript
POST /api/v1/notifications/send
GET /api/v1/notifications
GET /api/v1/notifications/{notificationId}
POST /api/v1/notifications/{notificationId}/mark-read

// Preferences
GET /api/v1/notifications/preferences
PUT /api/v1/notifications/preferences

// Templates (admin)
GET /api/v1/admin/notification-templates
POST /api/v1/admin/notification-templates
```

## Best Practices

- Respect user preferences
- Quiet hours support
- Rate limiting to avoid spam
- A/B testing for marketing notifications
- Track open rates and engagement

## Kết luận

Notification Service giữ liên lạc với khách hàng:
- ✅ Multi-channel delivery
- ✅ Template-based content
- ✅ User preferences respected
- ✅ Real-time và scheduled
- ✅ Analytics and tracking

