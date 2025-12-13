# 5.12. Image Filters & Kernel

Tài liệu này giải thích cơ chế cốt lõi của việc trích xuất đặc trưng (feature extraction) trong xử lý ảnh.

## 1. Định nghĩa và Cơ chế Hoạt động

### A. Các thuật ngữ cốt lõi
* **Kernel:** (Còn được gọi là **Convolution Matrix** hoặc **Mask**). Bản chất là một ma trận vuông nhỏ (thường là số lẻ như $3\times3$, $5\times5$...) chứa các hệ số quyết định tính chất xử lý.
* **Filtering & Convolution:**
    * **Filtering (Lọc ảnh):** Là khái niệm chung chỉ quá trình biến đổi ảnh bằng cách sử dụng Kernel.
    * **Convolution (Tích chập):** Là tên gọi kỹ thuật của phép toán trượt Kernel qua từng pixel của ảnh gốc để tính toán giá trị mới. Trong ngữ cảnh xử lý ảnh, hai thuật ngữ này thường gắn liền với nhau: thực hiện Filter chính là thực hiện phép Convolution.

### B. Quy trình hoạt động (Cơ chế trượt - Sliding Mechanism)
Quá trình Convolution diễn ra theo các bước lặp đi lặp lại như sau:
1.  **Alignment (Căn chỉnh):** Đặt tâm của Kernel trùng với pixel đang xét trên ảnh gốc ($P_{x,y}$).
2.  **Element-wise Multiplication (Nhân từng phần tử):** Nhân giá trị của từng ô trong Kernel với giá trị pixel tương ứng nằm đè bên dưới nó.
3.  **Summation (Tính tổng):** Cộng tất cả các kết quả phép nhân lại với nhau.
4.  **Assignment (Gán giá trị):** Lấy kết quả tổng đó gán cho vị trí pixel ($P_{x,y}$) trên ảnh mới (Feature Map).
5.  **Sliding (Trượt):** Dịch chuyển Kernel sang pixel kế tiếp và lặp lại cho đến hết ảnh.

### C. Ví dụ minh họa chi tiết

Xét bài toán tìm cạnh ngang trên ảnh đầu vào $4\times5$ với Kernel $3\times3$.

**1. Dữ liệu đầu vào (Input Image $4\times5$):**
$$
\begin{bmatrix}
50 & 50 & 50 & 100 & 100 \\
150 & 150 & 150 & 80 & 80 \\
250 & 250 & 250 & 200 & 200 \\
100 & 110 & 110 & 110 & 110
\end{bmatrix}
$$

**2. Kernel (Bộ lọc cạnh ngang):**
$$
\begin{bmatrix}
-1 & -1 & -1 \\
0 & 0 & 0 \\
1 & 1 & 1
\end{bmatrix}
$$

**3. Xác định kích thước Output:**
$$Output = (Input - Kernel + 1) = (4-3+1) \times (5-3+1) = \mathbf{2 \times 3}$$
Ma trận kết quả sẽ có 2 hàng và 3 cột. Chúng ta sẽ tính toán giá trị cho từng ô:

#### HÀNG 1 (Row 0)

* **Vị trí (0,0):** Quét cột 0, 1, 2 của hàng 0, 1, 2 Input.
    * Hàng trên: $(50 \times -1) + (50 \times -1) + (50 \times -1) = -150$
    * Hàng giữa: $(150 \times 0) + \dots = 0$
    * Hàng dưới: $(250 \times 1) + (250 \times 1) + (250 \times 1) = 750$
    * **Tổng: $-150 + 0 + 750 = \mathbf{600}$**

* **Vị trí (0,1):** Trượt sang phải 1 ô (Cột 1, 2, 3).
    * Hàng trên: $(50 \times -1) + (50 \times -1) + (100 \times -1) = -200$
    * Hàng giữa: $0$
    * Hàng dưới: $(250 \times 1) + (250 \times 1) + (200 \times 1) = 700$
    * **Tổng: $-200 + 0 + 700 = \mathbf{500}$**

* **Vị trí (0,2):** Trượt sang phải tiếp (Cột 2, 3, 4).
    * Hàng trên: $(50 \times -1) + (100 \times -1) + (100 \times -1) = -250$
    * Hàng giữa: $0$
    * Hàng dưới: $(250 \times 1) + (200 \times 1) + (200 \times 1) = 650$
    * **Tổng: $-250 + 0 + 650 = \mathbf{400}$**

#### HÀNG 2 (Row 1) - Kernel trượt xuống một dòng

