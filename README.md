RMI có thể hiểu là tầng cao hơn tầng giao tiếp socket (tất nhiên, nó cũng phải dùng socket để truyền dữ liệu text đi xa ^^)

`1- Vậy tại sao lại sinh ra RMI:
Như tên gọi, RMI là Remote Method Invocation: gọi một method từ xa. Cái này là yêu cầu hoàn toàn chính đáng, vì cũng giống như con người, app nó phải "giao thông" với nhau, chứ "tự sướng" 1 mình thì sẽ ... không vui nhiều thứ. Ví dụ, bạn viết app thời tiết, ok, giờ bạn làm thế nào lấy được thời tiết Hà Nội, ... chà, tìm 1 cái nhiệt kế, gắn sensor, rồi gửi thông tin về app. ... nhiều thứ phải làm phết nếu "tự sướng" kiểu này. Giờ đơn giản hơn, gọi một method của hãng thứ ba chẳng hạn (Yahoo), ai cũng vui, Yahoo quảng bá service, còn mình thì đỡ phải "phát minh ra cái bánh xe".

`2. Mình thấy RMI phải implement rất nhiều thứ.. dữ liệu cũng cần Relizerable .
Object phải được Serializable, vì network nó chỉ truyền các chuỗi text, chứ không thể truyền object trên RAM của máy được. Tưởng tượng như này, bạn có một con bò, làm sao bạn gửi sang Úc được, nếu chỉ được gửi 1kg một ^^ Vậy là bạn phải serialize (bắt nguồn từ từ "series", từng hàng, nối tiếp nhau), nghĩa là bạn phải tháo chân, cắt đầu, ... (nghe hơi dã man nhỉ, thôi tưởng tượng là con bò lắp ghép nhé). Sang phía bên Úc, muốn có lại được con bò này, bạn lại phải ghép lại (Unserialize). Cũng như việc bạn gửi object qua net thôi, phải serialize nó ra thành text, rồi mới truyền đi được. Quá trình này có thể gọi là marshall / unmarshal.

`3. Mong các Pro giải thích cho mình sao lại cần implement , và cách nó hoạt động
Phía client
Stub (đóng vai trò như 1 gateway)
1- Nó sẽ kết nối với server
2- Ghi và truyền (bằng cách serialize) các thông số (parametter) sang remote JVM
3- Đợi server
4- Đọc chuỗi server trả về và unserialize
5- Trả về obj/value cho app gọi.

Phía server
Skeleton (giờ dùng stub protocol, nhưng về cơ chế cũng như vậy)
1- Đọc params từ client gửi lên > rồi unserialize để lấy obj param.
2- Gọi hàm ra thực thi
3- Ghi và truyền kết quả cho stub (sau khi đã serialize)

`4- Các từ khóa : Registry , lookup ,bind
Registry: danh bạ đăng ký các method được phép gọi, chứ không phải thích gì gọi đấy được
Bind: "gắn" dịch vụ với tên nào đó với 1 skeleton cụ thể
Lookup: "tra cứu" các dịch vụ được cung cấp

Ưu / nhược của RMI
Ưu:
- Nhanh, dễ dùng
Nhược:
- Chỉ dùng được khi server / client cùng ngôn ngữ java (do cách serialize ở java # với PHP, C#)
- Để vượt qua hạn chế này, --> sử dụng webservice. Thế hệ đầu của WS là SOAP cũng cho phép giả lập cách invoke này (cũng gọi function, cũng có object ^^). Tuy nhiên, SOAP # với RMI ở chỗ dữ liệu nhận / gửi là XML documents. Tuy nhiên, nhược điểm của SOAP là nặng, phức tạp (cũng phải khai báo WDSL để client biết cách gọi function này, ...)
- Hiện nay, kỹ thuật WS phổ biến là RESTFUL (sử dụng các verb của HTTP để lưu các state/ function). Qui tất cả các function về 4 loại thôi: GET (lấy về), PUT (đẩy vào dữ liệu), POST (thay đổi dữ liệu) và DELETE (xóa dữ liệu). Function name giờ có thể truyền như 1 biến ^^

Ví dụ:
GET yahoo.api/v2/weather/today -> lấy dữ liệu ngày hôm nay, function là today
GET yahoo.api/v2/weather/weekend -> lấy dữ liệu 3 ngày cuối tuần
PUT yahoo.api/v2/weather/today (cái này thì chắc sensor gọi thôi), và thường đẩy data và body, gọi là payload (tải trọng). Payload có thể là params giống GET (kiểu two_hour=23do&three_hour=24do, nhưng cũng có thể là raw data (23rkjfdsjkdfd34dgd45). Giờ thường dùng dạng raw data (là các text) và dưới dạng JSON format (chứ ít dùng XML. do JSON kích thước nhỏ, dễ hiểu hơn)

Payload:{"today":{"two_hour":"23do", "three_hour":"24do"}} <- thuần text 
