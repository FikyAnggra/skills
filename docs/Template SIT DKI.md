

![][image1]

**System Integration Testing**  
Dana Kripto Indonesia Versi 1.0

| Disiapkan Oleh | : | Fiky Anggra Sepriaji |
| :---- | :---- | :---- |
| Tanggal | : | 08 Agustus 2025 |
| Nomor Dokumen | : |  |

[**1\. LATAR BELAKANG PROYEK	3**](#heading=)

[**2\. TUJUAN PROYEK	3**](#heading=)

[**3\. SOLUTION OVERVIEW PROYEK	3**](#heading=)

[A. Ruang Lingkup	3](#heading=)

[B. Batasan Dan Asumsi	3](#heading=)

[**4\. EXECUTIVE SUMMARY	3**](#heading=)

[**5\. LINGKUNGAN PENGUJIAN	3**](#heading=)

[**6\. SKENARIO DAN HASIL PENGUJIAN	4**](#heading=)

[**7\. KRITERIA KEBERHASILAN	4**](#heading=)

[**8\. RENCANA TINDAK LANJUT	4**](#heading=)

[**9\. LAMPIRAN	4**](#heading=)

# 

# 

1. # **LATAR BELAKANG PROYEK**

   Sesuai yang tertera pada dokumen BRD \[Nomor BRD\]

2. # **TUJUAN PROYEK**

   Sesuai yang tertera pada dokumen BRD \[Nomor BRD\]

3. # **SOLUTION OVERVIEW PROYEK**

1. ## **Ruang Lingkup**

   Cakupan sistem yang dikembangkan yakni

2. ## **Batasan Dan Asumsi**

   Fokus pengujian pada aspek yakni

4. # **EXECUTIVE SUMMARY**

   Fokus pengujian pada fitur ini yaitu 

| No | Fitur | Jumlah Skenario SIT | Hasil Pengujian |
| :---: | ----- | ----- | ----- |
| **1** | \[Nama Project\] | **\[Total skenario\]** | **Semua Skenario SIT Lolos Pengujian** |

   

   Secara keseluruhan atas semua fitur yang diuji, semua skenario SIT telah dilaksanakan dan hasilnya semua skenario SIT telah lolos pengujian, untuk detail skenario SIT secara lengkap dapat dilihat pada bagian ‘VI. SKENARIO DAN HASIL PENGUJIAN’ pada dokumen laporan ini.

5. # **LINGKUNGAN PENGUJIAN**

   Berikut adalah spesifikasi lingkungan pengujian SIT terkait fitur yang ada dalam dokumen ini yakni \[lingkungan pengujian\]

| Nama Sistem | Spesifikasi OS | Spesifikasi Aplikasi | Spesifikasi server | Spesifikasi Security |
| ----- | ----- | ----- | ----- | ----- |
| AFI Core System  | CentOS 7.6 64 bit | linux/apollo linux/javalinux/python | CPU: 4 vCPU | Gateway: 2 Instance |
|  |  |  | Ram: 8 GiB | Load Balancer: 1 CLB, 2 Listener |
|  |  |  | Storage: 140 GiB | WAF: Active |
|  |  |  |  | Firewall: allow 18081, 80, 443 |

6. # **SKENARIO DAN HASIL PENGUJIAN**

1. \[Nama Project\]  
   Modul: \[Modul\]  
   Tanggal Pengujian: \[Tanggal Pengujian\]  
   

| No. | Skenario | Langkah-Langkah | Hasil yang diharapkan | Hasil yang dicapai | Hasil |
| :---: | :---: | :---: | :---: | :---: | :---: |
|  |  |  |  |  |  |

7. # **KRITERIA KEBERHASILAN**

   

| No. | Kriteria Keberhasilan | Metrik Pengukuran | Target |
| :---: | :---: | :---: | :---: |
|  |  |  |  |

8. # **RENCANA TINDAK LANJUT**

   Berisi rencana untuk menindaklanjuti temuan selama SIT. Termasuk tindakan perbaikan atau perubahan yang diperlukan, serta tanggung jawab dan jadwal penyelesaian 

9. # **LAMPIRAN**

   Berisi tentang dokumen pendukung seperti diagram, jadwal rinci, anggaran yang lebih terperinci, atau informasi tambahan yang relevan

**Approval**

| Disiapkan Oleh: |  |  |
| :---- | :---- | :---- |
|  |  |  |
| Jabatan | : | QA Engineer |
| Nama | : | Fiky Anggra Sepriaji |
| Tanggal | : |  |

| Disiapkan Oleh: |  |  |  |  |  |
| :---- | :---- | :---- | :---- | :---- | :---- |
|  |  |  |  |  |  |
| Jabatan | : |  | Jabatan | : |  |
| Nama | : |  | Nama | : |  |
| Tanggal | : |  | Tanggal | : |  |

| Disiapkan Oleh: |  |  |  |  |  |  |  |  |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
|  |  |  |  |  |  |  |  |  |
| Jabatan | : |  | Jabatan | : |  | Jabatan | : |  |
| Nama | : |  | Nama | : |  | Nama | : |  |
| Tanggal | : |  | Tanggal | : |  | Tanggal | : |  |

[image1]: <data:image/png;base64,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>