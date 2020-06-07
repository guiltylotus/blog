---
layout: post
title: "Load Balancer"
category: system-design
author: lotus
short-description: Load Balancer có thể đặt ở những layer nào?
---

-----



Load Balancer đóng vai trò rất quan trọng trong thiết kế hệ thống. Vậy:
- Load Balancer là gì? Tại sao các hệ thống lớn cần có Load Balancer?
- Các thuật toán nào được sử dụng cho Load Balancer?
- Load Balancer có thể được đặt ở những layer nào?
- Nhược điểm 

# I. Load Balancer?
## Load Balancer được sinh ra để làm gì?

Trong mô hình Client-server đơn giản, chỉ có 1 server phục vụ nhiều clients cùng lúc. Việc server đó xử lý được bao 
clients trong 1 giây phụ thuộc vào tài nguyên của server đó (network, CPU, Memory, I/O). Khi dịch vụ của bạn trở nên phổ biến hơn, lượng người dùng càng ngày càng nhiều lên, 1 client có thể sẽ mất đến cả phút để đợi được phục vụ. Việc này rất dễ khiến những khách hàng của bạn thất vọng và bỏ đi. Để có thể phục vụ nhiều khách hàng hơn, bạn cần nhiều server hơn. 

Việc thêm nhiều servers vào hệ thống vẫn chưa đủ (Có thể khái báo thêm server bằng cách gán 1 domain với nhiều địa chỉ IP ở layer DNS), vì các server có nguồn tài nguyên khác nhau, vẫn có thể xảy ra tình trạng 1 server phải xử lý quá nhiều requests trong khi server còn lại quá rảnh rỗi. Tình trạng 1 server bị quá tải, khiến những khách hàng phaỉ chờ đợi quá lâu rồi bỏ đi vẫn không nằm trong tầm kiểm soát của bạn. 

Do đó, Load Balancer được sinh ra để kiểm soát và phân chia lượng requests đều vào các servers hiện có để sử dụng tối ưu lượng tài nguyên của servers.

## Những lợi ích mà Load Balancer mang lại

- *Efficient Resources Management* : Đây cũng chính là lý do Load Balancer được tạo ra. Bằng việc giao tiếp với những server trong cụm, Load Balancer biết được những server nào đang bận rộn và những server nào đang rảnh rỗi hơn để phân chia requests hợp lý.
- *Hidden Server Maintenance* : Việc nâng cấp version hoàn  toàn có thể được thực hiện mà không ảnh hưởng đến trải nghiệm người dùng. 1 server thực thiện nâng cấp sẽ được tách ra khỏi cụm trong khi những server còn lại tiếp tục xử lý requests. Sau khi nâng cấp version thành công, server đó sẽ được cho vào cụm, và đến lượt server khác thực hiện nâng cấp. 
- *Automated Scaling*: Việc thêm mới hay gỡ bỏ 1 server ra khỏi cụm cũng trở nên dễ dàng hơn mà không ảnh hưởng đến trải nghiệm người dùng.
- *Efficient failure Management*: Khi 1 server bị lỗi, Load balancer sẽ tách server lỗi ra khỏi cụm để đảm bảo những requests mới đến không bị ảnh hưởng. Server đó thực hiện recovery và sẽ được Load Balancer cho vào cụm khi hết lỗi

# II. Các thuật toán được áp dụng cho Load Balancer
## 1. Random Load Balancing

Chọn ngẫu nhiên một server để xử lý request. Việc này có thể được thực hiện bởi client hoặc server
- **Server-side**: Với mỗi request, Load Balancer sẽ chọn ngẫu nhiên 1 server để xử lý request.
- **Client-side**: Server sẽ cho client biết được IP của những server trong cụm. Việc lựa chọn server nào sẽ do client quyết định.

**Nhược điể**
- Vì là chọn ngẫu nhiên nên 1 server hoàn toàn có thể rơi vào tình trạng quá tải.

## 2. Round-Robin

Các server sẽ được cho vào queue và đợi đến lượt mình nhần request, server nào đã được nhận request sẽ được xếp xuống cuối queue

```
request-1 -> server-1
request-2 -> server-2
request-3 -> server-3
request-4 -> server-1

```

