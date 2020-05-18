---
layout: post
title: "Load Balancer"
category: system-design
author: lotus
short-description: Tại sao lại là Load Balancer? Thuật toán gì ẩn chứa bên trong? Load Balancer có thể đặt ở layer nào?
---

-----

Load Balancer đóng vai trò rất quan trọng trong thiết kế hệ thống. Vậy:
- Load Balancer là gì? Tại sao các hệ thống lớn cần có Load Balancer?
- Các thuật toán nào được sử dụng cho Load Balancer?
- Load Balancer có thể được đặt ở những layer nào?

### I. Load Balancer là gì? Tại sao các hệ thống cần có Load Balancer?

<div class="text-center">
    <img src="{{ site.baseurl }}/assets/load_balancer/client_server.jpg" alt="client_server"
        title="Client and server" style="width: 40%"/>
</div>


<div class="text-center">
    <img src="{{ site.baseurl }}/assets/load_balancer/client_servers.jpg" alt="client_servers"
        title="Client and servers" style="width: 40%"/>
</div>

**Kết luận:** Load Balancer là một server được sử dụng để cải hiện hiệu suất của dịch vụ (response nhanh hơn, xử lý được nhiều request trên 1 giây hơn) bằng cách chia đều lượng công việc cho nhiều server cùng xử lý. 

Ngoài ra còn Load Balancer còn đem lại những lợi ích sau:
- Scalable: Thêm mới, bớt server trở nên dễ dàng hơn, chỉ cần config cho Load Balancer biết sự tồn tại của server mới. 
- Giảm Downtime: lhi có 1 server trong cụm bị hỏng hay quá tải, Load Balancer sẽ chia khối lượng công việc của server đó cho những server còn lại để đảm bảo hệ thống vẫn hoạt động.


**Các bạn có thắc mắc như ví dụ ở trên, Load Balancer (website khautrangbaba.com) hoàn toàn có thể bị quả tải khi lượng request nhiều lên không?**

<div class="text-center">
    <img src="{{ site.baseurl }}/assets/load_balancer/client_lbs_server.jpg" alt="client_lbs_server"
        title="Client, Load Balancer and servers" style="width: 50%"/>
</div>


Trên thực tế mỗi service không chỉ có 1 Load Balancer. Các Load Balancer 1, 2 sẽ quản lý các Load Balancer ở layer thấp hơn, tương ứng 1.1, 1.2, 2.1, ..., để đảm bảo **High availability** cho hệ thống. Load Balancer 1 và 2 có thể giao tiếp trực tiếp với nhau để phân chia lượng request. 

Load Balancer có thể được đặt ở rất nhiều layer:
- DNS layer.
- Giữa client và server.
- Giữa server và database.
- Giữa chính những Load Balancer khác. 


### II. Các thuật toán được áp dụng cho Load Balancer
**Theo ví dụ ở trên, website khautrangbaba.com của bà Ba đã dùng thuật toán gì để chia đều đơn hàng đến các cơ sở?**

#### 1. Random load balancing
Việc chọn ngẫu nhiên này có thể được thực hiện bởi client hoặc server:

**Cách 1**: Với mỗi request, Load Balancer sẽ chọn ngẫu nhiên 1 server để xử lý request.

**Cách 2**: Server sẽ cho client biết được IP của những server trong cụm. Việc lựa chọn server nào sẽ do client quyết định.

**Ưu điểm:**
- Việc lập trình để xử lý rất đơn giản và không tốn nhiều não.

**Nhược điểm:** 
- Vì là chọn ngẫu nhiên nên 1 server hoàn toàn có thể rơi vào tình trạng quá tải.

#### 2. Round-Robin

Các bạn có thể hiển nó đơn giản qua ví dụ sau:
```
server1 192.168.2.16
server2 192.168.2.17
server3 192.168.2.18
```
Các server 1, 2, 3 sẽ xếp hàng đợi đến lượt của mình nhận request, server nào nhận được request rồi sẽ được chuyển xuống cuối hàng.

**Ưu điểm:**
- Tốn ít não ^^

**Nhược điểm:** 
- Do các server mạnh yếu khác nhau, nên việc sử dụng Round-Robin có thể làm 1 server bị quá tải trong khi server khác lại quá nhàn rỗi. Ví dụ, 192.168.2.16 có thông số Core i9 - RAM 16GB - ổ đĩa SSD - xử lý được 8 reuqest trên 1 giây, trong khi 192.168.2.18 CPU Pentium - RAM 2GB - ổ đĩa HĐD xử lý được 1 request trên 1 giây.
- Việc caching trong phương pháp này không được tối ưu. Giả sử client thực hiện 2 request giống nhau trên cùng 1 máy, thì request thứ nhất sẽ được xử lý bởi server 192.168.2.16, còn request thứ hai được thực hiện trên server 192.168.2.17, với cùng request giống nhau nhưng phải query vào DB 2 lần thực sự lẵng phí.

**Round-Robin DNS**

Phương pháp Round-Robin có thể được thực hiện ngay trên DNS layer. Rất đơn giản, các bạn chỉ cần config cho 1 domain name gắn với nhiều địa chỉ IP.

```
khautrangbaba.com 192.168.2.16
khautrangbaba.com 192.168.2.17
khautrangbaba.com 192.168.2.18
```

Khí request đến, các server sẽ được gọi tuần tự 192.168.2.16, 192.168.2.17, 192.168.2.18, rồi quay vòng về 192.168.2.16 cho request tiếp theo.

**Nhược điểm:** 
- Giống với Round-Robin cơ bản nhưng do việc config nằm ở DNS layer, nên việc cân bằng tải nằm ngoài sự kiểm soát của lập trình viên. Caching trên DNS layer cũng rất khó quản lý.

**Weighted-Round-Robin**
Là một phương pháp cải tiến của Round-Robin. Cho phép đánh trọng số (weight) với mỗi server, những server có trọng số cao hơn sẽ được Load Balancer ưu tiên cho xử lý nhiều request hơn.

#### 3. IP Hash
Dựa vào IP của client, Load Balancer sẽ quyết định server nào tiếp nhận request.
1 ví dụ cách chia client request ứng với server dựa theo cách lấy phần dư

```
hash(client IP) % (tổng số server) = server nhận request 
```

**Ưu điểm:**
- Với cùng 1 client IP, dù request nhiều lần cũng đều do 1 server xử lý, điều này đã tái sử dụng được cache 1 cách hiệu quả hơn.

**Nhược điểm:**
- Khi thêm mới hoặc xóa đi 1 server, vấn đề trở nên phức tạp hơn.

**Consistent Hashing**
1 phương pháp được sinh ra để tối ưu việc tái sử dụng cache. Các bạn có thể xem thêm tại [Consistent Hashing](https://www.youtube.com/watch?v=zaRkONvyGr8) được giải thích bởi anh Gaurav Sen.


### III. Load Balancer được đặt ở những layer nào trong hệ thống?