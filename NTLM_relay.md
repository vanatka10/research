# NTLM relay attack
## 1.	What is NTLM?
-	NTLM là 1 giao thức xác thực của windows, Nó sử dụng cơ chế challenge/response để xác thực user và máy tính. NTLM được ra mắt vào năm 1993 và được cải thiện thêm vào năm 1998. NTLM còn có thể dùng để lưu trữ mật khẩu(NTLM hash)
-	Quá trình xác thực của NTLM:
 ![image](https://github.com/vanatka10/research/assets/126310360/7f4ef4b3-d543-4c19-87ad-6f7f1c1890d3)

+ Client gửi request yêu cầu vào các tài nguyên mạng(like smb,).
+ Server gửi challenge cho client, đó là 1 giá trị 16-byte ngẫu nhiên(nonce), cần thiết trong quá trình authentication.
+ Client encrypt challenge này bằng hash của mật khẩu người dùng và trả kết quả về server.
+ Server sẽ gửi 3 thứ này cho domain controller:
•	Username
•	Challenge gửi cho client
•	Response nhận từ client
+ Domain controller sử dụng username để truy xuất hàm băm của mật khẩu người dùng từ cơ sở dữ liệu. Nó dùng hash của mật khẩu để mã hóa thử thách.
+ domain controller so sánh challenge được mã hóa mà nó đã mã hóa với response challenge do client gửi . Nếu chúng giống hệt nhau thì xác thực thành công.
## 2.	What is NTLM relay?
-	Về cơ bản trong NTLM relay attack, attacker sẽ nằm giữa client và server trên đường mạng để bắt các gói tin và chuyển tiếp request cho server giống như MITM. Khi đã được xác thực, quyền truy cập đó sẽ được cung cấp cho attacker chứ không phải client.
-	Trong relay attack, client tin rằng nó đang xác thực  với server mục tiêu mà nó muốn xác thực. Trong khi đó, server tin rằng attacker là một client đang cố gắng xác thực.
## 3.	Exploit 
 ![image](https://github.com/vanatka10/research/assets/126310360/577be033-e505-4062-b08e-789785f9d7d2)

Sử dụng crackapexec để kiểm tra chức năng smb signing có được bật không
![image](https://github.com/vanatka10/research/assets/126310360/5a05fe44-f90a-4646-a621-71b8c096ac50)


 
Dùng responder để listening  các event
![image](https://github.com/vanatka10/research/assets/126310360/b9e44290-719f-443a-b2bb-9b1ac283e585)

 
Truy cập vào smb bất kì mà không cần đăng nhập
 ![image](https://github.com/vanatka10/research/assets/126310360/eb9d7cc4-e1d9-4ae7-9fff-22433a608778)


Bên cạnh nó nếu smb signing được bật thì attacker vẫn có thể lấy hash từ credentials client khi login.
## 4.	Prevention
-	Bật chức năng SMB Signing.
-	Dùng Kerberos thay thế cho NTLM.
-	Bật EPA và tắt HTTP trên máy chủ AD CS.



Reference: https://security.stackexchange.com/questions/129832/understanding-ntlm-authentication-step-by-step
https://warroom.rsmus.com/how-to-perform-ntlm-relay/
 https://support.microsoft.com/vi-vn/topic/kb5005413-giảm-thiểu-tấn-công-chuyển-tiếp-ntlm-trên-dịch-vụ-chứng-chỉ-active-directory-ad-cs-3612b773-4043-4aa9-b23d-b87910cd3429
