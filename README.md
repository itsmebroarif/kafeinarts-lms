# Flowchart Aplikasi LMS (Multi-Role)

Role: **Siswa, Guru, Karyawan, Kepala Sekolah, Kepala Yayasan, Admin TU, Superadmin**

Isi dokumen:
1. Alur Umum (Login, RBAC, Dashboard)
2. Alur Fitur per Role (7 diagram)
3. Alur Akses Data (Role, Service, Database)
4. Matriks Hak Akses (CRUD)
5. Alur Proses Lintas Role (Tugas, Nilai, Raport)
6. Fallback ASCII (jika Mermaid tidak ter-render)

---

## 1. Alur Umum: Login, RBAC, Dashboard

```mermaid
flowchart TD
    A(["Mulai"]) --> B["Halaman Login"]
    B --> C{"Kredensial valid?"}
    C -- "Tidak" --> B1["Tampilkan error / Lupa password"]
    B1 --> B
    C -- "Ya" --> D{"Akun aktif?"}
    D -- "Tidak" --> B2["Akun dinonaktifkan, hubungi Admin TU"]
    B2 --> Z
    D -- "Ya" --> E["Buat sesi / token + catat Log Audit"]
    E --> F["Middleware RBAC: baca role + permission"]
    F --> G{"Role pengguna?"}

    G -- "Siswa" --> S["Dashboard Siswa"]
    G -- "Guru" --> GU["Dashboard Guru"]
    G -- "Karyawan" --> K["Dashboard Karyawan"]
    G -- "Kepala Sekolah" --> KS["Dashboard Kepala Sekolah"]
    G -- "Kepala Yayasan" --> KY["Dashboard Kepala Yayasan"]
    G -- "Admin TU" --> AT["Dashboard Admin TU"]
    G -- "Superadmin" --> SA["Dashboard Superadmin"]

    S --> H["Menu sesuai role"]
    GU --> H
    K --> H
    KS --> H
    KY --> H
    AT --> H
    SA --> H

    H --> I{"Akses fitur diizinkan?"}
    I -- "Tidak" --> J["403 Forbidden + catat Log Audit"]
    J --> H
    I -- "Ya" --> L["Jalankan fitur lewat API / Service"]
    L --> M[("Database")]
    M --> N["Tampilkan hasil ke pengguna"]
    N --> O{"Lanjut?"}
    O -- "Ya" --> H
    O -- "Tidak" --> Z(["Logout, sesi dihapus"])
```

---

## 2. Alur Fitur per Role

### 2.1 Siswa

```mermaid
flowchart TD
    S(["Dashboard Siswa"]) --> S1["Profil Saya"]
    S --> S2["Jadwal Pelajaran"]
    S --> S3["Kelas dan Mata Pelajaran"]
    S --> S4["Absensi"]
    S --> S5["Ujian / Kuis Online"]
    S --> S6["Nilai dan Raport"]
    S --> S7["Pengumuman"]
    S --> S8["Forum Diskusi"]
    S --> S9["Tagihan SPP"]

    S1 --> S1a["Lihat / ubah data pribadi terbatas, ganti password"]

    S2 --> S2a[("Data Jadwal: baca saja")]

    S3 --> S3a["Pilih mata pelajaran"]
    S3a --> S3b["Lihat materi: PDF, video, tautan"]
    S3a --> S3c["Lihat daftar tugas"]
    S3c --> S3d["Kerjakan dan unggah jawaban"]
    S3d --> S3e{"Sebelum deadline?"}
    S3e -- "Ya" --> S3f[("Simpan Pengumpulan Tugas")]
    S3e -- "Tidak" --> S3g["Tandai terlambat / ditolak sesuai aturan"]
    S3g --> S3f
    S3f --> S3h["Notifikasi ke Guru"]

    S4 --> S4a["Presensi mandiri jika dibuka guru"]
    S4a --> S4b[("Data Absensi")]
    S4 --> S4c["Lihat rekap kehadiran pribadi"]

    S5 --> S5a["Pilih ujian aktif"]
    S5a --> S5b["Kerjakan dengan timer"]
    S5b --> S5c["Submit"]
    S5c --> S5d[("Simpan Jawaban + skor otomatis untuk pilihan ganda")]

    S6 --> S6a["Lihat nilai tugas, ujian, rapor: baca saja"]
    S6a --> S6b["Unduh raport PDF jika sudah dipublikasi"]

    S7 --> S7a[("Data Pengumuman: baca saja")]
    S8 --> S8a["Buat topik / balas diskusi"]
    S9 --> S9a["Lihat status tagihan dan riwayat pembayaran"]
```

