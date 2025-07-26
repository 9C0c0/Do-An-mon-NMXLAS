# Đếm đối tượng - phím O (OBJECT COUNTING)
## Mục đích
Đếm số lượng các vật thể tìm thấy trong khung hình
## Ý tưởng thực hiện
Khi người dùng nhấn phím O, chương trình sẽ hoạt động như sau:
1. Chuyển đổi khung hình sang ảnh xám
2. Làm mờ ảnh để giảm nhiễu
3. Ngưỡng hóa ảnh xám thành ảnh nhị phân (đen/trắng)
4. Đảo ngược ảnh nhị phân
5. Tìm các đường viền trong ảnh thông qua hàm cv2.findContours
6. Khởi tạo biến đếm đối tượng
7. Duyệt qua từng vật thể được tìm thấy
8. Bỏ qua các vật thể quá nhỏ hoặc quá lớn (nhỏ hơn 0.2, lớn hơn 60 % khung hình)
## Công thức đã sử dụng
Tính diện tích vật thể so với diện tích khung hình
```python
percent = 100 * w * h / area
```
> w = chiều rộng | h = chiều cao | area = khung hình
## Ví dụ hoạt động
Khi người dùng nhấn phím O chương trình sẽ chạy hàm count_objects để tìm kiếm các đối tượng trong khung hình
## Code chính
```python
def count_objects(frame):
    global object_count

    frame_gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)    # chuyển đổi khung hình sang ảnh xám
    blurred = cv2.GaussianBlur(frame_gray, (auto_blur, auto_blur), 0) # làm mờ ảnh để giảm nhiễu
    _, thresh = cv2.threshold(blurred, auto_threshold, 255, cv2.THRESH_BINARY) # ngưỡng hóa ảnh xám thành ảnh nhị phân (đen/trắng)
    thresh = ~thresh  # đảo ngược ảnh nhị phân 
    
    contours, _ = cv2.findContours(thresh, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE) # tìm các đường viền trong ảnh
    object_count = 0  # khởi tạo biến đếm đối tượng

    for c in contours:  # duyệt qua từng vật thể được tìm thấy
        x, y, w, h = cv2.boundingRect(c) #tính hcn bao quanh vật thể
        percent = 100 * w * h / area #tính diện tích vật thể so với diện tích khung hình

        # bỏ qua các vật thể quá nhỏ hoặc quá lớn
        if percent < auto_percent or percent > 60:
            continue

        object_count += 1
        cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 255), 2)
        cv2.putText(frame, f'{object_count}', (x, y - 10), 
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 255), 2)

    return frame, object_count
```