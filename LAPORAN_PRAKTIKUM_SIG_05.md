# LAPORAN PRAKTIKUM SISTEM INFORMASI GEOGRAFIS (SIG)
## MODUL 05: APLIKASI QUANTUM GIS - 2 (GEOJSON & DATA NON-SPASIAL)

---

**Mata Kuliah:** Sistem Informasi Geografis  
**Program Studi:** Teknik Informatika / Sistem Informasi  
**Institusi:** STT Terpadu Nurul Fikri  
**Tahun Akademik:** 2026/2027  

---

## DAFTAR ISI
1. [Tujuan Praktikum](#1-tujuan-praktikum)
2. [Tugas Pendahuluan](#2-tugas-pendahuluan)
3. [Praktikum 1.1: Pembuatan Dataset Titik GeoJSON di geojson.io](#3-praktikum-11-pembuatan-dataset-titik-geojson-di-geojsonio)
4. [Praktikum 1.2: Input Dataset GeoJSON dan Export ke Shapefile pada QGIS](#4-praktikum-12-input-dataset-geojson-dan-export-ke-shapefile-pada-qgis)
5. [Praktikum Mandiri: Pemetaan Poligon Provinsi di Pulau Jawa](#5-praktikum-mandiri-pemetaan-poligon-provinsi-di-pulau-jawa)
6. [Kesimpulan](#6-kesimpulan)
7. [Daftar Berkas Proyek](#7-daftar-berkas-proyek)

---

## 1. Tujuan Praktikum
Setelah menyelesaikan praktikum ini, mahasiswa diharapkan mampu:
1. Memahami konsep, sintaks, dan pembuatan file format **GeoJSON** untuk data spasial.
2. Mengintegrasikan dataset GeoJSON ke dalam perangkat lunak **Quantum GIS (QGIS)**.
3. Melakukan konversi format data vektor (GeoJSON ke Shapefile `.shp`).
4. Menambahkan, memanipulasi, dan mengelola dataset atribut **non-spasial** (luas, populasi, tahun, dan GDP) pada attribute table layer peta di QGIS.
5. Memvisualisasikan data poligon multi-layer wilayah administratif (Provinsi se-Pulau Jawa) berbasis data terbuka GitHub.

---

## 2. Tugas Pendahuluan

### Pertanyaan 1: Jelaskan apa yang dimaksud dengan GeoJSON?
**Jawaban:**  
**GeoJSON** (*Geographic JSON*) adalah standar format pertukaran data geospasial berbasis teks terbuka (*open standard format*) yang menggunakan notasi objek JavaScript (*JavaScript Object Notation / JSON*). Format ini distandarisasi oleh IETF (*Internet Engineering Task Force*) di bawah spesifikasi **RFC 7946**.

GeoJSON dirancang untuk merepresentasikan fitur geografis sederhana (*simple geographical features*) beserta atribut non-spasialnya. GeoJSON mendukung berbagai jenis geometri spasial, di antaranya:
- **Point** (titik, misal: lokasi kota, fasilitas umum).
- **LineString** (garis, misal: jalan, sungai, jalur pipa).
- **Polygon** (area tertutup, misal: batas wilayah, danau, kavling).
- **MultiPoint, MultiLineString, dan MultiPolygon** (kumpulan dari geometri bertipe sama).
- **GeometryCollection** (kumpulan dari geometri dengan tipe berbeda).

Keunggulan utama GeoJSON adalah formatnya yang berbasis teks (*human-readable*), ringan (*lightweight*), dan mudah diproses secara *native* oleh web browser serta pustaka pemetaan modern seperti Leaflet, Mapbox GL JS, OpenLayers, maupun software SIG desktop seperti QGIS dan ArcGIS.

---

### Pertanyaan 2: Informasi apa saja yang dapat tersimpan dalam file format GeoJSON?
**Jawaban:**  
Sebuah file GeoJSON dapat menyimpan dua kategori informasi utama yang saling terhubung:

1. **Informasi Spasial (Geometri & Koordinat):**
   - **Tipe Geometri (`type`)**: Menentukan model representasi spasial (misal: `"Point"`, `"Polygon"`, `"LineString"`).
   - **Array Koordinat (`coordinates`)**: Pasangan nilai posisi geografis yang secara standar menggunakan sistem koordinat **WGS 84 (EPSG:4326)** dengan urutan baku:  
     `[Longitude (Bujur), Latitude (Lintang), Altitude (Ketinggian - opsional)]`.
   - **Bounding Box (`bbox`)**: Batas cakupan area geografis minimum dan maksimum dari seluruh fitur (*bounding box* opsional).

2. **Informasi Non-Spasial (Atribut / Metadata):**
   - Disimpan di dalam objek **`properties`** dalam bentuk pasangan *key-value*.
   - Atribut ini dapat berupa tipe data teks (*string*), angka (*integer/decimal*), boolean, tanggal, array, maupun objek bersarang (*nested object*).
   - Contoh informasi non-spasial: nama daerah, kode administrasi (BPS/Kemendagri), jumlah penduduk, luas wilayah, data Produk Domestik Regional Bruto (PDRB/GDP), tanggal pemutakhiran, dsb.

3. **Struktur Pembungkus (Struktural):**
   - **Feature**: Objek mandiri yang menggabungkan geometri spasial (`geometry`), atribut deskriptif (`properties`), dan pengenal unik (`id`).
   - **FeatureCollection**: Kumpulan (*array*) dari satu atau lebih objek `Feature` dalam satu kesatuan dokumen spasial.

---

### Pertanyaan 3: Berikan contoh dataset format GeoJSON, dan jelaskan seluruh fitur yang ada dalam dataset tersebut?
**Jawaban:**  
Berikut adalah contoh konkret dataset format GeoJSON yang merepresentasikan titik Kota Jakarta beserta atribut demografi dan ekonominya:

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "ID-JKT",
      "properties": {
        "nama": "DKI Jakarta",
        "kategori": "Ibu Kota / Pusat Bisnis",
        "luas_km2": 661.50,
        "populasi_ribu": 10684.90,
        "tahun": 2024,
        "GDP_billion_usd": 211.50
      },
      "geometry": {
        "type": "Point",
        "coordinates": [106.8456, -6.2088]
      }
    }
  ]
}
```

**Penjelasan Fitur dan Komponen dalam Dataset Tersebut:**
1. `"type": "FeatureCollection"`: Menyatakan bahwa file tersebut adalah wadah (*container*) tingkat teratas yang menaungi kumpulan fitur geografis.
2. `"features"`: Array yang berisi daftar seluruh objek fitur di dalam layer.
3. `"type": "Feature"`: Menandakan sebuah objek fitur geografis individual yang utuh.
4. `"id": "ID-JKT"`: *Identifier* unik untuk membedakan fitur ini dari fitur lainnya dalam basis data.
5. `"properties"`: Wadah data non-spasial (tabel atribut) yang mencakup:
   - `"nama"`: Nama daerah geografis (*string*).
   - `"kategori"`: Karakteristik atau fungsi wilayah (*string*).
   - `"luas_km2"`: Luas daratan wilayah dalam satuan kilometer persegi (*numeric/decimal*).
   - `"populasi_ribu"`: Jumlah penduduk terkini tahun 2024 dalam satuan ribuan jiwa (*numeric/decimal*).
   - `"tahun"`: Tahun referensi pencatatan data statistik (*integer*).
   - `"GDP_billion_usd"`: Indikator perekonomian daerah (PDRB/GDP nominal) dalam miliar Dolar AS (*numeric/decimal*).
6. `"geometry"`: Mendefinisikan posisi spasial di permukaan bumi:
   - `"type": "Point"`: Menyatakan geometri berupa titik tunggal koordinat.
   - `"coordinates": [106.8456, -6.2088]`: Koordinat lokasi presisi di mana `106.8456` adalah garis bujur (*Longitude*, timur) dan `-6.2088` adalah garis lintang (*Latitude*, selatan).

---

## 3. Praktikum 1.1: Pembuatan Dataset Titik GeoJSON di geojson.io

### 3.1 Langkah Kerja
1. Mengakses situs web pembuat GeoJSON interaktif di [https://geojson.io](https://geojson.io).
2. Mengarahkan kanvas peta ke Pulau Jawa, dengan fokus wilayah aglomerasi Jabodetabek (Jakarta, Bogor, Tangerang, Tangerang Selatan, Depok, dan Bekasi).
3. Menggunakan tool *Marker (Point)* pada panel toolbar sebelah kiri untuk menandai titik pusat ke-6 kota tersebut.
4. Memasukkan atribut non-spasial tahun 2024 untuk masing-masing titik melalui panel properti (nama, luas dalam $km^2$, populasi dalam ribuan jiwa, tahun 2024, dan GDP).
5. Memverifikasi seluruh data spasial dan non-spasial melalui tampilan **Table** di panel sebelah kanan geojson.io.
6. Mengunduh hasil titik koordinat dalam format file `jabotabek.geojson`.

### 3.2 Tabel Data Non-Spasial Jabodetabek (Tahun 2024)
| No | Nama Kota / Daerah | Koordinat [Long, Lat] | Luas ($km^2$) | Populasi 2024 (Ribu Jiwa) | Tahun | GDP (Miliar USD) |
|:--:|:-------------------|:---------------------:|:-------------:|:-------------------------:|:-----:|:----------------:|
| 1  | **Jakarta**           | [106.8456, -6.2088]   | 661.50        | 10,684.90                 | 2024  | 211.50           |
| 2  | **Tangerang**         | [106.6319, -6.1783]   | 164.50        | 1,927.82                  | 2024  | 14.17            |
| 3  | **Tangerang Selatan** | [106.7179, -6.2889]   | 147.19        | 1,429.52                  | 2024  | 6.85             |
| 4  | **Depok**             | [106.7942, -6.4025]   | 200.29        | 2,163.64                  | 2024  | 5.75             |
| 5  | **Bekasi**            | [106.9756, -6.2383]   | 204.52        | 2,528.13                  | 2024  | 7.81             |
| 6  | **Bogor**             | [106.7972, -6.5971]   | 118.50        | 1,067.80                  | 2024  | 3.74             |

### 3.3 Tangkapan Layar (Screenshots) geojson.io

#### Tampilan 1: Tabel Data Titik Koordinat di geojson.io
![Tampilan Tabel di geojson.io](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/1_geojson_io_table_view.png)
*Gambar 3.1: Verifikasi tabel dataset titik Jabodetabek pada antarmuka web geojson.io.*

#### Tampilan 2: Feature Properties Editor di geojson.io
![Tampilan Properti Titik di geojson.io](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/2_geojson_io_feature_properties.png)
*Gambar 3.2: Formulir atribut non-spasial (luas, nama, populasi, tahun, GDP) pada salah satu titik fitur di geojson.io.*

---

## 4. Praktikum 1.2: Input Dataset GeoJSON dan Export ke Shapefile pada QGIS

### 4.1 Langkah Kerja
1. Membuka aplikasi **Quantum GIS (QGIS 4.2.3)** dan membuat dokumen project baru dengan nama `praktikum_qgis2.qgz`.
2. Menambahkan layer basemap **Google Maps (XYZ Tiles)** dengan URL koneksi `https://mt1.google.com/vt/lyrs=m&x={x}&y={y}&z={z}` agar sebaran titik memiliki konteks visual topografi jalan dan perkotaan.
3. Memasukkan file `jabotabek.geojson` ke dalam panel Layers QGIS.
4. Mengekspor layer GeoJSON tersebut ke format **ESRI Shapefile**:
   - Klik kanan pada layer `jabotabek.geojson` $\rightarrow$ pilih **Export** $\rightarrow$ **Save Features As...**.
   - Format: *ESRI Shapefile*.
   - Nama Berkas: `jabotabek.shp`.
   - CRS: *EPSG:4326 - WGS 84*.
5. Membuka tabel atribut pada layer shapefile `jabotabek` yang baru dibuat (*Open Attribute Table*).
6. Mengaktifkan modus penyuntingan data (*Toggle Editing Mode* / ikon pensil).
7. Menambahkan kolom field baru untuk indikator ekonomi:
   - Klik ikon **New Field** (atau shortcut `Ctrl + W`).
   - Name: `GDP`.
   - Type: `Decimal number (real)`.
   - Length: `10`, Precision: `2`.
8. Mengisi data GDP nominal untuk masing-masing kota merujuk pada data referensi *Wikipedia & BPS (List of Indonesian cities by GDP)*.
9. Menyimpan penyuntingan tabel dan menyimpan proyek QGIS (`praktikum_qgis2.qgz`).

### 4.2 Tangkapan Layar (Screenshots) Hasil QGIS Praktikum 2

#### Tampilan 3: Peta Titik Jabodetabek di Atas Google Maps (QGIS)
![Peta Titik Jabodetabek di QGIS](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/3_qgis_jabotabek_full_window.png)
*Gambar 4.1: Visualisasi layar penuh aplikasi QGIS menampilkan titik sebaran kota Jabodetabek dengan simbolisasi marker merah tegas dan pelabelan nama kota di atas Google Maps.*

#### Tampilan 4: Attribute Table Layer Shapefile dengan Kolom GDP Baru (QGIS)
![Attribute Table QGIS Jabotabek](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/5_qgis_attribute_table_jabotabek_full.png)
*Gambar 4.2: Tampilan layar penuh jendela aplikasi QGIS dengan tabel atribut layer jabotabek.shp setelah penambahan kolom field GDP dalam modus editing.*

---

## 5. Praktikum Mandiri: Pemetaan Poligon Provinsi di Pulau Jawa

### 5.1 Langkah Kerja
1. Membuat dokumen project baru di Quantum GIS dan menyimpannya dengan nama `praktikum_qgis4.qgz`.
2. Mengunduh data poligon batas administratif provinsi di Pulau Jawa dari repositori GitHub resmi [yusufsyaifudin/wilayah-indonesia](https://github.com/yusufsyaifudin/wilayah-indonesia):
   - `31.geojson`: DKI Jakarta
   - `32.geojson`: Jawa Barat
   - `33.geojson`: Jawa Tengah
   - `34.geojson`: D.I. Yogyakarta
   - `35.geojson`: Jawa Timur
   - `36.geojson`: Banten
3. Menyimpan seluruh berkas GeoJSON tersebut ke direktori komputer lokal di folder `provinsi_jawa/`.
4. Mengintegrasikan seluruh berkas GeoJSON poligon provinsi ke dalam project QGIS.
5. Memperkaya setiap layer poligon dengan data statistik non-spasial terkini tahun 2024:
   - **Tahun**: `2024`
   - **Populasi**: Jumlah penduduk dalam ribuan jiwa (merujuk publikasi sensus/proyeksi BPS 2024).
   - **Luas Wilayah**: Luas daratan dalam kilometer persegi ($km^2$).
6. Menerapkan gaya visual tematik (*thematic categorized symbology*) dengan skema warna yang kontras dan harmonis untuk setiap provinsi (sesuai Gambar referensi Modul 5 halaman 6), serta mengaktifkan label nama provinsi dengan efek *text buffer* putih.

### 5.2 Tabel Data Non-Spasial 6 Provinsi di Pulau Jawa (Tahun 2024)
| Kode BPS | Nama Provinsi | Tahun | Populasi 2024 (Ribu Jiwa) | Luas Wilayah ($km^2$) | Kepadatan ($Jiwa/km^2$) | Warna Simbol |
|:--------:|:--------------|:-----:|:-------------------------:|:---------------------:|:-----------------------:|:------------:|
| **31**   | **DKI Jakarta**     | 2024  | 10,684.90                 | 661.50                | 16,152.53               | Biru Muda (*Dodger Blue*) |
| **32**   | **Jawa Barat**      | 2024  | 50,345.20                 | 35,377.76             | 1,423.07                | Hijau (*Emerald*) |
| **33**   | **Jawa Tengah**     | 2024  | 37,892.30                 | 32,800.69             | 1,155.23                | Kuning (*Sunflower*) |
| **34**   | **D.I. Yogyakarta** | 2024  | 3,761.90                  | 3,133.15              | 1,200.68                | Oranye (*Carrot*) |
| **35**   | **Jawa Timur**      | 2024  | 41,814.50                 | 47,803.49             | 874.72                  | Merah (*Alizarin*) |
| **36**   | **Banten**          | 2024  | 12,435.80                 | 9,662.92              | 1,286.96                | Ungu (*Amethyst*) |

### 5.3 Tangkapan Layar (Screenshots) Hasil Praktikum Mandiri

#### Tampilan 5: Peta Tematik Poligon Provinsi Se-Pulau Jawa (QGIS)
![Peta Poligon Provinsi Jawa di QGIS](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/4_qgis_pulau_jawa_mandiri_full_window.png)
*Gambar 5.1: Hasil visualisasi layar penuh aplikasi QGIS menampilkan batas area poligon seluruh provinsi di Pulau Jawa dengan pewarnaan tematik pembeda dan basemap Google Maps.*

#### Tampilan 6: Attribute Table Data Non-Spasial 6 Provinsi Jawa
![Attribute Table Provinsi Jawa](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/6_qgis_attribute_table_provinsi_jawa_full.png)
*Gambar 5.2: Tampilan layar penuh jendela aplikasi QGIS menampilkan tabel atribut non-spasial (nama, tahun, populasi, luas) untuk 6 provinsi di Pulau Jawa.*

---

## 6. Kesimpulan
1. **Fleksibilitas GeoJSON**: GeoJSON terbukti sangat efisien sebagai jembatan pertukaran data spasial modern antara platform berbasis web (*geojson.io*) dan aplikasi desktop SIG (*QGIS*) karena strukturnya yang sederhana, terbaca manusia, dan terstandarisasi.
2. **Integrasi Data Spasial dan Non-Spasial**: SIG mampu menggabungkan informasi posisi absolut (koordinat lintang-bujur dan geometri batas wilayah) dengan indikator statistik tabular (populasi, luas wilayah, dan nilai ekonomi GDP). Penggabungan ini memungkinkan analis melakukan analisis keruangan (*spatial analysis*) maupun visualisasi tematik (*thematic choropleth mapping*).
3. **Interoperabilitas Format SIG**: Konversi dari GeoJSON ke ESRI Shapefile (`.shp`) di QGIS dapat dilakukan secara langsung tanpa kehilangan integritas geometri maupun atribut tabel aslinya.
4. **Pemanfaatan Data Terbuka (Open Data)**: Pemanfaatan repositori publik seperti GitHub untuk batas administratif wilayah Indonesia mempermudah otomasi alur kerja analisis geospasial tanpa perlu melakukan digitasi manual dari awal.

---

## 7. Daftar Berkas Proyek

Seluruh berkas praktikum telah disimpan secara terstruktur di direktori kerja:
`c:\Users\awalu\OneDrive\Документы\Kuliah\Semester 7\SIG\praktikum5`

1. **Dataset Spasial:**
   - [`jabotabek.geojson`](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/jabotabek.geojson) (File GeoJSON titik pusat 6 kota)
   - [`jabotabek.shp`](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/jabotabek.shp) (ESRI Shapefile beserta file pendukung `.shx`, `.dbf`, `.prj`, `.cpg`)
   - [`provinsi_jawa/`](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/provinsi_jawa/) (Folder berisi 6 file GeoJSON poligon provinsi Jawa)
2. **File Proyek QGIS (Siap Buka):**
   - [`praktikum_qgis2.qgz`](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/praktikum_qgis2.qgz) (Project QGIS untuk Praktikum 1.2)
   - [`praktikum_qgis4.qgz`](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/praktikum_qgis4.qgz) (Project QGIS untuk Praktikum Mandiri)
3. **Koleksi Screenshots Laporan (Full Screen UI):**
   - [`screenshots/1_geojson_io_table_view.png`](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/1_geojson_io_table_view.png)
   - [`screenshots/2_geojson_io_feature_properties.png`](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/2_geojson_io_feature_properties.png)
   - [`screenshots/3_qgis_jabotabek_full_window.png`](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/3_qgis_jabotabek_full_window.png)
   - [`screenshots/4_qgis_pulau_jawa_mandiri_full_window.png`](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/4_qgis_pulau_jawa_mandiri_full_window.png)
   - [`screenshots/5_qgis_attribute_table_jabotabek_full.png`](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/5_qgis_attribute_table_jabotabek_full.png)
   - [`screenshots/6_qgis_attribute_table_provinsi_jawa_full.png`](file:///c:/Users/awalu/OneDrive/Документы/Kuliah/Semester%207/SIG/praktikum5/screenshots/6_qgis_attribute_table_provinsi_jawa_full.png)
