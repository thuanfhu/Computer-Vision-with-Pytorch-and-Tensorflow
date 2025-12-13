# 5.6. Resizing, Scaling & Interpolation trong OpenCV

Tài liệu này phân biệt rõ ràng các khái niệm `Resizing`, `Scaling` và đi sâu vào cơ chế hoạt động của các thuật toán `Interpolation` (Nội suy) để tối ưu hóa chất lượng ảnh.

## 1\. Phân biệt Image Resizing và Image Scaling

Mặc dù thường sử dụng chung một hàm `cv2.resize()`, hai khái niệm này có mục đích và hệ quả hình học khác nhau.

### A. Image Resizing (Thay đổi kích thước cố định)

  * **Định nghĩa:** Là hành động thay đổi kích thước cụ thể (`width` x `height`) của ảnh sang một kích thước đích cố định.
  * **Đặc điểm:** Thường **không quan tâm đến tỷ lệ khung hình (Aspect Ratio)**.
  * **Hệ quả:** Nếu tỷ lệ đích khác tỷ lệ gốc (ví dụ: $4:3 \to 16:9$), ảnh sẽ bị **méo (distorted)**, vật thể bị kéo giãn hoặc co lại phi thực tế.
  * **Code OpenCV:**
    ```python
    # Resize cứng về 1280x720, bất chấp ảnh gốc
    resized_img = cv2.resize(img, (1280, 720))
    ```

### B. Image Scaling (Tỷ lệ hóa ảnh)

  * **Định nghĩa:** Là việc thay đổi kích thước ảnh dựa trên một hệ số tỷ lệ (`scale factor`) hoặc thay đổi kích thước nhưng **giữ nguyên Aspect Ratio**.
  * **Đặc điểm:** Duy trì sự cân đối hình học của vật thể trong ảnh.
  * **Hệ quả:** Ảnh to lên hoặc nhỏ đi nhưng không bị méo.
  * **Code OpenCV:** Sử dụng tham số `fx` (trục x) và `fy` (trục y).
    ```python
    # Scale ảnh lên gấp 2 lần (fx=2, fy=2) -> Giữ nguyên Aspect Ratio
    scaled_img = cv2.resize(img, None, fx=2.0, fy=2.0)
    ```

-----

## 2\. Interpolation (Nội suy) là gì?

**Interpolation** không phải là kết quả đầu ra (kích thước), mà là **phương pháp (thuật toán)** để máy tính tính toán giá trị màu sắc của các điểm ảnh (pixel) mới khi lưới pixel thay đổi (do phóng to hoặc thu nhỏ).

### Cơ chế hoạt động chi tiết của các phương pháp

Dưới đây là giải thích chuyên sâu về cách từng thuật toán "suy luận" ra pixel mới:

#### 1\. `cv2.INTER_NEAREST` (Nearest Neighbor)

  * **Cơ chế:** Đơn giản nhất. Với mỗi vị trí pixel mới, thuật toán chỉ cần tìm **1 pixel gốc** có vị trí hình học gần nhất và sao chép giá trị đó.
  * **Vùng tham chiếu:** 1x1 pixel.
  * **Đặc điểm:**
      * Tốc độ: Nhanh nhất.
      * Chất lượng: Rất tệ khi phóng to, gây ra hiện tượng **răng cưa (aliasing)** và vỡ khối (blocky).

#### 2\. `cv2.INTER_LINEAR` (Bilinear Interpolation) - *Mặc định*

  * **Cơ chế:** Tính toán giá trị pixel mới dựa trên trung bình cộng có trọng số (weighted average) của **4 pixel lân cận** (lưới 2x2) theo cả hai chiều x và y. Pixel nào gần điểm cần tính hơn sẽ có trọng số lớn hơn.
  * **Vùng tham chiếu:** 2x2 pixel.
  * **Đặc điểm:**
      * Tốc độ: Nhanh.
      * Chất lượng: Làm mượt ảnh (smoothing), tốt hơn nhiều so với Nearest nhưng có thể làm mờ các cạnh sắc nét.

