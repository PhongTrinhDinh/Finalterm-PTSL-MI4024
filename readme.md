# Phân tích số liệu

Xây dựng mô hình hồi quy dự báo tuổi thọ (Life Expectancy) dựa trên các yếu tố kinh tế - xã hội và y tế.

## 📌 Tổng quan dữ liệu
*   **Nguồn dữ liệu:** Life Expectancy Data (WHO).
*   **Kích thước:** 2938 dòng, 22 cột.
*   **Biến mục tiêu (Target):** `Life expectancy`.
*   **Đặc điểm:** Dữ liệu chuỗi thời gian (Time-series) theo quốc gia, chứa nhiều giá trị thiếu (missing values) và giá trị ngoại lai (outliers).

## ⚙️ Quy trình thực hiện (Methodology)

### 1. Tiền xử lý dữ liệu (Data Preprocessing)
*   **Xử lý dữ liệu thiếu (Missing Values):** Áp dụng chiến lược đa tầng (Multi-stage Imputation):
    1.  Sắp xếp dữ liệu theo `Country` và `Year`.
    2.  **Nội suy tuyến tính (Linear Interpolation):** Điền khuyết dựa trên xu hướng thời gian của từng quốc gia.
    3.  **Forward/Backward Fill:** Xử lý các điểm dữ liệu thiếu ở đầu/cuối chuỗi thời gian.
    4.  **Median by Group:** Điền các giá trị còn lại bằng trung vị theo nhóm tình trạng phát triển (`Status`).
*   **Xử lý ngoại lai (Outliers):** Sử dụng phương pháp **Winsorizing** (giới hạn phân vị 5% - 95%) cho các biến như `HIV/AIDS`, `Adult Mortality`, `Measles`... để giảm ảnh hưởng của các giá trị cực đoan.
*   **Xử lý lỗi dữ liệu:** Loại bỏ hoặc sửa các giá trị phi lý (ví dụ: BMI > 100, Infant deaths sai lệch).

### 2. Lựa chọn và Biến đổi đặc trưng (Feature Engineering)
*   **Kiểm tra đa cộng tuyến:** Loại bỏ các biến có tương quan quá cao với nhau hoặc ít tương quan với biến mục tiêu (ví dụ: `infant deaths`, `percentage expenditure`, `Population`...).
*   **Chuẩn hóa phân phối:** Áp dụng biến đổi **Log (np.log1p)** cho các biến có phân phối lệch phải (Right-skewed) như `Adult Mortality`, `GDP`, `HIV/AIDS`.
*   **Scaling:** Sử dụng `StandardScaler` để đưa dữ liệu về cùng một tỷ lệ.
*   **Lựa chọn biến (Feature Selection):** Sử dụng thuật toán **Stepwise Selection** (kết hợp Forward Selection và Backward Elimination) dựa trên p-value để chọn ra tập biến tối ưu nhất.

### 3. Xây dựng Mô hình (Modeling)
*   **Mô hình cơ sở (OLS):** Bắt đầu với Hồi quy tuyến tính bình thường (Ordinary Least Squares).
    *   *Vấn đề phát hiện:* Có hiện tượng phương sai sai số thay đổi (**Heteroscedasticity**) khi kiểm tra phần dư.
*   **Mô hình cải tiến (WLS):** Chuyển sang Hồi quy bình phương tối thiểu có trọng số (**Weighted Least Squares**) để khắc phục hiện tượng phương sai thay đổi.
*   **Tinh chỉnh:** Sử dụng **Khoảng cách Cook (Cook's Distance)** để phát hiện và loại bỏ các điểm dữ liệu gây ảnh hưởng xấu (influential points) đến mô hình.

## 📊 Kết quả chính (Results)

Mô hình cuối cùng (WLS sau khi loại bỏ nhiễu) được đánh giá trên tập kiểm thử (Test set) cho kết quả rất khả quan:

*   **R-squared (R2):** `0.8502` (Mô hình giải thích được khoảng 85% sự biến thiên của dữ liệu).
*   **Mean Absolute Error (MAE):** `2.88` năm.
*   **Root Mean Squared Error (RMSE):** `3.74` năm.

### Khả năng dự báo
Mô hình có khả năng đưa ra dự báo tuổi thọ cụ thể kèm theo **khoảng tin cậy 95%** (95% Confidence Interval).

*Ví dụ thực tế từ model:*
> Với một mẫu dữ liệu đầu vào, mô hình dự đoán tuổi thọ là **69.9 tuổi**.
> Khoảng tin cậy 95%: Tuổi thọ thực tế sẽ nằm trong khoảng **(68.85, 70.97)**.

## 🛠 Công cụ sử dụng
*   **Ngôn ngữ:** Python
*   **Thư viện:** Pandas, NumPy, Matplotlib, Seaborn, Scikit-learn, Statsmodels.