### 2.2 Guru

```mermaid
flowchart TD
    G(["Dashboard Guru"]) --> G1["Kelas dan Mapel yang Diampu"]
    G --> G2["Jadwal Mengajar"]
    G --> G3["Absensi Siswa"]
    G --> G4["Kelola Ujian / Kuis"]
    G --> G5["Penilaian"]
    G --> G6["Input Nilai Akhir dan Catatan Raport"]
    G --> G7["Absensi Guru"]
    G --> G8["Komunikasi"]

    G1 --> G1a["Unggah / edit / hapus materi"]
    G1 --> G1b["Buat tugas + atur deadline"]
    G1b --> G1c[("Simpan Tugas")]
    G1c --> G1d["Notifikasi ke siswa kelas terkait"]

    G2 --> G2a[("Data Jadwal: baca saja")]

    G3 --> G3a["Pilih kelas dan pertemuan"]
    G3a --> G3b["Tandai hadir, izin, sakit, alpa"]
    G3b --> G3c[("Simpan Absensi")]

    G4 --> G4a["Buat bank soal"]
    G4a --> G4b["Susun ujian, atur waktu dan acak soal"]
    G4b --> G4c["Publikasikan"]
    G4c --> G4d[("Simpan Ujian")]

    G5 --> G5a["Lihat pengumpulan tugas"]
    G5a --> G5b["Beri nilai + komentar"]
    G5b --> G5c[("Simpan Nilai")]
    G5c --> G5d["Notifikasi ke siswa"]

    G6 --> G6a["Rekap nilai otomatis"]
    G6a --> G6b["Isi catatan wali kelas jika ada"]
    G6b --> G6c["Ajukan ke Kepala Sekolah untuk validasi"]

    G7 --> G7a[("Data Absensi Guru")]
    G8 --> G8a["Pengumuman kelas, pesan ke siswa / orang tua"]
```

### 2.3 Karyawan

```mermaid
flowchart TD
    K(["Dashboard Karyawan"]) --> K1["Profil Kepegawaian"]
    K --> K2["Absensi Karyawan"]
    K --> K3["Pengajuan Cuti / Izin"]
    K --> K4["Tugas dan Permintaan Internal"]
    K --> K5["Slip Gaji"]
    K --> K6["Pengumuman Internal"]

    K1 --> K1a["Lihat data pribadi, ajukan perubahan ke Admin TU"]

    K2 --> K2a["Check-in / Check-out"]
    K2a --> K2b[("Simpan Absensi Karyawan")]

    K3 --> K3a["Isi form pengajuan"]
    K3a --> K3b["Kirim ke Admin TU"]
    K3b --> K3c{"Disetujui?"}
    K3c -- "Ya" --> K3d[("Status Cuti: Disetujui")]
    K3c -- "Tidak" --> K3e[("Status Cuti: Ditolak + alasan")]
    K3d --> K3f["Notifikasi ke karyawan"]
    K3e --> K3f

    K4 --> K4a["Terima tugas dari Admin TU / Kepala Sekolah"]
    K4a --> K4b["Update status: proses, selesai"]
    K4b --> K4c[("Simpan Status Tugas")]

    K5 --> K5a["Lihat dan unduh slip gaji pribadi"]
    K6 --> K6a[("Data Pengumuman: baca saja")]
```

### 2.4 Kepala Sekolah

