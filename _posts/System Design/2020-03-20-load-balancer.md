---
layout: post
title: "Load Balancer"
category: system-design
author: lotus
short-description: Load Balancer là gì? Tại sao chúng ta cần nó? Những thuật toán được sử dụng cho Load Balancer
---

-----

Load Balancer đóng vai trò rất quan trọng trong thiết kế hệ thống. Vậy:
- Load Balancer là gì? Tại sao các hệ thống cần có Load Balancer?
- Các thuật toán nào được sử dụng cho Load Balancer?

Mình sẽ cùng các bạn trả lời các câu hỏi ở trên nhé.

### I. Load Balancer là gì? Tại sao các hệ thống cần có Load Balancer?

Chuyện kể rằng bà Ba mở một cửa hàng bán khẩu trang handmade online ở Hà Đông, 1 ngày bà bán hết cống suất được 10 khách mỗi ngày do tuổi đã già, sức đã yếu. Nhưng được cái giao hàng trong ngày, chất lượng đảm bảo.

*image*

Nhưng rồi năm 2020, mùa dịch COVID-19 bùng phát, 1 ngày bà Ba tiếp nhận đến 1000 đơn hàng. Do đó hàng bị giao chậm, khách hàng kêu ca nhiều. Bà Ba quyết định mở thêm 300 cơ sở và giao quyền cho bà Một, bà Hai, bà Tứ, ..., bà Bách quản lý, mỗi cơ sở dự tính 1 ngày tiếp nhận 10 đơn hàng. **Vấn đề phát sinh bây giờ là, làm sao để khách hàng biết được đến 2 cơ sở mới của bà?** Vì chi phí bỏ ra để marketing cho 300 cơ sở mới khá là tốn kém.

May mắn thay do hồi trẻ chăm chỉ đọc toidicodedao kèm theo một chút kiến thức về thuật toán, python, HTML + CSS + Javascript. Bà Ba đã tự viết một website bán hàng cho riêng mình với tên miền là khautrangbaba.com 
Giờ mọi chuyện thuận lời hơn rất nhiều, khách hàng chỉ cần đặt hàng trên website khautrangbaba.com, nhờ được cài đặt 1 vài thuật toán đơn giản, website này sẽ tự chia đều đơn hàng đến các cơ sở của bà Ba, kể cả khi bà Ba mở thêm thêm 1 cơ sở mới, và xóa nhòa nỗi đau mất khách hàng khi 1 cơ sở nào đó ngừng hoạt động.

*image*


Ở trong ví dụ này:
- **khách hàng** là các máy **client**
- **cơ sở của ba Ba** là các **server**
- **website khautrangbaba.com** đóng vai trò như **1 Load Balancer**

Kết luận lại, Load Balancer là một server được sử dụng để cải hiện hiệu suất của dịch vụ (response nhanh hơn, xử lý được nhiều request trên 1 giây hơn) bằng cách phân phối khối lượng công việc 1 cách hợp lý trên nhiều server. 
Nhờ có Load Balancer việc thêm mới 1 server, chỉ cần config cho Load Balancer biết sự tồn tại của server mới. Hay khi có 1 server trong cụm bị hỏng hay quá tải, Load Balancer sẽ chia khối lượng công việc của server đó cho những server còn lại để đảm bảo hệ thống vẫn hoạt động.


**Các bạn có thắc mắc như ví dụ ở trên, Load Balancer (website khautrangbaba.com) hoàn toàn có thể bị quả tải khi lượng request nhiều lên không?**

*answer*
*image*

Trên thực tế mỗi service không chỉ có 1 Load Balancer. Các Load Balancer 1, 2 sẽ quản lý các Load Balancer ở layer thấp hơn, tương ứng 1.1, 1.2, 2.1, ..., để đảm bảo **High availability** cho hệ thống. Load Balancer 1 và 2 có thể giao tiếp trực tiếp với nhau để phân chia lượng request. 

Load Balancer có thể được đặt ở rất nhiều layer:
- DNS layer.
- Giữa client và server.
- Giữa server và database.
- Giữa chính những Load Balancer khác. 


