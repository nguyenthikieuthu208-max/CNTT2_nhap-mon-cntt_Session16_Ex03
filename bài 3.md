# Bài 3: Xử lý lỗi bố cục giao diện hệ thống theo dõi xe



## Phần 1. Phân tích

### Chức năng 1: Định vị phần tử bằng Inspect Element
1. Nhấn **F12** (hoặc chuột phải vào khung lỗi, chọn **Inspect**).
2. Bấm biểu tượng **chọn phần tử** (mũi tên ở góc trên trái DevTools).
3. Rê chuột vào khung "Thông tin tài xế" trên trang, DevTools tô sáng và nhảy đến đúng thẻ trong **tab Elements (DOM Tree)**.
4. Thẻ lỗi (giả định): `<div class="driver-info">`. Làm tương tự với khung bên dưới: `<div class="station-list">`.
5. Xem tab **Styles** (CSS đang áp dụng) và tab **Computed** (giá trị cuối cùng trình duyệt tính ra) để tìm nguyên nhân.

### Chức năng 2: Bốn lớp của CSS Box Model
| Lớp (từ trong ra ngoài) | Vai trò |
|---|---|
| **Content** | Vùng chứa nội dung thật (chữ, hình), có `width` x `height` |
| **Padding** | Khoảng đệm **bên trong** viền, giữa nội dung và border |
| **Border** | Đường viền bao quanh padding |
| **Margin** | Khoảng cách **bên ngoài** viền, dùng để **đẩy các phần tử ra xa nhau** |

Để tách hai khung, lớp quan trọng nhất là **Margin** (khoảng cách giữa hai khung), kết hợp kiểm soát `position` và `height`.

### Sơ đồ Box Model hiện tại của phần tử lỗi (tab Computed)
Phần tử `.driver-info`, thông số giả định:

```text
+---------------------------------------------------+
|  margin                          -40               |
|  +---------------------------------------------+  |
|  |  border                      1                |  |
|  |  +---------------------------------------+  |  |
|  |  |  padding                  10           |  |  |
|  |  |  +-------------------------------+   |  |  |
|  |  |  |   content                     |   |  |  |
|  |  |  |   300 x 180                   |   |  |  |
|  |  |  +-------------------------------+   |  |  |
|  |  +---------------------------------------+  |  |
|  +---------------------------------------------+  |
+---------------------------------------------------+
margin-bottom: -40px  (giá trị âm)
```

| Thuộc tính | Giá trị giả định | Nhận xét |
|---|---|---|
| `width` x `height` (content) | 300 x 180 | Chiều cao cố định |
| `padding` | 10px | Bình thường |
| `border` | 1px | Bình thường |
| `margin-bottom` | **-40px** | **Giá trị âm kéo khung dưới trồi lên, gây chồng lấn** |
| `position` | `relative` | Vẫn nằm trong luồng bố cục |

**Chẩn đoán:** `margin-bottom` âm (và/hoặc chiều cao cố định làm nội dung tràn ra ngoài) khiến khung "Danh sách trạm" bị kéo đè lên khung "Thông tin tài xế".

---

## Phần 2. Triển khai: CSS mới (chỉ sửa trong tab Styles)

### Thao tác
1. Chọn thẻ `.driver-info` trong tab Elements.
2. Trong tab **Styles**, sửa trực tiếp các thuộc tính bên dưới (bấm vào giá trị để chỉnh, tick/bỏ tick để bật tắt thuộc tính).

### Thông số đề xuất
```css
.driver-info {
  margin-bottom: 20px;   /* thay -40px: tạo khoảng cách 20px với khung bên dưới */
  height: auto;          /* thay 180px: nội dung dài bao nhiêu thì khung cao bấy nhiêu */
  box-sizing: border-box;/* width/height đã gồm padding + border, tránh bị phình kích thước */
}

.station-list {
  margin-top: 20px;      /* đảm bảo khoảng cách phía trên */
  position: static;      /* trả về luồng bố cục bình thường nếu đang absolute */
}
```

| Thuộc tính | Cũ | Mới | Tác dụng |
|---|---|---|---|
| `margin-bottom` | -40px | 20px | Hết kéo đè, có khoảng cách giữa hai khung |
| `height` | 180px | auto | Nội dung không tràn ra ngoài khung |
| `box-sizing` | content-box | border-box | Kích thước khung ổn định, dễ tính toán |
| `position` (khung dưới) | absolute | static | Quay về luồng bình thường, xếp nối tiếp nhau |


---

## Phần 3. Kiểm thử: đổi Margin nhưng phần tử vẫn không di chuyển

Cần kiểm tra xem có thuộc tính nào **đang ghi đè hoặc vô hiệu hóa** margin không.

### Các bước kiểm tra
1. Chọn phần tử, mở tab **Styles**: nếu thuộc tính `margin` bị **gạch ngang** nghĩa là đã bị một quy tắc khác đè lên (độ ưu tiên cao hơn).
2. Mở tab **Computed**: xem giá trị `margin` **cuối cùng** trình duyệt dùng. Bấm mũi tên cạnh thuộc tính để biết **quy tắc nào đang quyết định** giá trị đó.
3. Tick **"Show all"** để xem cả các thuộc tính mặc định của trình duyệt.

### Các nguyên nhân thường gặp
| Nguyên nhân | Dấu hiệu | Cách xử lý |
|---|---|---|
| Quy tắc khác đè lên (độ ưu tiên selector cao hơn, hoặc có `!important`) | `margin` bị gạch ngang trong Styles | Dùng selector cụ thể hơn, hoặc kiểm tra quy tắc `!important` |
| `position: absolute` / `fixed` | Phần tử được định vị bằng `top`, `left`, không theo luồng | Sửa `top`/`left`, hoặc đổi sang `position: relative`/`static` |
| `float` chưa được xóa | Phần tử trôi sang một bên | Thêm `clear: both` hoặc `overflow: hidden` ở khung cha |
| Phần tử `display: inline` | Margin trên/dưới không có tác dụng | Đổi sang `display: block` hoặc `inline-block` |
| Margin collapsing (margin dồn lại) | Hai margin dọc liền kề chỉ lấy giá trị lớn hơn | Dùng padding, hoặc `display: flow-root` cho khung cha |
| Khung cha dùng Flex/Grid với `gap` | Khoảng cách do khung cha điều khiển | Chỉnh `gap` ở khung cha |
| Stylesheet mặc định của trình duyệt (User Agent Stylesheet) | Hiện trong Styles với nhãn "user agent stylesheet" | Ghi đè bằng quy tắc của trang web |