* **Vị trí (1,0):** Quét cột 0, 1, 2 của hàng 1, 2, 3 Input.
    * Hàng trên (Input Row 1): $(150 \times -1) + (150 \times -1) + (150 \times -1) = -450$
    * Hàng giữa (Input Row 2): $0$
    * Hàng dưới (Input Row 3): $(100 \times 1) + (110 \times 1) + (110 \times 1) = 320$
    * **Tổng: $-450 + 0 + 320 = \mathbf{-130}$**

* **Vị trí (1,1):** Trượt sang phải (Cột 1, 2, 3).
    * Hàng trên: $(150 \times -1) + (150 \times -1) + (80 \times -1) = -380$
    * Hàng giữa: $0$
    * Hàng dưới: $(110 \times 1) + (110 \times 1) + (110 \times 1) = 330$
    * **Tổng: $-380 + 0 + 330 = \mathbf{-50}$**

* **Vị trí (1,2):** Trượt sang phải (Cột 2, 3, 4).
    * Hàng trên: $(150 \times -1) + (80 \times -1) + (80 \times -1) = -310$
    * Hàng giữa: $0$
    * Hàng dưới: $(110 \times 1) + (110 \times 1) + (110 \times 1) = 330$
    * **Tổng: $-310 + 0 + 330 = \mathbf{20}$**

**4. Kết quả cuối cùng (Feature Map):**
$$
\text{Output} = \begin{bmatrix}
600 & 500 & 400 \\
-130 & -50 & 20
\end{bmatrix}
$$

*Nhận xét:* Hàng đầu tiên có giá trị dương rất lớn (600, 500, 400) cho thấy vùng ảnh phía trên có cạnh ngang rõ rệt (chuyển từ tối sang sáng hoặc ngược lại). Hàng thứ hai giá trị nhỏ (-130, -50, 20) cho thấy cạnh ngang ở vùng dưới yếu hơn hoặc không rõ ràng.

---

## 2. Các loại Kernel phổ biến trong OpenCV

Dưới đây là bảng tổng hợp các "khuôn mẫu" thường dùng nhất:

| Loại Filter | Mục đích | Đặc điểm Kernel | Hàm OpenCV |
| :--- | :--- | :--- | :--- |
| **Average Blur** | Làm mờ, giảm nhiễu đơn giản. | Tất cả hệ số đều bằng nhau ($1/n$). | `cv2.blur()` |
| **Gaussian Blur** | Làm mờ tự nhiên, giữ cạnh tốt hơn. | Hệ số ở tâm lớn nhất, giảm dần ra xa. | `cv2.GaussianBlur()` |
| **Sharpening** | Làm nét ảnh, tăng chi tiết. | Tâm dương rất lớn, xung quanh là số âm. | `cv2.filter2D()` |
| **Sobel (X/Y)** | Phát hiện cạnh (ngang/dọc). | Một bên âm, một bên dương, giữa là 0. | `cv2.Sobel()` |
| **Laplacian** | Phát hiện cạnh theo mọi hướng. | Tâm dương cực lớn, xung quanh âm (hoặc ngược lại). | `cv2.Laplacian()` |

---

## 3. Bản chất toán học: Tại sao phải nhân và cộng tổng?

Hãy hình dung Kernel giống như một **"khuôn mẫu" (template)** của một đặc điểm đang cần tìm kiếm (ví dụ: một cái cạnh ngang, một cái cạnh dọc, hay một chấm tròn).

* **Phép nhân ($Kernel \times Pixel$):** Để kiểm tra xem pixel tại vị trí đó có "ăn khớp" với khuôn mẫu hay không.
* **Phép tổng ($\sum$):** Để gom toàn bộ kết quả so khớp lại thành một con số duy nhất đại diện cho cả vùng đó.

**Ý nghĩa của giá trị kết quả:**
* **Giá trị càng lớn (Big Value):** Nghĩa là vùng ảnh đó **RẤT GIỐNG** với đặc điểm mà Kernel đang tìm kiếm.
* **Giá trị bằng 0 (hoặc gần 0):** Nghĩa là vùng ảnh đó **KHÔNG LIÊN QUAN** gì đến đặc điểm đang tìm, hoặc là một vùng phẳng lì (flat).

> **Ví dụ thực tế:** Quy trình chấm điểm thi trắc nghiệm.
> * **Đáp án đúng (Kernel) là:** A, B, C.
> * **Bài làm 1 (Ảnh):** A, B, C $\rightarrow$ Khớp hoàn toàn $\rightarrow$ **Điểm cao (Giá trị lớn)**.
> * **Bài làm 2 (Ảnh):** D, E, F $\rightarrow$ Không khớp $\rightarrow$ **0 điểm**.

---

## 4. Giải mã Logic Phát hiện Cạnh (Edge Detection)

Ví dụ về **Cạnh Dọc (Vertical Edge)** giúp làm rõ cơ chế này.

