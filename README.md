# 🎯 RFM Customer Segmentation Analysis

> Phân khúc khách hàng dựa trên hành vi mua hàng sử dụng kỹ thuật RFM (Recency, Frequency, Monetary) — Python & Power BI.

---

## 📌 Giới thiệu dự án

Dự án này áp dụng kỹ thuật **RFM Segmentation** trên bộ dữ liệu bán lẻ online tại UK (2009–2011) với hơn **1 triệu giao dịch** từ **5,878 khách hàng** tại **43 quốc gia**.

RFM là kỹ thuật phân tích hành vi khách hàng phổ biến nhất trong ngành Data Analytics và Marketing, giúp doanh nghiệp:
- Xác định nhóm khách hàng giá trị cao
- Phát hiện khách hàng có nguy cơ rời bỏ
- Đề xuất chiến lược marketing phù hợp từng nhóm

---

## 🔢 RFM là gì?

| Chỉ số | Ý nghĩa | Câu hỏi |
|---|---|---|
| **R** — Recency | Gần đây nhất | Khách mua gần đây chưa? |
| **F** — Frequency | Tần suất | Khách mua bao nhiêu lần? |
| **M** — Monetary | Giá trị | Khách chi bao nhiêu tiền? |

---

## 🎯 Phân khúc khách hàng

| Segment | Mô tả | Chiến lược |
|---|---|---|
| 💎 Champions | Mua gần đây, thường xuyên, chi nhiều | Reward, upsell |
| 💙 Loyal Customers | Mua thường xuyên, chi khá | Loyalty program |
| ⚠️ At Risk | Từng là khách tốt, lâu không mua | Win-back campaign |
| 🌱 New Customers | Mới mua lần đầu | Onboarding, nurture |
| 👻 Lost | Lâu không mua, chi ít | Re-engagement |
| 😴 Hibernating | Không hoạt động | Discount offer |
| 🌟 Potential Loyalists | Tiềm năng trở thành loyal | Build relationship |
| 🔮 Promising | Mới nhưng có tiềm năng | Early engagement |

---

## 🛠️ Công cụ & Thư viện

| Công cụ | Mục đích |
|---|---|
| Python 3.10 | Xử lý và phân tích dữ liệu |
| Pandas | Load, clean, transform data |
| NumPy | Tính toán số học |
| Matplotlib | Visualization cơ bản |
| Seaborn | Visualization nâng cao |
| Scikit-learn | Preprocessing |
| Power BI | Interactive Dashboard |
| VS Code + Jupyter | Môi trường phát triển |

---

## 📁 Cấu trúc dự án

```
rfm-segmentation/
├── data/
│   ├── online_retail_II.csv        ← Dataset gốc
│   ├── powerbi_rfm.csv             ← RFM scores
│   ├── powerbi_segments.csv        ← Segment summary
│   ├── powerbi_monthly.csv         ← Monthly revenue
│   └── powerbi_country.csv         ← Country revenue
├── charts/
│   └── rfm_analysis.png            ← EDA charts
├── RFM_Segmentation.ipynb          ← Python notebook
├── RFM_Dashboard.pbix              ← Power BI dashboard
└── README.md
```

---

## 📦 Dataset

- **Nguồn:** [Kaggle — Online Retail II UCI](https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci)
- **Thời gian:** 01/12/2009 – 09/12/2011
- **Quy mô:** 1,067,371 giao dịch
- **Sau khi làm sạch:** 805,549 giao dịch hợp lệ

---

## 🔍 Quy trình phân tích

### Bước 1 — Load & Khám phá
- Load file CSV (~1M dòng)
- Kiểm tra missing values và kiểu dữ liệu

### Bước 2 — Làm sạch dữ liệu
- Xóa dòng không có Customer ID (22.77%)
- Loại bỏ đơn hàng bị hủy (Invoice bắt đầu bằng 'C')
- Lọc Quantity và Price > 0
- Tạo cột TotalPrice = Quantity × Price

### Bước 3 — Tính RFM
- **Recency:** Số ngày kể từ lần mua cuối
- **Frequency:** Số lần mua (unique Invoice)
- **Monetary:** Tổng chi tiêu

### Bước 4 — Chấm điểm (1-5)
- Dùng `pd.qcut()` chia đều thành 5 nhóm
- R: thấp = tốt (mua gần đây) → điểm cao
- F, M: cao = tốt → điểm cao

### Bước 5 — Phân khúc
- Kết hợp R, F, M scores → 8 segments
- Phân tích đặc điểm từng nhóm

### Bước 6 — Visualization & Dashboard
- 4 biểu đồ EDA với Python
- Power BI Dashboard 3 trang

---

## 💡 Insights chính

### 👥 Khách hàng
- Tổng **5,878 khách hàng** từ 43 quốc gia
- **UK chiếm ~85%** tổng doanh thu
- Trung bình mỗi khách mua **6.29 lần**

### 💎 Champions
- Chỉ **1,300 Champions (22%)** tạo ra **68.35% doanh thu**
- Avg Monetary: **9,329 GBP** — gấp 4x Loyal Customers
- Avg Frequency: **17 lần** mua

### ⚠️ Nhóm cần chú ý
- **615 At Risk** — từng chi 2,517 GBP, cần win-back ngay
- **1,275 Lost** — đã rời bỏ hoàn toàn, chi phí tái kích hoạt cao
- **510 Hibernating** — cần discount để kéo lại

### 📈 Doanh thu
- Tổng doanh thu: **17,743,429 GBP**
- **Q4 luôn đạt đỉnh** doanh thu (mùa lễ hội tháng 11-12)
- Tăng trưởng mạnh từ 2010 sang 2011

---

## 📊 Power BI Dashboard

| Trang | Nội dung |
|---|---|
| Overview | KPIs + Monthly Revenue + Top 10 Countries |
| RFM Analysis | Segment Donut + Revenue Bar + Performance Table |
| Customer Details | Scatter Plot + Avg Spending + KPI Cards |

---

## 🚀 Hướng dẫn chạy

```bash
# 1. Clone repo
git clone https://github.com/trantanphat0811/rfm-segmentation.git
cd rfm-segmentation

# 2. Tạo môi trường
python -m venv venv
venv\Scripts\activate  # Windows

# 3. Cài thư viện
pip install pandas numpy matplotlib seaborn scikit-learn jupyter ipykernel openpyxl

# 4. Tải dataset từ Kaggle vào thư mục data/
# https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci

# 5. Chạy notebook
jupyter notebook RFM_Segmentation.ipynb
```

---

## 📋 Business Recommendations

```
💎 Champions (1,300 KH):
   → VIP rewards program
   → Early access to new products
   → Personalized thank-you offers

⚠️ At Risk (615 KH):
   → Win-back email campaign
   → "We miss you" discount 20-30%
   → Survey để hiểu lý do rời bỏ

🌱 New Customers (443 KH):
   → Welcome email series
   → First purchase discount cho lần 2
   → Product recommendation

👻 Lost (1,275 KH):
   → Last-chance offer với discount cao
   → Nếu không phản hồi → remove khỏi list
```

---

## 👤 Tác giả

**Trần Tấn Phát**
- 📧 trantanphat08112004@gmail.com
- 🎓 Sinh viên Công nghệ Dữ liệu — Đại học Văn Lang
- 💼 GitHub: [github.com/trantanphat0811](https://github.com/trantanphat0811)

---

## 📄 License

Dataset thuộc UCI Machine Learning Repository, công bố công khai trên Kaggle.