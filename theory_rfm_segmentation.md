# 📚 Theory — RFM Customer Segmentation

> Tài liệu lý thuyết nền tảng cho dự án phân khúc khách hàng theo mô hình RFM.

---

## 1. RFM Analysis là gì?

**RFM** là kỹ thuật phân tích hành vi khách hàng dựa trên 3 chỉ số:

| Chữ cái | Tiếng Anh | Tiếng Việt | Câu hỏi |
|---|---|---|---|
| **R** | Recency | Tính gần đây | Khách hàng mua lần cuối cách đây bao lâu? |
| **F** | Frequency | Tần suất | Khách hàng đã mua bao nhiêu lần? |
| **M** | Monetary | Giá trị tiền tệ | Khách hàng đã chi tổng bao nhiêu tiền? |

### Nguyên tắc cơ bản:
```
R thấp  → Mua gần đây → Khách hàng ĐANG HOẠT ĐỘNG
F cao   → Mua nhiều lần → Khách hàng TRUNG THÀNH
M cao   → Chi nhiều tiền → Khách hàng GIÁ TRỊ CAO
```

---

## 2. Tại sao RFM quan trọng?

### 2.1 Nguyên tắc Pareto (80/20)
Trong kinh doanh, thường có hiện tượng:
> **20% khách hàng tạo ra 80% doanh thu**

Trong dự án của chúng ta:
> **22% Champions tạo ra 68% doanh thu**

→ Tập trung giữ chân nhóm này có tác động lớn hơn nhiều so với tìm khách hàng mới.

### 2.2 Chi phí giữ chân vs. Thu hút mới
```
Chi phí thu hút khách hàng mới = 5-7 lần
chi phí giữ chân khách hàng cũ
```

→ RFM giúp xác định đúng nhóm cần đầu tư nguồn lực.

### 2.3 Cá nhân hóa Marketing
Thay vì gửi cùng 1 email cho tất cả:
- Champions → Email VIP + early access
- At Risk → "Chúng tôi nhớ bạn" + discount 20%
- Lost → Last-chance offer hoặc bỏ qua

---

## 3. Cách tính điểm RFM

### 3.1 Xác định ngày tham chiếu (Reference Date)
```python
# Thường lấy ngày giao dịch cuối cùng + 1 ngày
reference_date = df['InvoiceDate'].max() + timedelta(days=1)
```

### 3.2 Tính R, F, M cho từng khách hàng
```python
rfm = df.groupby('Customer ID').agg(
    Recency   = ('InvoiceDate', lambda x: (reference_date - x.max()).days),
    Frequency = ('Invoice',     'nunique'),   # Số hóa đơn duy nhất
    Monetary  = ('TotalPrice',  'sum')
)
```

### 3.3 Chấm điểm 1-5 (Quantile-based Scoring)

**Quantile** là gì? Chia dữ liệu thành các phần bằng nhau:
```
Q1 (20%) | Q2 (40%) | Q3 (60%) | Q4 (80%) | Q5 (100%)
  Điểm 1 |  Điểm 2  |  Điểm 3  |  Điểm 4  |   Điểm 5
```

```python
# R: Recency thấp = tốt → đảo ngược thang điểm
rfm['R_Score'] = pd.qcut(rfm['Recency'],
                          q=5,
                          labels=[5, 4, 3, 2, 1])  # Đảo ngược!

# F và M: cao = tốt → điểm tăng dần
rfm['F_Score'] = pd.qcut(rfm['Frequency'].rank(method='first'),
                          q=5,
                          labels=[1, 2, 3, 4, 5])

rfm['M_Score'] = pd.qcut(rfm['Monetary'],
                          q=5,
                          labels=[1, 2, 3, 4, 5])
```

**Tại sao dùng rank() cho Frequency?**
→ Vì Frequency thường có nhiều giá trị bằng nhau (tie), `rank(method='first')` phá vỡ tie bằng cách dùng thứ tự xuất hiện.

---

## 4. Phân khúc khách hàng (Customer Segments)

### 4.1 Các nhóm phân khúc chính

```
R=5, F=5, M=5 → 💎 CHAMPIONS
R≥4, F≥4, M≥4 → 💎 Champions
R≥3, F≥3, M≥3 → 💙 Loyal Customers
R≥4, F≤2      → 🌱 New Customers
R≥3, F≥2, M≥2 → 🌟 Potential Loyalists
R=3, F≤2      → 🔮 Promising
R≤2, F≥3, M≥3 → ⚠️  At Risk
R≤2, F≥4, M≥4 → 🚨 Cannot Lose Them
R≤2, F≤2, M≤2 → 👻 Lost
Còn lại        → 😴 Hibernating
```

### 4.2 Đặc điểm từng nhóm

**💎 Champions (Nhóm xuất sắc nhất)**
```
Recency  : Rất gần đây (R=4-5)
Frequency: Rất thường xuyên (F=4-5)
Monetary : Chi tiêu rất cao (M=4-5)
Hành động: VIP program, early access, referral program
```

