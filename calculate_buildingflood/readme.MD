# Analisis Spasial Keterpaparan Bangunan terhadap Bahaya Banjir

**Studi Kasus: Kecamatan Bogor Tengah, Kota Bogor**

Proyek ini menghitung luas bangunan yang berada di zona bahaya banjir, dirinci per kelas bahaya (rendah, sedang, tinggi), lalu merekapnya sebagai indeks keterpaparan per kelurahan. Seluruh pengolahan dilakukan dengan QGIS.

## Pertanyaan Penelitian

Berapa luas bangunan di zona bahaya banjir per kelas bahaya, dan bagaimana sebarannya per kelurahan?

## Data

| Data | Sumber | Keterangan |
|---|---|---|
| Batas administrasi (kecamatan, kelurahan) | Badan Informasi Geospasial (BIG) | Polygon |
| Footprint bangunan | OpenStreetMap | Polygon, key `building` |
| Indeks bahaya banjir | InaRISK, BNPB | Polygon kelas rendah, sedang, tinggi |

- **Datum:** WGS 84
- **Proyeksi analisis:** UTM Zona 48 S (EPSG:32748)

## Alur Analisis

```
Bangunan OSM (1 kecamatan)
        |
        v
Fix Geometries
        |
        v
Select by Location (intersects) dengan zona banjir InaRISK
        |
        v
Reproject ke EPSG:32748
        |
        v
Field Calculator: skor_banjir, luas_ha, indeks
        |
        v
Statistics by Categories (luas per kelas bahaya)
        |
        v
Centroids bangunan --> Join Attributes by Location (summary) per kelurahan
        |
        v
Peta tematik indeks keterpaparan
```

### Tahapan

1. **Pengambilan data (retrieval).** Muat bangunan OSM dan batasi pada satu kecamatan. Perbaiki geometri tidak valid dengan *Fix Geometries*.
2. **Seleksi spasial.** *Select by Location* dengan predikat `intersects` untuk memilih bangunan yang bersinggungan dengan zona banjir, lalu ekspor menjadi layer baru.
3. **Reproject.** Ubah layer ke EPSG:32748 agar perhitungan luas dalam satuan meter valid.
4. **Reklasifikasi.** Ubah kelas bahaya menjadi skor angka: Rendah = 1, Sedang = 2, Tinggi = 3 (`skor_banjir`).
5. **Perhitungan atribut.**
   - `luas_ha` = `$area / 10000`
   - `indeks` = `luas_ha * skor_banjir`
6. **Pengukuran.** *Statistics by Categories* dengan kategori `skor_banjir` dan nilai `luas_ha` untuk mendapatkan total luas per kelas bahaya.
7. **Rekap per kelurahan.** Buat centroid bangunan, lalu jalankan *Join Attributes by Location (summary)* ke layer kelurahan dengan ringkasan `sum` dan `count` pada `luas_ha` dan `indeks`.
8. **Visualisasi.** Simbologi *Graduated* pada `indeks_sum`, lalu susun layout peta.

## Keluaran

- Tabel total luas bangunan terpapar per kelas bahaya
- Layer kelurahan berisi `luas_ha_sum`, `indeks_sum`, dan jumlah bangunan
- Peta tematik: *Indeks Keterpaparan Bangunan terhadap Bahaya Banjir per Kelurahan di Kecamatan Bogor Tengah, Kota Bogor*

## Hasil Utama

### Luas Bangunan Terpapar Banjir per Kelurahan (`luas_ha`)

| No. | Kelurahan | `luas_ha` (ha) |
|---|---|---|
| 1 | Babakan Pasar | 0,0061132 |
| 2 | Babakan | NULL |
| 3 | Cibogor | NULL |
| 4 | Ciwaringin | NULL |
| 5 | Gudang | NULL |
| 6 | Kebon Kelapa | 0,0211291 |
| 7 | Pabaton | NULL |
| 8 | Paledang | 0,0041635 |
| 9 | Panaragan | NULL |
| 10 | **Sempur** | **0,0312445** |
| 11 | Tegallega | NULL |

**Nilai luas tertinggi: Kelurahan Sempur (0,0312445 ha).**

Seluruh nilai luas bangunan terpapar per kelurahan tergolong **rendah** (nilai maksimum 0,0312445 ha).

Urutan nilai dari yang tertinggi: Sempur (0,0312445 ha), Kebon Kelapa (0,0211291 ha), Babakan Pasar (0,0061132 ha), Paledang (0,0041635 ha).

Catatan:
- Nilai **NULL** berarti tidak ada nilai `luas_ha` yang terekam pada kelurahan tersebut, yaitu tidak ada bangunan terpilih yang cocok dengan kelurahan itu pada hasil join.
- Nilai di atas bersumber dari tabel atribut hasil join per kelurahan. Jika nilai ini adalah hasil **sum**, maka angka tersebut merupakan total luas bangunan terpapar per kelurahan.

### Luas Bangunan per Kelas Bahaya

| Kelas Bahaya | Skor | Luas Bangunan (ha) |
|---|---|---|
| Rendah | 1 | 0.0626503 ha |
| Sedang | 2 | ... |
| Tinggi | 3 | ... |

> Isi dari hasil Statistics by Categories (kolom `sum`).

## Keterbatasan

- Metode *Select by Location* menghitung **luas utuh** bangunan terpilih, sehingga bangunan yang hanya sebagian masuk zona tetap dihitung penuh. Hasil cenderung sedikit melebihi nilai sebenarnya dan sebaiknya dibaca sebagai **estimasi**.
- Untuk hasil lebih presisi, gunakan overlay **Intersection** agar bangunan terpotong sesuai batas zona.
- Zona banjir InaRISK berbasis grid sehingga batas zona kasar.
- Kelengkapan footprint bangunan OSM berbeda-beda antarwilayah.
- Penggunaan centroid memastikan tiap bangunan hanya masuk satu kelurahan.

## Struktur Repositori

```
.
├── data/        # data mentah dan hasil olahan (shp/gpkg)
├── output/      # tabel statistik dan peta (png/pdf)
├── project/     # file proyek QGIS (.qgz)
└── README.md
```

> Sesuaikan struktur dengan isi repositori Anda.

## Output 

[Unduh layout peta (PDF)](output/LAYOUT.pdf)

## Perangkat Lunak

- QGIS 3.x (Processing Toolbox: Fix Geometries, Select by Location, Reproject Layer, Field Calculator, Statistics by Categories, Centroids, Join Attributes by Location)

## Sumber Data dan Atribusi

- Badan Informasi Geospasial (BIG): batas administrasi
- OpenStreetMap contributors: data bangunan, lisensi [ODbL](https://www.openstreetmap.org/copyright)
- InaRISK, Badan Nasional Penanggulangan Bencana (BNPB): indeks bahaya banjir

## Penulis

Nama, instansi/kampus, 2026