**Kernel tìm Cạnh Dọc (Prewitt/Sobel đơn giản):**
Cấu trúc: Bên Trái là số âm, Bên Phải là số dương.
$$
K = \begin{bmatrix}
-1 & 0 & 1 \\
-1 & 0 & 1 \\
-1 & 0 & 1
\end{bmatrix}
$$
*(Logic: Lấy Phải trừ Trái. Nếu khác nhau $\rightarrow$ Có cạnh dọc)*

### Ví dụ 1: Vùng ảnh CÓ cạnh dọc (Trái sáng - Phải tối)
Giả sử vùng ảnh: Cột trái là 200 (Sáng), Cột phải là 0 (Đen).
$$
\text{Image} = \begin{bmatrix}
200 & 100 & 0 \\
200 & 100 & 0 \\
200 & 100 & 0
\end{bmatrix}
$$

**Phép tính Convolution:**
* Cột 1 (Image) nhân Cột 1 (Kernel): $200 \times (-1) + 200 \times (-1) + 200 \times (-1) = \mathbf{-600}$
* Cột 2 (Image) nhân Cột 2 (Kernel): $100 \times 0 + \dots = \mathbf{0}$
* Cột 3 (Image) nhân Cột 3 (Kernel): $0 \times 1 + \dots = \mathbf{0}$

**Tổng:** $-600 + 0 + 0 = \mathbf{-600}$ (Lấy trị tuyệt đối là 600 - Big Value).
$\rightarrow$ **Kết luận:** Máy tính phát hiện **CÓ CẠNH DỌC**.

### Ví dụ 2: Vùng ảnh PHẲNG (Không có cạnh)
Giả sử vùng ảnh toàn màu xám (100).
$$
\text{Image} = \begin{bmatrix}
100 & 100 & 100 \\
100 & 100 & 100 \\
100 & 100 & 100
\end{bmatrix}
$$

**Phép tính Convolution:**
* Cột 1: $100 \times (-1) \times 3 = \mathbf{-300}$
* Cột 2: $100 \times 0 = \mathbf{0}$
* Cột 3: $100 \times 1 \times 3 = \mathbf{300}$

**Tổng:** $-300 + 0 + 300 = \mathbf{0}$.
$\rightarrow$ **Kết luận:** Hai bên triệt tiêu nhau. Máy tính báo **KHÔNG CÓ CẠNH**.

---

## 5. Kết quả đầu ra: Trắng Đen hay Xám?

Quá trình Edge Detection chia làm 2 giai đoạn xử lý riêng biệt:

### Giai đoạn 1: Feature Map (Ảnh Xám) - Output của Kernel
Sau khi thực hiện phép nhân và cộng tổng ở trên, kết quả thu được là một ma trận các con số thực (ví dụ: -600, 0, 450...). Khi hiển thị dữ liệu này lên màn hình, nó sẽ là **Ảnh Xám (Grayscale)**.
* **Ý nghĩa:** Độ sáng của pixel đại diện cho **ĐỘ MẠNH (Magnitude)** của cạnh.
    * Sáng rực (Trắng): Cạnh rất sắc nét, độ tương phản cao.
    * Xám mờ: Cạnh yếu, mờ nhạt.
    * Đen: Không có cạnh (vùng phẳng).

### Giai đoạn 2: Binary Map (Ảnh Nhị Phân) - Sau khi Threshold
Để sử dụng được dữ liệu cho các thuật toán sau đó, máy tính cần ra quyết định dứt khoát: "Cạnh" hay "Không phải cạnh". Bước này áp dụng một **Ngưỡng (Threshold)**, ví dụ là 100.
* Nếu Giá trị > 100 $\rightarrow$ Gán thành 255 (**TRẮNG**).
* Nếu Giá trị < 100 $\rightarrow$ Gán thành 0 (**ĐEN**).
* **Ý nghĩa:** Tạo ra ranh giới rõ ràng để tách vật thể ra khỏi nền.

### Ứng dụng thực tế
1.  **Xe tự lái (Lane Detection):**
    * Dùng Kernel cạnh xiên/dọc để tìm vạch kẻ đường (ra ảnh xám).
    * Dùng Threshold để lọc bỏ mặt đường nhựa, chỉ giữ lại vạch sơn trắng (ra ảnh nhị phân) để xe đi đúng làn.
2.  **Quét mã vạch (Barcode Scanner):**
    * Mã vạch là chuỗi các cạnh dọc đen/trắng liên tiếp. Kernel cạnh dọc giúp máy đọc chính xác khoảng cách giữa các vạch này.
3.  **Y tế (X-Ray):**
    * Dùng Kernel làm nét (Sharpening) để làm nổi bật các vết rạn xương nhỏ (cạnh yếu) mà mắt thường khó thấy trên phim chụp.