```mermaid
flowchart TD
    KS(["Dashboard Kepala Sekolah"]) --> A1["Monitoring Akademik"]
    KS --> A2["Monitoring Guru dan Karyawan"]
    KS --> A3["Validasi dan Persetujuan"]
    KS --> A4["Laporan"]
    KS --> A5["Pengumuman Sekolah"]
    KS --> A6["Supervisi"]

    A1 --> A1a["Statistik nilai per kelas / mapel"]
    A1 --> A1b["Rekap kehadiran siswa"]
    A1 --> A1c["Progres ujian dan tugas"]

    A2 --> A2a["Absensi dan kinerja guru"]
    A2 --> A2b["Absensi karyawan"]

    A3 --> A3a["Validasi nilai raport dari Guru"]
    A3a --> A3b{"Valid?"}
    A3b -- "Ya" --> A3c[("Raport: Dipublikasi")]
    A3b -- "Tidak" --> A3d["Kembalikan ke Guru + catatan revisi"]
    A3 --> A3e["Setujui cuti guru / karyawan jika diatur"]

    A4 --> A4a["Laporan akademik, kehadiran, kepegawaian"]
    A4a --> A4b["Ekspor PDF / Excel"]
    A4b --> A4c["Kirim ke Kepala Yayasan"]

    A5 --> A5a["Buat pengumuman, pilih target role"]
    A5a --> A5b[("Simpan Pengumuman")]

    A6 --> A6a["Catat hasil supervisi guru"]
    A6a --> A6b[("Simpan Catatan Supervisi")]
```

### 2.5 Kepala Yayasan

```mermaid
flowchart TD
    KY(["Dashboard Kepala Yayasan"]) --> Y1["Ringkasan Eksekutif"]
    KY --> Y2["Laporan Akademik"]
    KY --> Y3["Laporan SDM"]
    KY --> Y4["Laporan Keuangan"]
    KY --> Y5["Evaluasi Kebijakan"]

    Y1 --> Y1a["KPI: jumlah siswa, kelulusan, kehadiran, kinerja"]
    Y2 --> Y2a["Laporan dari Kepala Sekolah: baca saja"]
    Y3 --> Y3a["Jumlah guru, karyawan, absensi, cuti"]
    Y4 --> Y4a["Ringkasan SPP dan tunggakan"]
    Y5 --> Y5a["Catatan arahan ke Kepala Sekolah"]
    Y5a --> Y5b[("Simpan Arahan / Catatan")]
    Y5b --> Y5c["Notifikasi ke Kepala Sekolah"]

    Y1a --> Y6["Ekspor laporan PDF / Excel"]
    Y2a --> Y6
    Y3a --> Y6
    Y4a --> Y6
```

> Kepala Yayasan bersifat **read-only** pada data operasional. Hak tulis hanya pada catatan arahan.

### 2.6 Admin TU

```mermaid
flowchart TD
    AT(["Dashboard Admin TU"]) --> T1["Data Master"]
    AT --> T2["Manajemen Akun"]
    AT --> T3["Kepegawaian"]
    AT --> T4["Administrasi Akademik"]
    AT --> T5["Keuangan"]
    AT --> T6["Surat dan Arsip"]
    AT --> T7["Inventaris"]

    T1 --> T1a["Siswa, guru, karyawan, kelas, mapel, tahun ajaran"]
    T1a --> T1b[("CRUD Data Master")]

    T2 --> T2a["Buat akun siswa / guru / karyawan"]
    T2a --> T2b["Reset password, aktif / nonaktif akun"]
    T2b --> T2c[("Simpan Users + Log Audit")]

    T3 --> T3a["Proses pengajuan cuti / izin"]
    T3 --> T3b["Rekap absensi karyawan"]
    T3 --> T3c["Penggajian dan slip gaji"]
    T3c --> T3d[("Simpan Slip Gaji")]

    T4 --> T4a["Atur jadwal pelajaran"]
    T4 --> T4b["Penempatan siswa ke kelas"]
    T4 --> T4c["Pengelolaan pindah / mutasi / lulus"]
    T4 --> T4d["Cetak raport yang sudah divalidasi"]

    T5 --> T5a["Buat tagihan SPP"]
    T5a --> T5b["Catat pembayaran"]
    T5b --> T5c[("Simpan Transaksi")]
    T5c --> T5d["Rekap tunggakan"]

    T6 --> T6a["Surat masuk / keluar, arsip digital"]
    T7 --> T7a["Data aset, peminjaman, perawatan"]
```

### 2.7 Superadmin

