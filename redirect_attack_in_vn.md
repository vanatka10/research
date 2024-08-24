Dạo gần đây bực mình ko chịu được search trên google mà toàn ra link chuyển hướng tới web cá độ, bực mình quá nên mình đi research luôn garrrkkkk.

1. Recon
   -Sau một hồi quan sát thì có 1 số điểm chung trong các trang web sau:
   + Đều là web server window IIS của window phiên bản <=10 và những trang web check được ngôn ngữ thì đều được viết bằng ASP.NET 4.0.30319
   + Khi copy link và paste vô chỗ khác gì không bị chuyển hướng và tất cả các link đó đều là đường dẫn tùm lum chỉ dẫn đến 404 page
   + Đa só là các page ít được chăm chút cập nhật thường xuyên như mấy trang gov, lâu lâu đăng tin 1 bài, hoặc là page đăng nhập tra cứu điểm của trường đại học,bên hust còn xài http cơ mà ;v
   
![image](https://github.com/vanatka10/research/assets/126310360/a5ffe4c7-20cd-4645-ba98-d3f85c218769)
2. check
- Dùng F12 xem trang web script gì ko mà sao nó chuyển hướng được hay vậy.
- nothing.. không có gì cả, lạ nhỉ?
-  1 số trang web cũng thế này, mặc dù ko biết có liên quan hay ko nhưng hiển thị exception là hỏng r  
![image](https://github.com/vanatka10/research/assets/126310360/9102bc13-1483-4a24-a113-a29a9573ea3f)
- 1 trang web thường khi được truy cập, 1 đường rất thẳng đúng ko
 ![image](https://github.com/vanatka10/research/assets/126310360/87e93bdd-29b8-485a-911a-289476b6be93)
- Nhưng trang web bị nhiễm thì ko như vậy, nó đổi hướng khá nhanh nên mình dùng burpsuite để bắt request luôn.
![image](https://github.com/vanatka10/research/assets/126310360/808fe6a6-5721-4892-a68e-5f63d8895a1a)
- Nhìn request phía trên cho dù bị obfuscator cũng đoán ra là nó là đoạn javascript chuyển hướng tới web cá độ
3. Phân tích
  Nhưng tại sao khi mình copy và paste nó lại không hiện javascript đó? cùng so sánh 2 request nào:
  *copy:
  ![image](https://github.com/vanatka10/research/assets/126310360/5ccd3978-02c7-4bad-84e0-fbd7370a131f)
  *click từ google:
  ![image](https://github.com/vanatka10/research/assets/126310360/c1441b2a-ef75-437f-9c49-bcbbd99d7dce)
  - Có vẻ là khác ở referer ,khi mình thêm: header ```Referer: https://www.google.com/``` đoạn script đã hiện ra
  
  *Cho ai không biết:Referer là 1 HTTP header xác định địa chỉ của trang web (tức là URI hoặc IRI) liên kết với tài nguyên được yêu cầu. Bằng cách kiểm lại trang giới thiệu, trang web mới có thể thấy yêu cầu bắt nguồn từ đâu. Tóm gọn là cho biết request này được chuyển hướng từ đâu
  - - Bến cạnh đó nó không chỉ xuất hiện ở mỗi uri đó mà bất kì địa chỉ không có trên page làm cho status page 404 thì vẫn bị
  -Từ những thông tin trên mình phỏng đoán có lẽ đây là 1 lỗ hổng CVE google của window server và hacker đã chạy tool để auto detect và exploit nhanh chóng các trang web đó, vậy thử check CVE xem thử
  - 1 tí osint
     ![image](https://github.com/vanatka10/research/assets/126310360/5961386b-b0c0-4914-855b-a03943f62ff3)
  - dạo 1 vòng CVE của ASP.net và IIS thì mình thấy khá nhiều CVE khá liên quan như XSS, HTTP Response Smuggling và có mấy crit vuln nữa như rce. Đa số đều là CVE từ năm 2015 trở lên đúng là version cũ có khác nhiều vuln vl
  - Mình thấy đây là chèn thẳng script vô 404 và có cả check referer để ẩn script nữa chứ không giống store xss lắm, có khi hacker remote được server rồi cũng nên, chắc dùng tool tạo redirect tới web cá độ có tiền hơn,rce làm căng còn có khi bị điều tra ra :)).
  4. Tóm lại
    
    Cập nhật version điiiiiiiiii
  

Lâu lâu vòng lại thì thấy có bài viết của vnpt nói khá rõ về vấn đề này:
https://sec.vnpt.vn/2024/05/part-1-hacker-thuc-hien-black-hat-seo-cac-trang-web-bat-hop-phap-bang-tan-cong-redirect-nhu-the-nao/

-Tên của kỹ thuật này được gọi là:Black Hat SEO

-Tóm tắt nguyên nhân theo bài viết là do hacker đã chèn mã độc vào file cấu hình của webserver cụ thể là ```C:\inetpub\temp\appPools\<site>``` ở trên IIS trong đó bài viết đã nêu 3 cách hacker thường sử dụng để chuyển hướng:
+Chuyển hướng URL bằng cách xử lý các mã lỗi
+Chuyển hướng URL bằng cách chèn các cấu hình độc hại thực hiện redirect trực tiếp
+Chuyển hướng URL bằng cách chèn các module độc hại (.dll trên Windows, .so trên Linux)

1 cách được đề cập bên trên là chuyển hướng xử lý thông báo lỗi 404 được hacker thêm vào tệp .htaccess được mô tả ở hình dưới.
![image](https://github.com/user-attachments/assets/71618ed3-695f-44cf-a90a-aa998e2c2c73)


Còn theo mình research trước đó thì hacker thường sẽ cấu hình chèn thêm những câu lệnh javascript vào phần xử lý lỗi 404 chuyển hướng tới trang web cần seo, theo mình nghĩ tất cả các web server bị như vậy đều sẽ có cùng 1 lỗ hổng chung nên hacker mới có thể tạo script để dùng cho nhiều webserver như vậy. Thường những trang web gov rất ít khi được cập nhật hoặc được theo dõi thường xuyên và chúng thường được cấu hình gần giống nhau (chắc theo quy định ;v) nên rất dễ làm mục tiêu tấn công

