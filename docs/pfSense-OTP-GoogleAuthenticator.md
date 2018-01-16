## Hướng dẫn cấu hình xác thực OTP sử dụng Google Authenticator cho PFSense

### Mục tiêu

Bài lab hướng dẫn việc sử dụng phương thức xác thực 1 lần OTP Google Authenticator để xác thực user khi quay VPN tới PFSense

## Mô hình 
- Sử dụng mô hình dưới để cài đặt
![img](../images/PFSense.jpg)

## IP Planning
- Phân hoạch IP cho các máy chủ trong mô hình trên
![img](../images/ip-planning-pfsense.jpg)

## Chuẩn bị và môi trường LAB
Hệ thống VMWare Workstation với các card mạng sau
- VMNet8: NAT ra Internet.
- VMnet3: LAN2 (giả lập mạng private trong PFSense)
- VMnet1: Host-only (quản lý PFSense)
![img](../images/vmware_net.jpg)
 

## Tạo máy ảo PFSense trên VMWare
- Cấu hình máy ảo và card mạng như sau:
![img](../images/configuration.jpg)

Lưu ý: OS chọn Other/FreeBSD (32 hoặc 64 tùy phiên bản)

![img](../images/free_bsd_version.jpg)

## Cài đặt PFSense
- Thực hiện theo hướng dẫn [sau](pfSense-install.md)

## Cấu hình VPN mode TAP theo hướng dẫn [sau](pfSense-OpenVPN-TAPmode.md)

## Cấu hình sử dụng GoogleAuth

- Vào System > PackageManager > Available Packager, tìm và cài đặt gói freeradius3
![img](../images/GoogleAuth/1.jpg)

- Vào Services > FreeRADIUS > Interfaces, Add thêm interface cho freeradius
![img](../images/GoogleAuth/2.jpg)

![img](../images/GoogleAuth/3.jpg)

- Vào Services > FreeRADIUS > NAS/Clients, tạo NAS client
![img](../images/GoogleAuth/4.jpg)

- Vào Services > FreeRADIUS > Users, tạo user được chứng thực bởi Radius
![img](../images/GoogleAuth/9.jpg)

- Lựa chọn sử dụng One-time password, sau đó dùng ứng dụng Google authenticator để quét QR Code hoặc nhấn vào "Generate OTP Secret" để lấy init secret
![img](../images/GoogleAuth/10.jpg)

![img](../images/GoogleAuth/11.png)

- Vào System > User Manager > Authentication Servers, tạo server chứng thực. Lưu ý: sử dụng Shared secret đã khai báo ở bước tạo NAS Client
![img](../images/GoogleAuth/5.jpg)

- Vào VPN > OpenVPN > Servers, sửa lại Authentication Backend của VPN Server là Freeradius server vừa tạo ở trên
![img](../images/GoogleAuth/6.jpg)

- Mặc định, OpenVPN Client sẽ thực hiện bắt tay lại với server sau 3600s, do Google OTP chỉ có hiệu lực trong 30s, do đó, việc này sẽ khiến Client bị mất kết nối và phải log in lại sau 3600s. Để disable việc bắt tay lại này, thực hiện như sau.
Tại VPN > OpenVPN > Servers, Thêm `reneg-sec 0`  vào "Custom Options"
![img](../images/GoogleAuth/7.jpg)

- Tại VPN > OpenVPN > Client Export Utility, Thêm `reneg-sec 0`  vào "Additional configuration options"
![img](../images/GoogleAuth/8.jpg)

- Để kiểm tra việc xác thực bằng freeradius, vào Diagnostics > Authentication, nhập user vừa tạo và password. Lưu ý, password = PIN + Google Authenticator code.
![img](../images/GoogleAuth/12.jpg)

- Vào VPN > OpenVPN > Client Export Utility, download gói cài đặt openvpn cho client và cài đặt.

- Khi nhận được yêu cầu xác thực, nhập tên user và password.
![img](../images/GoogleAuth/13.jpg)


Tham khảo:

[1] - https://vorkbaard.nl/how-to-set-up-openvpn-with-google-authenticator-on-pfsense/