```mermaid
flowchart TD
    SA(["Dashboard Superadmin"]) --> P1["Manajemen Role dan Permission"]
    SA --> P2["Manajemen Semua Pengguna"]
    SA --> P3["Konfigurasi Sistem"]
    SA --> P4["Keamanan dan Audit"]
    SA --> P5["Data dan Pemeliharaan"]
    SA --> P6["Integrasi"]

    P1 --> P1a["Buat / ubah role"]
    P1a --> P1b["Atur permission per modul: lihat, tambah, ubah, hapus"]
    P1b --> P1c[("Simpan Roles + Permissions")]
    P1c --> P1d["Cache permission di-refresh"]

    P2 --> P2a["CRUD semua akun termasuk Admin TU"]
    P2a --> P2b["Impersonate / reset akses darurat"]

    P3 --> P3a["Identitas sekolah / yayasan, tahun ajaran aktif"]
    P3 --> P3b["Pengaturan notifikasi, email, penyimpanan file"]

    P4 --> P4a["Lihat Log Audit: siapa, apa, kapan"]
    P4 --> P4b["Kebijakan password, sesi, 2FA"]
    P4 --> P4c["Blokir IP / sesi mencurigakan"]

    P5 --> P5a["Backup dan restore database"]
    P5 --> P5b["Import / export data massal"]
    P5 --> P5c["Arsip tahun ajaran lama"]

    P6 --> P6a["API, SSO, Google Workspace / email gateway / WhatsApp"]
```

---

## 3. Alur Akses Data (Role, Service, Database)

### 3.1 Lapisan Akses

```mermaid
flowchart LR
    subgraph U["Pengguna"]
        direction TB
        U1["Siswa"]
        U2["Guru"]
        U3["Karyawan"]
        U4["Kepala Sekolah"]
        U5["Kepala Yayasan"]
        U6["Admin TU"]
        U7["Superadmin"]
    end

    subgraph FE["Frontend (Web / Mobile)"]
        FE1["Menu dinamis sesuai permission"]
    end

    subgraph GW["API Gateway"]
        GW1["Autentikasi: token / sesi"]
        GW2["Otorisasi RBAC + scope data"]
        GW3["Validasi input dan rate limit"]
    end

    subgraph SV["Service"]
        SV1["Akademik: materi, tugas, ujian, nilai"]
        SV2["Absensi"]
        SV3["Kepegawaian dan Cuti"]
        SV4["Keuangan: SPP, gaji"]
        SV5["Laporan"]
        SV6["Pengumuman dan Notifikasi"]
        SV7["Admin: user, role, setting"]
    end

    subgraph DB["Database dan Storage"]
        DB1[("Users, Roles, Permissions")]
        DB2[("Siswa, Guru, Karyawan")]
        DB3[("Kelas, Mapel, Jadwal")]
        DB4[("Materi, Tugas, Pengumpulan")]
        DB5[("Ujian, Soal, Jawaban, Nilai")]
        DB6[("Absensi Siswa / Guru / Karyawan")]
        DB7[("Tagihan, Pembayaran, Gaji")]
        DB8[("Pengumuman, Surat, Inventaris")]
        DB9[("Log Audit")]
        ST[["File Storage"]]
    end

    U --> FE --> GW
    GW1 --> GW2 --> GW3
    GW3 --> SV1 & SV2 & SV3 & SV4 & SV5 & SV6 & SV7

    SV1 --> DB3 & DB4 & DB5 & ST
    SV2 --> DB6
    SV3 --> DB2 & DB6
    SV4 --> DB7
    SV5 --> DB5 & DB6 & DB7 & DB2
    SV6 --> DB8
    SV7 --> DB1 & DB2 & DB3 & DB8

    SV1 & SV2 & SV3 & SV4 & SV7 -.-> DB9
```

### 3.2 Aturan Scope Data (Row-Level)

```mermaid
flowchart TD
    R(["Request data masuk"]) --> A{"Role?"}

    A -- "Siswa" --> B1["Filter: hanya data milik siswa itu + kelasnya"]
    A -- "Guru" --> B2["Filter: hanya kelas dan mapel yang diampu"]
    A -- "Karyawan" --> B3["Filter: hanya data pribadi + tugas yang ditugaskan"]
    A -- "Kepala Sekolah" --> B4["Filter: seluruh data sekolah, read dan validasi"]
    A -- "Kepala Yayasan" --> B5["Filter: seluruh data yayasan, read-only agregat"]
    A -- "Admin TU" --> B6["Filter: data master dan administrasi, tanpa konfigurasi sistem"]
    A -- "Superadmin" --> B7["Tanpa batasan data, semua aksi diaudit"]

    B1 & B2 & B3 & B4 & B5 & B6 & B7 --> C["Query ke database dengan scope"]
    C --> D{"Data ditemukan dan diizinkan?"}
    D -- "Tidak" --> E["404 / 403"]
    D -- "Ya" --> F["Kembalikan data"]
    F --> G{"Aksi tulis?"}
    G -- "Ya" --> H[("Catat ke Log Audit")]
    G -- "Tidak" --> I(["Selesai"])
    H --> I
    E --> I
```

