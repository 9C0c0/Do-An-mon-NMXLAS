# LƯU LỊCH SỬ ĐO LƯỜNG (HISTORY SAVE)

## Mục đích

Lưu lại lịch sử các thao tác hoặc kết quả xử lý (ví dụ: số lượng đối tượng đếm được, thời gian thao tác, trạng thái khung hình) để người dùng có thể xem lại hoặc xuất ra file.

## Thư viện đã sử dụng

```py
import json #lưu lịch sử đo dưới dạng .json
import csv # lưu lịch sử đo dưới dạng .csv
from datetime import datetime # xác định thời gian thực hiện phép đo
import os # kiểm tra và tạo thư mục nếu chưa tồn tại
```

## Ý tưởng thực hiện

Khi người dùng nhấn phím H, chương trình sẽ hoạt động như sau:

1. Tạo lớp `MeasurementHistory` để quản lý lịch sử đo lường.
2. Lịch sử sẽ được lưu trong file `camruler_history.json`.
3. Mỗi lần đo, một đối tượng `dict` được tạo chứa các thông tin:

- Loại đo (`auto`, `manual`)
- Kích thước đo được (chiều dài x, y, đường chéo, diện tích)
- Thời gian đo
- Ghi chú (nếu cần)

4. Có thể xuất lịch sử ra định dạng `.csv` để dễ phân tích.
5. Có thể xóa toàn bộ lịch sử đo lường khi nhấn phím `e`.

## Công thức đã sử dụng

- Chiều dài cạnh:

```py
llen = hypot(xlen, ylen)
```

- Chiều dài trung bình (nếu hình vuông hoặc gần vuông):

```py
avg_len = (x_length + y_length) / 2
```

- Lưu thời gian hiện tại:

```py
from datetime import datetime
now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
```

## Ví dụ hoạt động

Sau mỗi lần đếm đối tượng và đo vật thể, khi nhấn phím H, chương trình sẽ lưu lại số lượng đối tượng và thời gian vào danh sách `history`. Khi cần, có thể xuất danh sách này ra file CSV.

Sau khi thực hiện phép đo, kết quả sẽ được lưu như sau:

```json
{
  "timestamp": "2025-07-25 17:45:21",
  "type": "manual",
  "x_length": 32.5,
  "y_length": 48.9,
  "diagonal_length": 58.3,
  "area": 1602.7,
  "avg_len": null,
  "unit": 100,
  "auto_mode": false
}
```

Bạn có thể nhấn `h` để xem lịch sử đo trên giao diện và nhấn `e` để xuất toàn bộ dữ liệu đo thành file `camruler_export.csv`

Bạn cũng có thể xóa toàn bộ lịch sử đo lường bằng cách nhấn `e` để xóa.

## Code chính

```py
class MeasurementHistory:
    def __init__(self, filename=history_file, max_entries=max_history_entries):
        self.filename = filename
        self.max_entries = max_entries
        self.measurements = []
        self.load_history()

    def load_history(self):
        """tải lịch sử đo lường từ file"""
        if os.path.exists(self.filename):
            try:
                with open(self.filename, 'r', encoding='utf-8') as f:
                    self.measurements = json.load(f)
            except:
                self.measurements = []

    def save_history(self):
        """Lưu lịch sử đo lường vào file"""
        try:
            with open(self.filename, 'w', encoding='utf-8') as f:
                json.dump(self.measurements, f, indent=2, ensure_ascii=False)
        except Exception as e:
            print(f"Error saving history: {e}")

    def add_measurement(self, measurement_type, x_len, y_len, l_len, area, avg_len=None, auto_mode=False):
        """Thêm một đo lường mới vào lịch sử"""
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        measurement = {
            'timestamp': timestamp,
            'type': measurement_type, # 'tự động' hoặc 'thủ công'
            'x_length': round(x_len, 2), # chiều dài trục x
            'y_length': round(y_len, 2), # chiều dài trục y
            'diagonal_length': round(l_len, 2), # chiều dài đường chéo
            'area': round(area, 2), # diện tích hình chữ nhật
            'unit': unit_suffix, # đơn vị đo lường (vd: mm)
            'auto_mode': auto_mode
        }
    def clear_history(self):
        """Xóa tất cả lịch sử đo lường"""
        self.measurements = []
        self.save_history()
        print("HISTORY: cleared all measurements")

    def export_to_csv(self, filename=None):
        """Xuất lịch sử vào file CSV"""
        if not filename:
            filename = f"camruler_export_{datetime.now().strftime('%Y%m%d_%H%M%S')}.csv"

        try:
            with open(filename, 'w', encoding='utf-8') as f:
                # Write header
                f.write("Timestamp,Type,X_Length,Y_Length,Diagonal_Length,Area,Average_Length,Unit,Auto_Mode\n")
                # Write data
                for m in self.measurements:
                    avg_len = m.get('average_length', '')
                    f.write(f"{m['timestamp']},{m['type']},{m['x_length']},{m['y_length']},{m['diagonal_length']},{m['area']},{avg_len},{m['unit']},{m['auto_mode']}\n")

            print(f"HISTORY: Exported to {filename}")
            return True
        except Exception as e:
            print(f"Error exporting history: {e}")
            return False
```