### II. Các thuật toán được áp dụng cho Load Balancer.
**Theo ví dụ ở trên, website khautrangbaba.com của bà Ba đã dùng thuật toán gì để chia đều đơn hàng đến các cơ sở?**

#### 1. Random load balancing
Chúng ta có 2 cách chọn ngẫu nhiên ở đây.

**Cách 1**: Với mỗi request, Load Balancer sẽ chọn ngẫu nhiên 1 server để xử lý request.
**Cách 2**: Load Balancer sẽ cho client biết được IP của tất cả server trong cụm. Việc lựa chọn server nào sẽ do client quyết định.

**nhược điểm** của phương pháp này rất rõ ràng, vì là chọn ngẫu nhiên nên 1 cơ sở hoàn toàn có thể rơi vào tình trạng quá tải.

#### 2. Round-Robin
Các server sẽ được lựa chọn theo tuần tự. Khi request đầu tiên đến, Load Balancer sẽ chọn máy chủ đầu tiên trong danh sách của mình để xử lý request, request thứ 2 sẽ được xử lý bởi server thứ 2, request thứ 3 được xử lý bởi server thứ 3. Cứ như vậy đến hết server cuối cùng, request mới sẽ được xử lý bởi server đầu tiên, và tiếp tục tăng dần.

**Round-Robin DNS**
Phương pháp Round-Robin có thể được thực hiện ngay trên DNS layer. Rất đơn giản, các bạn chỉ cần config cho 1 domain name gắn với nhiều địa chỉ IP.

```
khautrangbaba.com 192.168.2.16
khautrangbaba.com 192.168.2.17
khautrangbaba.com 192.168.2.18
```

Khí request đến, các server sẽ được gọi tuần tự 192.168.2.16, 192.168.2.17, 192.168.2.18, rồi quay vòng về 192.168.2.16 cho request tiếp theo.

**Nhược điểm:** 
- Do các server mạnh yếu khác nhau, nên việc sử dụng Round-Robin có thể làm 1 server bị quá tải trong khi server khác lại quá nhàn rỗi. Ví dụ, 192.168.2.16 có thông số Core i9 - RAM 16GB - ổ đĩa SSD - xử lý được 8 reuqest trên 1 giây, trong khi 192.168.2.18 CPU Pentium - RAM 2GB - ổ đĩa HĐD xử lý được 1 request trên 1 giây.
- Việc caching trong phương pháp này không được tối ưu. Giả sử client thực hiện 2 request giống nhau trên cùng 1 máy, thì request thứ nhất sẽ được xử lý bởi server 192.168.2.16, còn request thứ hai được thực hiện trên server 192.168.2.17, với cùng request giống nhau nhưng phải query vào DB 2 lần thực sự lẵng phí. **Don't repeat yourself**.

**Weighted-Round-Robin**
Là một phương pháp cải tiến của Round-Robin. Cho phép đánh trọng số (weight) với mỗi server, những server có trọng số cao hơn sẽ được Load Balancer ưu tiên cho xử lý nhiều request hơn.

#### 3. IP Hash load balancing
Giả sử ba Ba có 3 server và 1 Load Balancer. Khi nhận được request từ client có IP 192.168.1.1, IP này sẽ được cho qua 1 hàm **hash** và cho ra kết quả là 17. Con số 17 này sẽ được chia cho tổng số server, lấy phần dư - tương đương với số thự tự của server sẽ xử lý yêu cầu

```
17 % 3 => 2
server thứ 2 sẽ nhận và xử lý yêu cầu của client có IP 192.168.1.1
```

**Ưu điểm:**
- Dễ thấy là phương pháp này đã tái sử dụng được cache 1 cách hiệu quả hơn. Như ví dụ trên sau khi trước khi trả kết quả cho client có IP 192.168.1.1, kết quả này sẽ được cache lại trên server 2. Những lần gọi request tượng tự của client IP 192.168.1.1 sau này sẽ **cache hit** và được trả về nhanh hơn.

**Nhược điểm:**
- Khi thêm mới hoặc xóa đi 1 server, vấn đề trở nên phức tạp hơn. Giả sử khi chỉ còn 2 server, với cùng request IP 192.168.1.1, server được chỉ định để xử ly request lúc này phải là 1 (17 % 2).

**More info coming soon**


