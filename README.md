# Thailand-Online-Fraud-Risk-Analysis

**การวิเคราะห์พื้นที่เสี่ยงของคดีหลอกลวงออนไลน์ในประเทศไทย โดยใช้เทคนิคการจัดกลุ่มข้อมูลและแดชบอร์ดเชิงโต้ตอบ**
*Risk Area Analysis of Online Fraud Cases in Thailand Using Data Clustering Techniques and an Interactive Dashboard*

โปรเจกต์การค้นคว้าอิสระ (Independent Study) หลักสูตรวิทยาศาสตรมหาบัณฑิต สาขาวิชาวิทยาการข้อมูลและการวิเคราะห์
ศูนย์วิเคราะห์ข้อมูลดิจิทัลอัจฉริยะพระจอมเกล้าลาดกระบัง (KMITL) — ปีการศึกษา 2569

---

## ภาพรวมโครงการ (Overview)

โครงการนี้วิเคราะห์ข้อมูลสถิติคดีหลอกลวงออนไลน์จาก **CCIB (กองบัญชาการตำรวจสืบสวนสอบสวนอาชญากรรมทางเทคโนโลยี)** ครอบคลุม 77 จังหวัดทั่วประเทศไทย ช่วงเดือนมีนาคม 2565 – ธันวาคม 2566 (22 เดือน) โดยมีวัตถุประสงค์หลัก 4 ข้อ:

1. วิเคราะห์รูปแบบและแนวโน้มของคดีหลอกลวงออนไลน์เชิงพื้นที่และเชิงเวลา
2. จัดกลุ่มพื้นที่ (จังหวัด) ตามระดับความเสี่ยง ด้วยเทคนิค Clustering
3. วิเคราะห์โปรไฟล์ผู้เสียหายเชิงพรรณนา (เพศ / อายุ / อาชีพ)
4. พัฒนา Interactive Dashboard เพื่อสนับสนุนการตัดสินใจเชิงนโยบาย

### ผลลัพธ์สำคัญ
- K-Means Clustering (K=2) ให้ผลจัดกลุ่มคุณภาพดี: **Silhouette Score = 0.6121**, **Davies-Bouldin Index = 0.5584**
- แบ่งพื้นที่เป็นกลุ่มเสี่ยงสูง 22.1% และกลุ่มเสี่ยงต่ำ 77.9%
- **กรุงเทพมหานคร นนทบุรี และเชียงใหม่** อยู่ในกลุ่มเสี่ยงสูงตลอดทั้ง 22 เดือน
- ผู้เสียหายเป็นเพศหญิง 63.8%, กลุ่มอายุ 30–44 ปี มีจำนวนคดีสูงสุด, อาชีพพนักงานเอกชน/รับจ้าง มีจำนวนคดีสูงสุดในกลุ่มที่ระบุอาชีพชัดเจน
- ผลทดสอบทางสถิติยืนยันว่าเพศ อายุ และอาชีพ มีความแตกต่างกันอย่างมีนัยสำคัญ

## ขั้นตอนการทำงาน (Pipeline & Methodology)

### Phase 1–2: Data Preparation (`01_ccib_data_pipeline.ipynb`)
- นำเข้าไฟล์ Excel ดิบจาก CCIB (สถิติจำนวนคดีออนไลน์, มูลค่าความเสียหาย แยกตามจังหวัด/เพศ/อายุ/อาชีพ)
- รัน pipeline (`ccib_pipeline.py`) เพื่อตรวจสอบคุณภาพข้อมูล (data quality check), ตรวจปีที่ไม่ตรงกับชื่อไฟล์ (year mismatch), และคัดกรองไฟล์ที่มีปัญหาออกโดยอัตโนมัติ
- Output ที่ได้แบ่งเป็น 3 ส่วน: `clean/` (ข้อมูลสะอาดพร้อมใช้), `rejected/` (ไฟล์ที่ถูกคัดออกพร้อมเหตุผล), `report/` (รายงานตรวจสอบคุณภาพข้อมูล)
- ไฟล์หลักที่ได้: `ccib_main_clean.parquet/csv` และ `ccib_cluster_base.parquet/csv` (ใช้ต่อใน Phase 4)