**Nhược điểm**
- Do các server có tài nguyên khác nhau, nên việc sử dụng Round-Robin có thể làm 1 server bị quá tải trong khi server khác lại quá nhàn rỗi. Ví dụ, server-1 có thông số Core i9 - RAM 16GB - ổ đĩa SSD - xử lý được 8 reuqest trên 1 giây, trong khi server-2 CPU Pentium - RAM 2GB - ổ đĩa HĐD xử lý được 1 request trên 1 giây.
- Việc caching trong phương pháp này không được tối ưu. Giả sử client thực hiện 2 request giống nhau trên cùng 1 máy, thì request thứ nhất sẽ được xử lý bởi server-1, còn request thứ hai được thực hiện trên server-2, với cùng request giống nhau nhưng phải query vào DB 2 lần thực sự lẵng phí.

**Weight Round-Robin**

Một phương pháp cải tiến của Round-Robin, được thực hiện trên Load Balancer. Cho phép đánh trọng số (weight) với mỗi server, những server có trọng số cao hơn sẽ được Load Balancer ưu tiên cho xử lý nhiều request hơn. 

Điều này khắc phục được nhược điểm về sự khác nhau về tài nguyên của thuật toán Round-Robin

## 3. Least Recent Connection 

Load Balancer sẽ chọn ra server hiện đang có ít kết nối đến nó nhất trong hệ thống. 

**Nhược điểm**
- [Least Connections is Not Least Loaded](https://devcentral.f5.com/s/articles/back-to-basics-least-connections-is-not-least-loaded). Thời gian để xử lý requests là hoàn toàn khác nhau. Những requests yêu cầu 1 lượng lớn data hay những tính toán phức tạp sẽ cần nhiều thời gian để xử lý hơn. 

Ví dụ: Server-1 xử lý được 10 requests đơn giản trong 1 giây. Trong khi server-2 đang phải xử lý 9 requests phức tạp tốn 20 giây. Khi này, Load Balancer nhận được 1 request đòi tính toán phức tạp, lượng connections của Server-1 nhiều hơn Server-2 nên Load Balancer sẽ ưu tiên chọn Server-2, khiến cho Server-2 đã quá tải lại càng thêm quá tải ^^ 

- Tương tự như round-robin, thuật toán này không phân biệt được server nào có nhiều tài nguyên hơn, xử lý được nhiều request hơn. Do đó chúng ta hoàn toàn có thể áp dụng *Weight Least Connections* để giải quyết vấn đề này.

## 4. Least Response Time 

Load Balancer sẽ sử dụng *health check* để tính toán response time của các server. Response time được tính bằng khoảng thời gian Load Balancer bắt đầu gửi request đến khi nhận được packet đầu tiên (còn được gọi là TTFB - Time to First Byte)

Để chọn ra server nào nhận request tiếp theo, Load Balancer sẽ tính với công thức:

```
*số connections hiện tại* x TTFB  
```

và chuyển request cho server có kết quả nhỏ nhất.

**Nhược điểm**
- Nhược điểm của phương pháp cũng như Round-Robin không biết được server nào xử lý được nhiều requests hơn. Nến cũng có thể áp dụng *Weight Least Response Time* ^^
- Sau 1 khoảng thời gian, server phải xử lý thêm request *health check* từ Load Balancer.

## 5. IP-based Hashing

Load Balancer sẽ dựa vào IP của client để tạo ra 1 mã hash. Mã hash này sẽ quyết định server nào tiếp nhận và xử lý request. Với cùng 1 client IP, dù request nhiều lần cũng đều do 1 server xử lý, điều này đã tái sử dụng được cache 1 cách hiệu quả.

**Nhược điểm**
- Khi 1 server bị mất kết nối hay hỏng hóc, Load Balancer sẽ ngắt kết nối với server đó và phải invalid toàn bộ cặp mapping đang map tới server lỗi.
- Việc thêm mới 1 server vào cụm sẽ làm code trở nên phức tạp hơn khi phải thêm những giải thuật để Load Balancer mapping lượng request đề vào các servers sao cho hiệu quả.

**Consistent Hashing**
1 phương pháp được sinh ra để tối ưu việc tái sử dụng cache. Các bạn có thể xem thêm tại [Consistent Hashing](https://www.youtube.com/watch?v=zaRkONvyGr8) được giải thích bởi anh Gaurav Sen.


# III. Load Balancer levels
## 1. DNS layer
Vào cái thời mà Load Balancer chưa phổ biến, người ta thường sử dụng DNS để phân chia requests. Chỉ cần config cho 1 domain name mapping với nhiều địa chỉ IP.

```
example.com 1.1.1.1
example.com 2.2.2.2
```
DNS sẽ thực hiện chia requests vào các server theo thuật toán Round-Robin

## 2. Ở giữa Client và Server
## 3. Ở giữa Server và Database

# IV. Nhược điểm

## Single point of failure