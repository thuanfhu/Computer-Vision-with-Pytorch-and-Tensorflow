# 5.13. Applying Blur filters: Average, Gaussian, Median

Trong xử lý ảnh, **Blurring (Làm mờ)** hay **Smoothing** là kỹ thuật được sử dụng để giảm nhiễu (noise), loại bỏ các chi tiết thừa hoặc làm mịn ảnh. Bản chất của nó là sử dụng một cửa sổ trượt (Kernel) để tính toán lại giá trị pixel dựa trên các pixel lân cận.

## 1\. Dữ liệu mẫu dùng để tính toán (Toán học)

Để so sánh 3 phương pháp, ta xét một ma trận ảnh **$7 \times 7$** có nền màu tối (giá trị **10**) và bị dính một điểm **nhiễu muối tiêu (cực sáng)** ở tâm (giá trị **200**).

**Input Image ($7 \times 7$):**

$$
\begin{bmatrix}
10 & 10 & 10 & 10 & 10 & 10 & 10 \\
10 & 10 & 10 & 10 & 10 & 10 & 10 \\
10 & 10 & 10 & 10 & 10 & 10 & 10 \\
10 & 10 & 10 & \mathbf{200} & 10 & 10 & 10 \\
10 & 10 & 10 & 10 & 10 & 10 & 10 \\
10 & 10 & 10 & 10 & 10 & 10 & 10 \\
10 & 10 & 10 & 10 & 10 & 10 & 10
\end{bmatrix}
$$

Chúng ta sẽ tính toán cho vùng **$3 \times 3$ tại tâm** (nơi chứa điểm nhiễu 200) và một vùng **$3 \times 3$ ở góc** (nơi chỉ toàn số 10).

---

## 2\. Average Filtering (Làm mờ trung bình)

### A. Bản chất & Cơ chế

-   **Loại:** Bộ lọc Tuyến tính (Linear Filter).
-   **Cơ chế:** Tính trung bình cộng của tất cả các pixel nằm trong Kernel. Mọi pixel có vai trò ngang nhau.
-   **Kernel chuẩn ($3 \times 3$):**
    $$
    K = \frac{1}{9} \begin{bmatrix}
    1 & 1 & 1 \\
    1 & 1 & 1 \\
    1 & 1 & 1
    \end{bmatrix}
    $$

### B. Tính toán minh họa

**Trường hợp 1: Tại vùng nhiễu (Tâm 200)**

-   Tổng giá trị: $(10 \times 8 \text{ ô nền}) + 200 = 80 + 200 = 280$.
-   Kết quả: $280 / 9 \approx \mathbf{31}$.
    -   _Nhận xét:_ Nhiễu giảm từ 200 xuống 31, nhưng bị "loang" ra xung quanh.

**Trường hợp 2: Tại vùng nền (Toàn số 10)**

-   Tổng giá trị: $10 + 10 + \dots + 10 = 10 \times 9 = 90$.
-   Kết quả: $90 / 9 = \mathbf{10}$.
    -   _Giải thích:_ Vì tất cả pixel đều bằng nhau, trung bình cộng của chúng chính là giá trị đó. Ảnh nền không bị thay đổi.

### C. Code OpenCV & Ứng dụng

-   **Hàm:** `cv2.blur(src, ksize)`
-   **Ưu điểm:** Nhanh, đơn giản.
-   **Nhược điểm:** Làm nhòe ảnh rất mạnh, mất chi tiết cạnh.

<!-- end list -->

```python
# Làm mờ với kernel 5x5
avg_blur = cv2.blur(image, (5, 5))
```

---

## 3\. Gaussian Filtering (Làm mờ Gauss)

### A. Bản chất & Cơ chế

-   **Loại:** Bộ lọc Tuyến tính (Linear Filter).
-   **Cơ chế:** Tính trung bình cộng có trọng số (Weighted Average). Pixel ở gần tâm có trọng số cao, xa tâm trọng số thấp (theo hình quả chuông).
-   **Kernel xấp xỉ ($3 \times 3$):**
    $$
    K = \frac{1}{16} \begin{bmatrix}
    1 & 2 & 1 \\
    2 & \mathbf{4} & 2 \\
    1 & 2 & 1
    \end{bmatrix}
    $$

### B. Tính toán minh họa

**Trường hợp 1: Tại vùng nhiễu (Tâm 200)**

-   Tâm (nhân 4): $200 \times 4 = 800$.
-   Xung quanh (nhân 1 hoặc 2): Tổng trọng số xung quanh là 12. Giá trị là $10 \times 12 = 120$.
-   Tổng: $800 + 120 = 920$.
-   Kết quả: $920 / 16 = 57.5 \approx \mathbf{58}$.
    -   _Nhận xét:_ Giá trị cao hơn Average (58 \> 31) vì nó ưu tiên giữ lại giá trị tâm. Ảnh mờ tự nhiên hơn.