---

## 4. Matriks Hak Akses

Keterangan: **C** Create, **R** Read, **U** Update, **D** Delete, **V** Validasi/Setujui, **-** tidak ada akses, **(own)** hanya milik sendiri, **(kelas)** hanya kelas yang diampu.

| Data / Modul | Siswa | Guru | Karyawan | Kepsek | Ketua Yayasan | Admin TU | Superadmin |
|---|---|---|---|---|---|---|---|
| Profil & akun | R U (own) | R U (own) | R U (own) | R U (own) | R U (own) | CRUD | CRUD |
| Role & permission | - | - | - | - | - | - | CRUD |
| Data master (kelas, mapel, tahun ajaran) | R | R | - | R | R | CRUD | CRUD |
| Jadwal | R | R (own) | - | R | - | CRUD | CRUD |
| Materi | R | CRUD (kelas) | - | R | - | R | CRUD |
| Tugas & pengumpulan | CR (own) | CRUD (kelas) | - | R | - | R | CRUD |
| Ujian & soal | R, jawab | CRUD (kelas) | - | R | - | R | CRUD |
| Nilai | R (own) | CRU (kelas) | - | R, V | R (agregat) | R | CRUD |
| Raport | R (own) | CRU (kelas) | - | R, V | R | R, cetak | CRUD |
| Absensi siswa | R (own), presensi | CRU (kelas) | - | R | R (agregat) | R | CRUD |
| Absensi guru / karyawan | - | CR (own) | CR (own) | R | R (agregat) | CRUD | CRUD |
| Cuti / izin | - | CR (own) | CR (own) | V | R | R, V | CRUD |
| Tagihan SPP | R (own) | - | - | R | R (agregat) | CRUD | CRUD |
| Slip gaji | - | R (own) | R (own) | R | R (agregat) | CRUD | CRUD |
| Pengumuman | R | CR (kelas) | R | CRUD | R | CRUD | CRUD |
| Surat & arsip | - | - | - | R | - | CRUD | CRUD |
| Inventaris | - | - | R (jika relevan) | R | R | CRUD | CRUD |
| Laporan & ekspor | - | R (kelas) | - | R, ekspor | R, ekspor | R, ekspor | R, ekspor |
| Konfigurasi sistem | - | - | - | - | - | - | CRUD |
| Log audit | - | - | - | - | - | - | R |
| Backup & restore | - | - | - | - | - | - | CRUD |

---

## 5. Alur Proses Lintas Role

### 5.1 Siklus Tugas dan Nilai sampai Raport

```mermaid
sequenceDiagram
    autonumber
    actor Guru
    actor Siswa
    participant Sistem as LMS (API + DB)
    actor Kepsek as Kepala Sekolah
    actor TU as Admin TU
    actor Yayasan as Kepala Yayasan

    Guru->>Sistem: Buat materi dan tugas (deadline)
    Sistem-->>Siswa: Notifikasi tugas baru
    Siswa->>Sistem: Unggah jawaban
    Sistem-->>Guru: Notifikasi pengumpulan
    Guru->>Sistem: Beri nilai dan komentar
    Sistem-->>Siswa: Notifikasi nilai
    Guru->>Sistem: Finalisasi nilai akhir + catatan raport
    Sistem->>Kepsek: Ajukan validasi raport
    alt Valid
        Kepsek->>Sistem: Setujui dan publikasikan
        Sistem-->>Siswa: Raport tersedia
        Sistem-->>TU: Raport siap dicetak
    else Perlu revisi
        Kepsek->>Sistem: Kembalikan dengan catatan
        Sistem-->>Guru: Notifikasi revisi
    end
    Kepsek->>Sistem: Ekspor laporan akademik
    Sistem-->>Yayasan: Laporan tersedia
```

### 5.2 Pengajuan Cuti Karyawan / Guru

