# Tài liệu Core Banking SDK.Finance - Masan TTGT

Tài liệu chi tiết về hệ thống Core Banking SDK.Finance cho dự án Masan TTGT, được xây dựng bằng [Docusaurus 3](https://docusaurus.io/).

## 📚 Nội dung Tài liệu

Tài liệu bao gồm 16 modules chi tiết:

1. **Giới thiệu CBS SDK.Finance** - Tổng quan về hệ thống
2. **Kiến trúc Hệ thống** - Kiến trúc 6 lớp, microservices, API Gateway
3. **Account Management** - Quản lý tài khoản khách hàng
4. **Transaction Service** - Xử lý giao dịch
5. **Payment Service** - Dịch vụ thanh toán (QR, Bank Transfer, Card)
6. **Merchant Management** - Quản lý thương nhân
7. **Credit Service** - Quản lý tài khoản tín dụng
8. **Card Issuing & Processing** - Phát hành và xử lý thẻ
9. **Ledger System** - Hệ thống sổ cái (Double-entry Bookkeeping)
10. **KYC/KYB** - Xác minh danh tính khách hàng (5 levels, AML Screening)
11. **Risk Management** - Quản lý rủi ro
12. **Notification Service** - Dịch vụ thông báo
13. **Reports & Analytics** - Báo cáo và phân tích
14. **User & Access Management** - Quản lý người dùng và phân quyền
15. **Configuration & Settings** - Cấu hình hệ thống
16. **API Gateway** - Cổng API

## 🚀 Cài đặt

### Yêu cầu

- Node.js >= 20.0
- npm hoặc yarn

### Cài đặt Dependencies

```bash
npm install
```

hoặc

```bash
yarn install
```

## 💻 Chạy Development Server

```bash
npm start
```

hoặc

```bash
yarn start
```

Lệnh này sẽ khởi động development server tại `http://localhost:3000`. Hầu hết các thay đổi sẽ được cập nhật ngay lập tức mà không cần restart server.

## 🏗️ Build

```bash
npm run build
```

Lệnh này sẽ tạo static content vào thư mục `build`, có thể được host trên bất kỳ static hosting service nào.

## 📖 Cách Viết và Cập nhật Tài liệu

### Cấu trúc Thư mục

```
docs/
├── cbs-sdk-finance/          # Tài liệu Core Banking
│   ├── _category_.json       # Cấu hình category
│   ├── 01-intro.md          # Giới thiệu
│   ├── 02-architecture.md   # Kiến trúc
│   ├── 03-account-management.md
│   └── ...                   # Các modules khác
├── msn-proj-intro.md        # Giới thiệu dự án Masan
└── intro.md                 # Trang chủ docs
```

### Thêm/Sửa Tài liệu

1. Mở file markdown (.md) tương ứng trong thư mục `docs/`
2. Chỉnh sửa nội dung theo format Markdown
3. Lưu file - Docusaurus sẽ tự động reload

### Thêm Sơ đồ Mermaid

Tài liệu hỗ trợ Mermaid diagrams:

```markdown
\`\`\`mermaid
graph TD
    A[Start] --> B[Process]
    B --> C[End]
\`\`\`
```

### Thêm Code Examples

```markdown
\`\`\`typescript
interface Example {
  id: string;
  name: string;
}
\`\`\`
```

## 🤝 Cộng tác

### Clone Repository

```bash
git clone https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git
cd YOUR_REPO_NAME
npm install
```

### Workflow

1. **Tạo Branch mới** cho tính năng/sửa lỗi:
   ```bash
   git checkout -b feature/ten-tinh-nang
   ```

2. **Commit thay đổi**:
   ```bash
   git add .
   git commit -m "Mô tả thay đổi"
   ```

3. **Push branch**:
   ```bash
   git push origin feature/ten-tinh-nang
   ```

4. **Tạo Pull Request** trên GitHub

### Quy tắc Commit Message

- `feat:` - Thêm tính năng mới
- `fix:` - Sửa lỗi
- `docs:` - Cập nhật tài liệu
- `style:` - Format code, không thay đổi logic
- `refactor:` - Refactor code
- `chore:` - Cập nhật dependencies, config

Ví dụ:
```bash
git commit -m "docs: Cập nhật hướng dẫn KYC Level 4 và 5"
git commit -m "feat: Thêm sơ đồ Ledger System"
```

## 🌐 Deploy

### Deploy lên GitHub Pages

```bash
npm run build
```

Sau đó config GitHub Pages trong repository settings để serve từ thư mục `build/`.

### Deploy lên Vercel/Netlify

1. Connect repository với Vercel/Netlify
2. Build command: `npm run build`
3. Output directory: `build`
4. Deploy!

## 📝 Cấu hình

Cấu hình chính nằm trong file `docusaurus.config.ts`:

- **Title & Tagline**: Thông tin hiển thị
- **Theme**: Cấu hình giao diện
- **Navbar**: Menu điều hướng
- **Footer**: Chân trang

## 🛠️ Công nghệ Sử dụng

- [Docusaurus 3.9.2](https://docusaurus.io/) - Static site generator
- [React 19](https://react.dev/) - UI framework
- [TypeScript](https://www.typescriptlang.org/) - Type safety
- [Mermaid](https://mermaid.js.org/) - Diagrams
- [Prism](https://prismjs.com/) - Syntax highlighting

## 📧 Liên hệ

Nếu có câu hỏi hoặc cần hỗ trợ, vui lòng tạo issue trên GitHub hoặc liên hệ team.

## 📄 License

[Thêm license nếu cần]

---

**Lưu ý**: Tài liệu này dành cho mục đích nội bộ dự án Masan TTGT. Vui lòng không chia sẻ ra bên ngoài.