**Trường hợp 2: Tại vùng nền (Toàn số 10)**

-   Tính toán: $(10 \times 1) + (10 \times 2) + \dots + (10 \times 4) + \dots = 10 \times (\text{Tổng trọng số})$.
-   Tổng trọng số của Kernel là 16.
-   Kết quả: $(10 \times 16) / 16 = \mathbf{10}$.
    -   _Giải thích:_ Tương tự như Average, trên một vùng phẳng màu, Gaussian không làm thay đổi giá trị màu.

### C. Code OpenCV & Ứng dụng

-   **Hàm:** `cv2.GaussianBlur(src, ksize, sigmaX)`
-   **Ứu điểm:** Mờ tự nhiên, giữ cạnh tốt hơn Average.
-   **Trường hợp dùng tốt nhất:** Giảm nhiễu Gaussian (nhiễu hạt mịn), làm mịn da, tiền xử lý cho hầu hết các bài toán AI.

<!-- end list -->

```python
# Kernel 5x5, sigma=0 (tự tính toán)
gauss_blur = cv2.GaussianBlur(image, (5, 5), 0)
```

---

## 4\. Median Filtering (Làm mờ Trung vị)

### A. Bản chất & Cơ chế

-   **Loại:** Bộ lọc Phi tuyến tính (Non-linear Filter).
-   **Cơ chế:** KHÔNG dùng phép cộng nhân. Gom tất cả pixel trong cửa sổ, **SẮP XẾP (Sort)** từ nhỏ đến lớn, và chọn số nằm chính giữa (**Median**).

### B. Tính toán minh họa

**Trường hợp 1: Tại vùng nhiễu (Tâm 200)**

-   Danh sách pixel trong vùng $3 \times 3$: `[10, 10, 10, 10, 200, 10, 10, 10, 10]`.
-   Sắp xếp: `[10, 10, 10, 10,` **`10`** `, 10, 10, 10, 200]`.
-   Kết quả (Số ở giữa): **10**.
    -   _Nhận xét:_ **Nhiễu biến mất hoàn toàn\!** Giá trị quay về đúng 10 như nền. Đây là sức mạnh tuyệt đối của Median.

**Trường hợp 2: Tại vùng nền (Toàn số 10)**

-   Danh sách: `[10, 10, 10, 10, 10, 10, 10, 10, 10]`.
-   Sắp xếp: Vẫn y nguyên.
-   Kết quả: **10**.

### C. Code OpenCV & Ứng dụng

-   **Hàm:** `cv2.medianBlur(src, ksize)` (ksize phải là số lẻ).
-   **Ưu điểm:** Loại bỏ hoàn toàn nhiễu muối tiêu mà không làm mờ cạnh của vật thể lớn.
-   **Trường hợp dùng tốt nhất:** Ảnh bị nhiễu lấm tấm đen trắng, ảnh scan tài liệu cũ.

<!-- end list -->

```python
# ksize là số nguyên (ví dụ 5), tương đương cửa sổ 5x5
median_blur = cv2.medianBlur(image, 5)
```

---

## 5\. Tổng kết so sánh

| Đặc điểm            | Average Filter                        | Gaussian Filter                         | Median Filter                        |
| :------------------ | :------------------------------------ | :-------------------------------------- | :----------------------------------- |
| **Bản chất toán**   | Trung bình cộng                       | Trung bình có trọng số                  | Sắp xếp & Chọn số giữa               |
| **Xử lý nhiễu 200** | $\downarrow$ 31 (Nhiễu bị loang rộng) | $\downarrow$ 58 (Nhiễu dịu đi, mềm mại) | **$\downarrow$ 10 (Nhiễu biến mất)** |
| **Độ giữ nét**      | Kém nhất (Mờ đều)                     | Trung bình (Mờ tự nhiên)                | Tốt nhất (Giữ cạnh sắc nét)          |
| **Dùng khi nào?**   | Ít dùng, cần tốc độ cực cao.          | **Mặc định** cho hầu hết tác vụ.        | Chuyên trị **Nhiễu muối tiêu**.      |

### Tại sao vùng toàn số 10 thì kết quả vẫn là 10?

Dù bạn dùng phương pháp nào:

1.  **Average:** $(10+10+...+10)/9 = 90/9 = 10$.
2.  **Gaussian:** Tổng trọng số luôn được chuẩn hóa về 1. $(10 \times \text{Weight}_1 + 10 \times \text{Weight}_2...) = 10 \times (\sum \text{Weights}) = 10 \times 1 = 10$.
3.  **Median:** Sắp xếp một dãy toàn số 10 thì số ở giữa chắc chắn là 10.

$\rightarrow$ **Kết luận:** Các bộ lọc làm mờ (Blur) chỉ thay đổi giá trị ở những nơi **có sự chênh lệch màu sắc** (như nhiễu hoặc cạnh vật thể). Ở những vùng phẳng (flat region) đồng màu, chúng không làm thay đổi ảnh.