### Phase 3: Exploratory Data Analysis
- ตรวจสอบ Missing Value และสัดส่วนข้อมูล "ไม่ระบุ"
- วิเคราะห์การกระจายตัวของข้อมูล, ตรวจจับ Outlier ด้วย IQR และทำ Winsorization
- สร้างกราฟภาพรวม 6 ชุด และแผนที่ Heatmap ความเสี่ยงระดับจังหวัด (geopandas/folium)

### Phase 4: Feature Engineering & Risk Score
- คำนวณดัชนีความเสี่ยง: **RiskScore = 0.6 × Case Rate (normalized) + 0.4 × Growth Rate (normalized)**
- ทดสอบการแจกแจงข้อมูลด้วย Shapiro-Wilk Test
- กำหนดเกณฑ์ threshold แบบ Rule-based จาก Natural Break ของข้อมูลจริง (threshold = 0.40)
- ทำ Sensitivity Analysis ปรับน้ำหนักตัวแปรเพื่อยืนยันความเสถียรของสูตร

### Phase 5: Clustering & Risk Classification
- เปรียบเทียบ 3 เทคนิค: K-Means, DBSCAN (พร้อม parameter tuning ด้วย K-Distance Graph), และ Rule-based Classification
- เลือก K-Means (K=2) เป็นแนวทางหลัก เนื่องจากให้ผลจัดกลุ่มเสถียรและตีความเชิงนโยบายได้ชัดเจนที่สุด
- ตรวจสอบความเสถียรของผลลัพธ์ด้วย Sensitivity Analysis (Adjusted Rand Index)

### Phase 6: Statistical Testing
- ทดสอบสมมติฐานเชิงสถิติเกี่ยวกับความแตกต่างของคดีตามมิติเชิงพื้นที่ เวลา และโปรไฟล์ผู้เสียหาย (เพศ/อายุ/อาชีพ)

---

## เทคโนโลยีที่ใช้ (Tech Stack)

| หมวด | เครื่องมือ |
|---|---|
| ภาษา | Python 3 (Google Colab) |
| จัดการข้อมูล | pandas, pyarrow, openpyxl |
| Machine Learning | scikit-learn (K-Means, DBSCAN, StandardScaler) |
| สถิติ | scipy (Shapiro-Wilk, hypothesis testing) |
| การแสดงผล | matplotlib, seaborn, geopandas, folium, branca |
| แบบอักษรภาษาไทย | Google Fonts – Sarabun |

---

## วิธีใช้งาน (How to Run)

1. เปิดไฟล์ `notebooks/01_ccib_data_pipeline.ipynb` บน Google Colab เพื่อรัน pipeline ทำความสะอาดข้อมูลดิบจาก CCIB (ต้องมี `ccib_pipeline.py` และโฟลเดอร์ข้อมูลดิบ)
2. ปรับ path `--raw-dir` และ `--output-dir` ให้ตรงกับตำแหน่งไฟล์ของคุณใน Google Drive
3. เมื่อได้ไฟล์ clean (`ccib_main_clean.parquet`, `ccib_cluster_base.parquet`) แล้ว เปิด `notebooks/02_eda_clustering_dashboard.ipynb` เพื่อรันการวิเคราะห์ EDA, Risk Score, Clustering และ Statistical Testing ต่อ

> **หมายเหตุ:** ไฟล์ข้อมูลดิบจาก CCIB และไฟล์ output ขนาดใหญ่ไม่ได้รวมอยู่ใน repository นี้ เนื่องจากข้อจำกัดด้านขนาดไฟล์และความเป็นส่วนตัวของข้อมูล

---

## เอกสารประกอบ

เล่มรายงานฉบับสมบูรณ์ (บทที่ 1–5, ระเบียบวิธีวิจัย, ผลการวิเคราะห์ และข้อเสนอแนะเชิงนโยบาย) อยู่ที่ [`docs/IS_thesis_full_report.pdf`](docs/IS_thesis_full_report.pdf)

---

## 📜 License

โปรเจกต์นี้จัดทำขึ้นเพื่อวัตถุประสงค์ทางการศึกษาเป็นส่วนหนึ่งของการค้นคว้าอิสระ (Independent Study) — สงวนลิขสิทธิ์ตามข้อกำหนดของคณะวิทยาศาสตร์ สถาบันเทคโนโลยีพระจอมเกล้าเจ้าคุณทหารลาดกระบัง