```mermaid
flowchart LR
    A["Karyawan / Guru isi form cuti"] --> B["Status: Menunggu"]
    B --> C["Admin TU verifikasi kelengkapan"]
    C --> D{"Lengkap?"}
    D -- "Tidak" --> E["Kembalikan ke pengaju"]
    E --> A
    D -- "Ya" --> F["Kepala Sekolah setujui / tolak"]
    F --> G{"Keputusan"}
    G -- "Setuju" --> H[("Status: Disetujui, absensi ditandai cuti")]
    G -- "Tolak" --> I[("Status: Ditolak + alasan")]
    H --> J["Notifikasi ke pengaju"]
    I --> J
    H --> K["Rekap cuti masuk laporan SDM Yayasan"]
```

### 5.3 Pembuatan Akun dan Penentuan Hak Akses

```mermaid
flowchart LR
    A["Superadmin buat role + permission"] --> B["Admin TU input data master pengguna"]
    B --> C["Admin TU buat akun + pilih role"]
    C --> D["Sistem kirim kredensial awal"]
    D --> E["Pengguna login pertama"]
    E --> F["Wajib ganti password"]
    F --> G["Akses sesuai role aktif"]
    G --> H[("Log Audit")]
```

---

## 6. Fallback ASCII (Renderer-Agnostic)

### 6.1 Alur Umum

```
[Mulai]
   |
   v
[Halaman Login] <-----------------------------+
   |                                          |
   v                                          |
<Kredensial valid?> --Tidak--> [Pesan error]--+
   |
  Ya
   v
<Akun aktif?> --Tidak--> [Hubungi Admin TU] --> [Selesai]
   |
  Ya
   v
[Buat sesi/token + Log Audit]
   |
   v
[Middleware RBAC: baca role + permission]
   |
   v
<Role?>
   |-- Siswa ----------> [Dashboard Siswa] ---------+
   |-- Guru -----------> [Dashboard Guru] ----------+
   |-- Karyawan -------> [Dashboard Karyawan] ------+
   |-- Kepala Sekolah -> [Dashboard Kepsek] --------+--> [Menu sesuai role]
   |-- Kepala Yayasan -> [Dashboard Yayasan] -------+            |
   |-- Admin TU -------> [Dashboard Admin TU] ------+            v
   |-- Superadmin -----> [Dashboard Superadmin] ----+   <Akses diizinkan?>
                                                          |            |
                                                        Tidak         Ya
                                                          v            v
                                                   [403 + Audit]  [Service/API]
                                                                       |
                                                                       v
                                                                  [(Database)]
                                                                       |
                                                                       v
                                                              [Tampilkan hasil]
                                                                       |
                                                                       v
                                                                  [Logout]
```

### 6.2 Akses Data

```
 Siswa  Guru  Karyawan  Kepsek  Yayasan  Admin TU  Superadmin
   \      \      |        |       /        /         /
    +------+-----+--------+------+---------+---------+
                          |
                          v
                 [Frontend: menu dinamis]
                          |
                          v
        [API Gateway: Auth -> RBAC -> Validasi]
                          |
      +-----------+-------+--------+-----------+----------+
      v           v                v           v          v
  [Akademik]  [Absensi]  [Kepegawaian/Cuti] [Keuangan] [Laporan]
      |           |                |           |          |
      v           v                v           v          v
 (Materi,     (Absensi       (Guru,Karyawan, (Tagihan, (Agregat dari
  Tugas,       siswa/guru/    Cuti)           Gaji)     semua tabel)
  Ujian,       karyawan)
  Nilai)
                          |
                          v
                  (Log Audit: semua aksi tulis)

 Scope data:
   Siswa      -> data sendiri
   Guru       -> kelas/mapel yang diampu
   Karyawan   -> data sendiri + tugas
   Kepsek     -> seluruh sekolah (read + validasi)
   Yayasan    -> agregat (read-only)
   Admin TU   -> data master + administrasi
   Superadmin -> semua + konfigurasi (diaudit)
```

### 6.3 Siklus Tugas sampai Raport

```
Guru: buat tugas --> Siswa: kerjakan & unggah --> Guru: beri nilai
   --> Guru: finalisasi nilai akhir --> Kepsek: validasi
          |                                 |
          |                 +---------------+---------------+
          |                 v                               v
          |             [Valid]                       [Perlu revisi]
          |                 |                               |
          |      Raport dipublikasi                         |
          |        |            |                           |
          |        v            v                           |
          |   Siswa lihat   Admin TU cetak                  |
          |                                                 |
          +<------------------------------------------------+
                                  |
                      Kepsek ekspor laporan --> Yayasan
```