**💙 Loyal Customers (Khách trung thành)**
```
Recency  : Gần đây (R=3-4)
Frequency: Thường xuyên (F=3-4)
Monetary : Chi tiêu khá (M=3-4)
Hành động: Loyalty points, membership benefits
```

**⚠️ At Risk (Có nguy cơ rời bỏ)**
```
Recency  : Lâu rồi không mua (R=1-2)
Frequency: Từng mua thường xuyên (F≥3)
Monetary : Từng chi nhiều (M≥3)
Hành động: Win-back email, "Chúng tôi nhớ bạn", discount 20-30%
```

**🌱 New Customers (Khách hàng mới)**
```
Recency  : Mới mua gần đây (R=4-5)
Frequency: Mới mua 1-2 lần (F≤2)
Monetary : Chi tiêu chưa cao
Hành động: Welcome series, hướng dẫn sản phẩm, ưu đãi lần 2
```

**👻 Lost (Đã rời bỏ)**
```
Recency  : Rất lâu rồi (R=1-2)
Frequency: Ít (F=1-2)
Monetary : Thấp (M=1-2)
Hành động: Last-chance offer hoặc loại khỏi danh sách
```

---

## 5. Các kỹ thuật Python sử dụng

### 5.1 pd.qcut() vs pd.cut()

```python
# pd.cut() — chia theo khoảng giá trị bằng nhau
pd.cut(data, bins=5)   # Mỗi bin có khoảng giá trị = nhau

# pd.qcut() — chia theo số lượng phần tử bằng nhau
pd.qcut(data, q=5)     # Mỗi bin có số phần tử = nhau
```

**Trong RFM dùng qcut vì:**
→ Muốn mỗi nhóm điểm có số lượng khách hàng tương đương nhau
→ Nếu dùng cut, nhóm điểm 5 có thể chỉ có vài khách hàng

### 5.2 Lambda Function trong groupby

```python
# Cách thông thường
def tinh_recency(x):
    return (reference_date - x.max()).days

# Lambda — viết gọn hơn
rfm = df.groupby('Customer ID').agg(
    Recency = ('InvoiceDate', lambda x: (reference_date - x.max()).days)
)
```

### 5.3 Apply với hàm tự định nghĩa

```python
def segment_customer(row):
    r, f, m = row['R_Score'], row['F_Score'], row['M_Score']
    if r >= 4 and f >= 4 and m >= 4:
        return 'Champions'
    elif r <= 2 and f <= 2 and m <= 2:
        return 'Lost'
    # ...

rfm['Segment'] = rfm.apply(segment_customer, axis=1)
# axis=1 → áp dụng theo hàng (mỗi khách hàng)
# axis=0 → áp dụng theo cột
```

---

## 6. Visualization cho RFM

### 6.1 Scatter Plot — Recency vs Monetary
**Mục đích:** Nhìn thấy phân bổ khách hàng theo 2 chiều

```python
plt.scatter(rfm['Recency'], rfm['Monetary'],
            c=rfm['Segment'].map(color_dict),
            alpha=0.6)
```

**Đọc chart:**
- Góc trên trái (Recency thấp, Monetary cao) → Champions lý tưởng
- Góc dưới phải (Recency cao, Monetary thấp) → Lost customers

### 6.2 Log Scale cho Monetary
```python
ax.set_yscale('log')
```
**Tại sao cần log scale?**
→ Monetary có outlier rất lớn (600K GBP) làm phần lớn điểm bị dồn về 0
→ Log scale giúp nhìn rõ hơn sự phân bổ ở mức thấp

### 6.3 Pie Chart cho Segment Distribution
**Đọc chart:**
- Mảnh to = nhiều khách hàng
- Nhưng không thể hiện doanh thu → cần kết hợp Bar Chart Revenue

---

## 7. Business Recommendations (Đề xuất kinh doanh)

Đây là phần **quan trọng nhất** — điểm khác biệt giữa DA giỏi và DA bình thường.

### Framework RACE:
```
R — Reach    : Tiếp cận đúng nhóm khách hàng
A — Act      : Khuyến khích hành động mua hàng
C — Convert  : Chuyển đổi thành doanh thu
E — Engage   : Giữ chân và tạo loyalty
```

### Đề xuất theo từng nhóm:

**💎 Champions (1,300 KH — 68% doanh thu):**
```
→ VIP membership với quyền lợi đặc biệt
→ Early access sản phẩm mới
→ Referral program: giới thiệu bạn bè nhận thưởng
→ Personalized thank-you gift
```

**⚠️ At Risk (615 KH — 8.72% doanh thu):**
```
→ Email win-back: "Chúng tôi nhớ bạn!"
→ Discount 20-30% cho lần mua tiếp theo
→ Survey ngắn để hiểu lý do không mua
→ Thời hạn ưu đãi rõ ràng (urgency)
```

