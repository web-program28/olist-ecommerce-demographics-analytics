# End-to-End Analytics: Brazilian E-Commerce Spatial Distribution & Customer Segmentation

## 📌 Executive Summary
Proyek ini merupakan analisis data demografi secara menyeluruh (end-to-end) terhadap **99.441 data pelanggan** e-commerce Olist di Brasil. Menggunakan kombinasi **SQL (MySQL)** untuk ekstraksi database relasional, **R Programming** untuk kontrol integritas data, dan **Tableau** untuk pemetaan geospasial makro, proyek ini berhasil memetakan wilayah konsentrasi pasar utama guna memberikan rekomendasi alokasi anggaran pemasaran yang efisien kepada manajemen.

---

## 🛠️ Tech Stack & Architecture
* **Database Management:** MySQL (Advanced querying, Window Functions, CTE, Logika Kondisional)
* **Data Programming:** RStudio / R Programming (Data parsing error handling, Descriptive statistics, `ggplot2` visualization)
* **Data Storytelling:** Tableau Public (Choropleth mapping, Diverging color scaling)

---

## 💻 Technical Implementation

### 1. Database Extraction & Manipulation (MySQL)
Proyek diawali dengan mengimpor dataset mentah ke dalam MySQL Workbench. Guna mengidentifikasi wilayah penopang bisnis utama, saya menerapkan **Analisis Pareto 80/20** menggunakan *Window Function* (`OVER`) dan *Common Table Expression* (CTE):

<details>
<summary><b>📖 Klik di sini untuk melihat Query Pareto SQL</b></summary>

```sql
WITH akumulasi_pelanggan AS (
    SELECT 
        customer_state,
        COUNT(customer_id) AS jumlah_pelanggan,
        SUM(COUNT(customer_id)) OVER(ORDER BY COUNT(customer_id) DESC) AS running_total,
        SUM(COUNT(customer_id)) OVER() AS grand_total
    FROM olist_customers_dataset
    GROUP BY customer_state
)
SELECT 
    customer_state AS provinsi,
    jumlah_pelanggan,
    ROUND((running_total / grand_total) * 100, 2) AS persentase_kumulatif
FROM akumulasi_pelanggan;
Selain itu, saya merancang skema klasifikasi pasar berbasis wilayah (Rule-Based Clustering) dengan fungsi CASE WHEN untuk membagi kota-kota ke dalam tier kepadatan pasar tertentu:

```sql
SELECT
    customer_city AS nama_kota,
    COUNT(customer_id) AS total_pelanggan,
    CASE
        WHEN COUNT(customer_id) > 5000 THEN 'Saturated Market (Top Tier)'
        WHEN COUNT(customer_id) BETWEEN 1000 AND 5000 THEN 'Developing Market (Mid Tier)'
        ELSE 'Spotted Market (Low Tier)'
    END AS kategori_pasar
FROM olist_customers_dataset
GROUP BY customer_city
ORDER BY total_pelanggan DESC;
50 GROUP BY customer_city
```

## 2. Data Integrity Control (R Programming)
Ketika memindahkan data dari database ke lingkungan RStudio, ditemukan kendala teknis berupa kegagalan pemisahan kolom (parsing error) akibat ketidakcocokan tanda pembatas (delimiter). Kendala ini diselesaikan secara programatik menggunakan argumen `sep = ";"` pada fungsi bawaan R:

```r
# Handling error parsing delimiter data mentah
data_pelanggan <- read.csv("~/Documents/PORTOFOLIO/Dataset/olist_customers_dataset.csv", sep = ";")

# Validasi struktur data pasca-perbaikan
str(data_pelanggan)

library(tidyverse)

# Analisis Agregat Nilai Unik
data_pelanggan %>%
  summarise(
    Total_Transaksi = n(),
    Total_Pelanggan_Unik = n_distinct(customer_unique_id),
    Total_Kota = n_distinct(customer_city),
    Total_Provinsi = n_distinct(customer_state)
  )

# Visualisasi Distribusi Frekuensi
ggplot(data = data_pelanggan, aes(x = fct_infreq(customer_state))) +
  geom_bar(fill = "steelblue", color = "black") +
  theme_minimal() +
  labs(
    title = "Distribusi Frekuensi Pelanggan Berdasarkan Kode Provinsi",
    x = "Kode Provinsi (State)",
    y = "Jumlah Pelanggan"
  ) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

3. Geospatial Visualization (Tableau)
Untuk menjembatani temuan teknis dengan kebutuhan pemangku kepentingan non-teknis, data ditransformasikan ke dalam peta kepadatan wilayah (Choropleth Map) menggunakan skala gradasi Red-Blue Diverging di Tableau:
