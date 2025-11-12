# Giới thiệu

Cụm Dự án TTGT Masan (Trung gian thanh toán) dựa trên giấy phép trung gian thanh toán, gồm 4 dự án:

1. **Ví điện tử**: triển khai ví điện tử cho các đối tượng khách hàng có đặc tính khác nhau.
2. **Cổng thanh toán và dịch vụ hỗ trợ thu chi hộ**.
3. **Vending machine**: Máy bán hàng tự động.
4. **Retail Supreme**: Tích hợp trung gian thanh toán vào chuỗi phân phối hàng tiêu dùng Masan.

## Dự án Ví điện tử

Ví điện tử hướng đến đối tượng người dùng là nhà bán lẻ hàng tiêu dùng Masan, hội viên Winlife.

### Đặc điểm

- **Nhà bán lẻ hàng tiêu dùng**: tiệm tạp hoá, sạp chợ. Ưu tiên tính năng đơn giản, phục vụ kinh doanh như chấp nhận thanh toán, khai thuế, tín dụng.
- **Hội viên Winlife**: Đa số là người trẻ, sống ở thành thị, ưa thích mua hàng ở siêu thị, cửa hàng tiện lợi. Họ đã đồng ý đăng ký hội viên để tích luỹ điểm, do đó ưu tiên chức năng hiện đại, đầy đủ.
- **Các thành phần khác**: Delivery man của Nhà phân phối, Salespeople của Masan consumer, 2 đối tượng này là những người đi phát triển khách hàng điểm chấp nhận thanh toán.

**Lưu ý đặc biệt**: cần triển khai hệ thống core banking, sử dụng giải pháp của sdk.finance.

## Dự án Cổng thanh toán và dịch vụ hỗ trợ thu chi hộ

Cung cấp hạ tầng thanh toán thông qua:

- **Cổng thanh toán**: kết nối người thanh toán và trung gian thanh toán, chấp nhận nhiều phương thức thanh toán: qrcode, facepay, thẻ emv.
- **Dịch vụ hỗ trợ thu hộ**: giúp merchant triển khai giải pháp thanh toán điện tử.
- **Dịch vụ hỗ trợ chi hộ**: kết nối chặt chẽ với tài khoản ví của merchant để thực hiện chi hộ theo lệnh của merchant.

## Dự án Vending Machine

Triển khai hệ thống máy bán hàng tự động, gồm:

- Phần cứng và firmware mua của nhà sản xuất.
- Thực hiện phát triển phần mềm và tích hợp hệ thống thanh toán.

## Dự án Retail Supreme

### Định nghĩa

Retail Supreme là dự án tối ưu hoá quy trình phân phối hàng tiêu dùng Masan. Dự án Trung gian thanh toán tham gia với vai trò nghiên cứu tích hợp các giải pháp thanh toán, tài chính vào quy trình phân phối hàng hoá để giảm lượng thanh toán tiền mặt, tăng giao dịch không tiền mặt.

### Từ viết tắt

- **MCH**: Masan Consumer Holdings
- **NPP**: Đại lý phân phối hàng tiêu dùng Masan ở từng khu vực
- **Salesman**: Đại diện kinh doanh/ Salespeople của MCH
- **Shipper**: Người giao hàng của NPP
- **NBL**: Nhà bán lẻ/ Retailers
- **NTD**: Người tiêu dùng/ consumers
- **DMS**: Distribution Management System