**🌱 New Customers (443 KH):**
```
→ Welcome email series (3-5 email trong 30 ngày đầu)
→ Giảm giá lần mua thứ 2
→ Hướng dẫn sử dụng sản phẩm
→ Gợi ý sản phẩm liên quan
```

**👻 Lost (1,275 KH — 1.85% doanh thu):**
```
→ Last-chance campaign với ưu đãi lớn
→ Nếu không phản hồi sau 2-3 email → remove khỏi list
→ Tiết kiệm chi phí marketing cho nhóm tiềm năng hơn
```

---

## 8. Metrics đánh giá hiệu quả RFM

### 8.1 Customer Lifetime Value (CLV)
```
CLV = Avg Order Value × Purchase Frequency × Customer Lifespan
```
- Ước tính tổng giá trị 1 khách hàng mang lại trong suốt vòng đời

### 8.2 Churn Rate (Tỉ lệ rời bỏ)
```
Churn Rate = Số khách rời bỏ / Tổng số khách đầu kỳ × 100%
```
- Giảm churn rate 5% có thể tăng lợi nhuận 25-95%

### 8.3 Retention Rate (Tỉ lệ giữ chân)
```
Retention Rate = 100% - Churn Rate
```

---

## 9. Hạn chế của RFM

| Hạn chế | Giải thích | Giải pháp |
|---|---|---|
| Không xét sản phẩm | Không biết khách mua gì | Kết hợp với product analysis |
| Không xét lý do | Không biết tại sao mua/không mua | Kết hợp survey, feedback |
| Ngưỡng chủ quan | Rule-based segmentation phụ thuộc người làm | Dùng K-means clustering |
| Snapshot in time | Chỉ phản ánh 1 thời điểm | Chạy RFM định kỳ (hàng tháng) |

---

## 10. Nâng cao — K-Means Clustering

Thay vì rule-based (thủ công), có thể dùng machine learning:

```python
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans

# Chuẩn hóa dữ liệu (quan trọng!)
scaler = StandardScaler()
rfm_scaled = scaler.fit_transform(rfm[['Recency','Frequency','Monetary']])

# Tìm số cluster tối ưu bằng Elbow Method
inertia = []
for k in range(2, 11):
    km = KMeans(n_clusters=k, random_state=42)
    km.fit(rfm_scaled)
    inertia.append(km.inertia_)

# Chọn k tại điểm "khuỷu tay" trên đồ thị
```

**Ưu điểm của K-Means:**
- Khách quan, không phụ thuộc quy tắc người làm
- Tự động tìm ra các nhóm tự nhiên trong dữ liệu
- Có thể tái chạy định kỳ để cập nhật phân khúc

---

## 11. Câu hỏi phỏng vấn thường gặp

**Q: RFM là gì và ứng dụng trong thực tế thế nào?**
A: RFM là phương pháp phân khúc khách hàng dựa trên Recency, Frequency, Monetary. Ứng dụng trong: email marketing cá nhân hóa, phân bổ ngân sách marketing, chiến lược giữ chân khách hàng.

**Q: Tại sao Recency điểm cao lại là R thấp?**
A: Vì Recency đo số ngày từ lần mua cuối đến hiện tại — số ngày càng nhỏ nghĩa là mua càng gần đây, tức là khách hàng đang hoạt động tốt. Nên R thấp = tốt, được cho điểm cao.

**Q: Tại sao dùng qcut thay vì cut?**
A: qcut chia đều số lượng phần tử vào mỗi bin, giúp mỗi nhóm điểm có số lượng khách hàng tương đương. cut chia theo khoảng giá trị đều, có thể tạo ra nhóm rất nhiều và nhóm rất ít.

**Q: Sau khi phân khúc xong, bạn đề xuất gì cho business?**
A: Tập trung nguồn lực vào Champions và At Risk. Champions mang lại ROI cao nhất khi giữ chân. At Risk có giá trị cao nhưng đang có nguy cơ rời bỏ — đây là cơ hội win-back tốt nhất.

**Q: Hạn chế của RFM là gì?**
A: RFM chỉ dựa trên hành vi giao dịch, không xét đến: sản phẩm khách mua, lý do mua/không mua, yếu tố nhân khẩu học. Có thể cải thiện bằng cách kết hợp K-Means clustering hoặc thêm features khác.

**Q: CLV là gì và liên quan thế nào đến RFM?**
A: Customer Lifetime Value là tổng giá trị khách hàng mang lại trong suốt vòng đời. RFM giúp xác định CLV hiện tại, từ đó dự đoán CLV tương lai và ưu tiên đầu tư vào đúng nhóm khách hàng.

---

*Tài liệu này được biên soạn dựa trên thực tế thực hiện dự án RFM Customer Segmentation — Online Retail II Dataset.*