#### 3\. `cv2.INTER_CUBIC` (Bicubic Interpolation)

  * **Cơ chế:** Sử dụng đa thức bậc 3 (cubic polynomial) để tính toán. Nó xét đến **16 pixel lân cận** (lưới 4x4). Các trọng số không chỉ phụ thuộc vào khoảng cách mà còn mô phỏng đường cong, cho phép "vượt ngưỡng" (overshoot) nhẹ để tạo cảm giác sắc nét hơn.
  * **Vùng tham chiếu:** 4x4 pixel.
  * **Đặc điểm:**
      * Tốc độ: Chậm hơn Linear.
      * Chất lượng: Ảnh sắc nét hơn, giữ chi tiết tốt hơn Linear.

#### 4\. `cv2.INTER_LANCZOS4` (Lanczos Interpolation)

  * **Cơ chế:** Sử dụng hàm Sinc ( $\frac{\sin(x)}{x}$ ) làm cửa sổ nội suy. Nó xem xét một vùng rất rộng là **64 pixel lân cận** (lưới 8x8). Đây là thuật toán phức tạp nhất về mặt toán học trong các tùy chọn cơ bản.
  * **Vùng tham chiếu:** 8x8 pixel (rất rộng).
  * **Đặc điểm:**
      * Tốc độ: **Chậm nhất** (chi phí tính toán cao gấp nhiều lần Linear).
      * Chất lượng: **Độ nét cực cao**. Bảo toàn tốt các chi tiết tần số cao.
      * Nhược điểm phụ: Có thể gây ra hiện tượng "Ringing" (bóng ma/viền mờ) ở các vùng có độ tương phản quá gắt (như chữ đen trên nền trắng).

#### 5\. `cv2.INTER_AREA` (Resampling using Pixel Area Relation)

  * **Cơ chế:** Khác biệt hoàn toàn với các phương pháp trên. Nó tính toán quan hệ diện tích giữa pixel đích và các pixel nguồn. Về cơ bản, nó "trộn" tất cả các pixel nguồn nằm trong vùng của pixel đích lại.
  * **Đặc điểm:**
      * **Chuyên dùng để thu nhỏ (Downscaling):** Giúp loại bỏ hiện tượng nhiễu (moiré) và gợn sóng cực tốt.

-----

## 3\. Bảng so sánh và Chiến lược sử dụng

Để đạt hiệu quả tốt nhất giữa Tốc độ (Performance) và Chất lượng (Quality), hãy áp dụng bảng chiến lược sau:

| Phương pháp | Vùng tham chiếu | Tốc độ | Chất lượng | Trường hợp sử dụng TỐT NHẤT |
| :--- | :--- | :--- | :--- | :--- |
| **Nearest** | 1x1 | Rất nhanh | Kém (Răng cưa) | Debug, Masking, Pixel Art, cần tốc độ tối đa. |
| **Linear** | 2x2 | Nhanh | Khá (Hơi mờ) | **Mặc định**. Stream video, Real-time processing. |
| **Cubic** | 4x4 | Trung bình | Tốt (Sắc nét) | **Phóng to (Upscaling)** thông thường. |
| **Lanczos4** | 8x8 | Rất chậm | **Xuất sắc** | **Phóng to chất lượng cao**, in ấn, xử lý ảnh tĩnh (Offline), Data Augmentation cho AI. |
| **Area** | N/A | Nhanh | Mượt mà | **Thu nhỏ (Downscaling)** ảnh. |

-----

## 4\. Code Mẫu Thực Tế (Best Practices)

```python
import cv2

img = cv2.imread('input_image.jpg')

# --- TRƯỜNG HỢP 1: THU NHỎ ẢNH (DOWNSCALING) ---
# Dùng INTER_AREA để tránh bị nhiễu hạt
downscaled = cv2.resize(img, None, fx=0.5, fy=0.5, interpolation=cv2.INTER_AREA)

# --- TRƯỜNG HỢP 2: PHÓNG TO THỜI GIAN THỰC (REAL-TIME UPSCALING) ---
# Dùng INTER_LINEAR để đảm bảo tốc độ FPS cao
realtime_upscale = cv2.resize(img, None, fx=2.0, fy=2.0, interpolation=cv2.INTER_LINEAR)

# --- TRƯỜNG HỢP 3: PHÓNG TO CHẤT LƯỢNG CAO (HIGH QUALITY UPSCALING) ---
# Dùng INTER_LANCZOS4 để ảnh sắc nét nhất (chấp nhận chậm)
# Thích hợp cho xử lý ảnh y tế, in ấn, hoặc chuẩn bị dữ liệu train AI
high_quality_upscale = cv2.resize(img, None, fx=4.0, fy=4.0, interpolation=cv2.INTER_LANCZOS4)
```