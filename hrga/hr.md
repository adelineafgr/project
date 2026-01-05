# Modul HR: Alur Pelamar ke Karyawan

## Gambaran Umum

Dokumentasi teknis untuk modul Human Resources yang mencakup proses end-to-end konversi pelamar kerja menjadi karyawan aktif dengan catatan profil lengkap.

## Cakupan

**Termasuk dalam Cakupan:**
- Permintaan karyawan baru
- Persetujuan permintaan karyawan baru
- Proses rekrutment
- Database pelamar dan karyawan
- Manajemen posting lowongan kerja
- Pengiriman email undangan dan hasil


## Aktor / Komponen

- **Pelamar**: Individu yang mengajukan lamaran kerja
- **SPV HR**: staff yang mereview dan memproses lamaran
- **Manager HR** : staff yang menyetujui dan memproses lamaran
- **Head of Support** : staff yang menyetujui permintaan karyawan dan penerimaan karyawan baru
- **User**: Melakukan permintaan karyawan dan pengambil keputusan untuk persetujuan/penolakan calon pelamar
- **Direktur**: Penyetuju permintaan karyawan

## Data & Input

**1. Form permintaan penambahan karyawan baru untuk user**
- Data 
- Pilihan divisi dan posisi
- Disediakan template Berita Acara Penambahan Karyawan
- 
**2. Form permintaan karyawan baru untuk HR**
**3. Form Berita Acara Penambahan Karyawan**
**4. Form Berita Acara PHK**
**5. Biodata singkat pelamar**
**6. Biodata lengkap pelamar**
**7. Form Mutasi/Promosi/Demosi**
**8. Assesment Mutasi/Promosi/Demosi**
**9. Form Psikotes**
**10. Form hasil wawancara**
**11. Surat keputusan karyawan**

**Data Pelamar:**
- Informasi personal (nama, detail kontak, identifikasi)
- Detail lamaran (posisi yang dilamar, tanggal pengajuan)
- Dokumen (CV, sertifikat, surat lamaran)

**Data Profil Karyawan:**
- ID Karyawan (di-generate sistem)
- Informasi personal (diambil dari data pelamar)
- Detail kepegawaian (posisi, departemen, tanggal mulai, jenis kepegawaian)
- Informasi kontak
- Kontak darurat
## Alur Proses Permintaan Karyawan Baru

```mermaid
flowchart TD
    REQ((Permintaan<br/>karyawan baru))
    REQ --> REQUEST{Permintaan karyawan}
    REQUEST --> TAMBAH[Penambahan karyawan]
    REQUEST --> GANTI[Pergantian Karyawan]

    TAMBAH --> FORM_TAMBAH[Form Penambahan Karyawan]
    GANTI --> FORM_GANTI[Form Pergantian Karyawan]

    FORM_TAMBAH --> DRAFT[Status: Draft]
    FORM_GANTI --> DRAFT

    DRAFT --> CONFIRM[Status: Confirm]
    CONFIRM --> COO[Status: COO]

    COO --> FORM_HR[Form permintaan<br/>yang diisi oleh HR]
    FORM_HR --> VERIF_REQ_HR{Verifikasi?}
    VERIF_REQ_HR --> |ACCEPT| VERIF_HR[Status: Verifikasi-HR]
    VERIF_REQ_HR --> |REJECT| DRAFT

    VERIF_HR --> VERIF_HEAD[Status: Verifikasi-Head]
    VERIF_HEAD --> VERIF_REQ_DIR{Verifikasi?}
    VERIF_REQ_DIR --> |ACCEPT| CLOSE[Status: CLOSE]
    VERIF_REQ_DIR --> |REJECT| DRAFT

    CLOSE --> HIRING((HIRING))

```

## Alur Proses Hiring Kandidat Internal

```mermaid
flowchart TD
    HIRE_INTERNAL((HIRING INTERNAL))
    HIRE_INTERNAL --> MOVE[Pilih Mutasi/Promosi/Demosi]
    MOVE --> SELECT[Pilih karyawan<br/>bisa lebih dari satu]
    SELECT --> ASSESMENT[Assesment]
    ASSESMENT --> PASSED{Passed?}
    PASSED --> |YA| SK[\Surat Keputusan<br/> dibuat oleh SPV HR\]
    PASSED --> |TIDAK| COUNT_ASSESMENT{Sudah melakukan<br/>lebih dari dua kali</br>assesment untuk posisi ini?}

    COUNT_ASSESMENT --> |YA| HIRE_EXTERNAL[Hiring Eksternal]
    COUNT_ASSESMENT --> |TIDAK| SCND_ASSES{Assesment kedua?}

    SCND_ASSES --> |YA| HIRE_INTERNAL
    SCND_ASSES --> |TIDAK| HIRE_EXTERNAL

    SK --> SK_HR[Approve SK<br/>oleh Manager HR]
    SK_HR --> SK_HEAD[Approve SK<br/> oleh Head of Support]

    SK_HEAD --> AKSES[Perubahan kantor dan akses]
```

## Alur Proses Hiring Kandidat Internal

```mermaid
flowchart TD
    HIRE_EXTERNAL((HIRING EXTERNAL))
    HIRE_EXTERNAL --> DB[DB Pelamar]
    HIRE_EXTERNAL --> WEB_ETOS[Job Opening Website ETOS]

    DB --> SCREENING[Screening]
    WEB_ETOS --> SCREENING[Screening]

    SCREENING --> PASSED_SCREENING{Passed Screening?}
    PASSED_SCREENING --> |YA| BIO[Isi form biodata lengkap]
    PASSED_SCREENING --> |TIDAK| NO[No Action]

    BIO --> PSIKOTES[Ikuti psikotes]
    PSIKOTES --> PASSED_PSIKOTES{Lolos psikotes?}
    PASSED_PSIKOTES --> |YA| INVITATION_HR[Invitation Interview HR]
    PASSED_PSIKOTES --> |TIDAK| EMAIL_REJECT[Kirim email rejection]

    INVITATION_HR --> INTERVIEW_HR[Interview HR]
    INTERVIEW_HR --> UPLOAD_HR[Upload hasil wawancara HR]
    UPLOAD_HR --> RESULT_HR[Kirim email<br/>hasil wawancara HR]
    RESULT_HR --> PASSED_HR{Passed Interview HR?}
    PASSED_HR --> |YA| INVITATION_USER[Invitation Interview User]

    INVITATION_USER --> INTERVIEW_USER[Interview User]
    INTERVIEW_USER --> UPLOAD_USER[Upload hasil<br/>wawancara user]
    UPLOAD_USER --> RESULT_USER[Kirim email hasil<br/>wawancara user]
    RESULT_USER --> PASSED_USER{Passed Interview User?}
    PASSED_USER --> |YA| SK_NEW[Surat keputusan<br/>karyawan yang dipilih]

    SK_NEW --> SENT_OFFERING[Kirim offering ke<br/>karyawan terpilih]
    SENT_OFFERING --> ACCEPT_OFFERING{Offering accepted?}
    ACCEPT_OFFERING --> |ACCEPT| JOIN[Join date]
    ACCEPT_OFFERING --> |REJECT| ALASAN[Alasan offering ditolak]
    ALASAN --> NEW_ACTION[Pilih aksi hiring eksternal]

    NEW_ACTION --> INVITATION_USER
    NEW_ACTION --> INVITATION_HR
    NEW_ACTION --> SCREENING

    JOIN --> PROFILE[Profiling karyawan]
```

## Logika Detail

1. **Pelamar mengajukan lamaran** melalui sistem
   - Data pelamar disimpan di database pelamar
   - Status lamaran diset menjadi "Diajukan"

2. **HR menerima dan mereview lamaran**
   - Staff HR mengakses data pelamar
   - Melakukan screening awal berdasarkan persyaratan minimum

3. **Keputusan screening awal**
   - Jika persyaratan tidak terpenuhi: lanjut ke alur penolakan
   - Jika persyaratan terpenuhi: teruskan ke hiring manager

4. **Hiring manager mereview lamaran**
   - Hiring manager mengevaluasi kualifikasi pelamar
   - Membuat keputusan untuk melanjutkan atau menolak

5. **Jika ditolak di tahap manapun:**
   - Update status pelamar menjadi "Ditolak"
   - Catat alasan penolakan
   - Arsipkan data lamaran

6. **Jika disetujui oleh hiring manager:**
   - Jadwalkan interview
   - Catat detail dan hasil interview

7. **Evaluasi hasil interview**
   - Jika tidak lolos: lanjut ke alur penolakan
   - Jika lolos: siapkan surat penawaran

8. **Proses surat penawaran**
   - Generate dan kirim surat penawaran
   - Tunggu respon pelamar

9. **Keputusan penerimaan penawaran**
   - Jika ditolak: lanjut ke alur penolakan
   - Jika diterima: lanjut ke pembuatan karyawan

10. **Kumpulkan data karyawan tambahan**
    - HR mengumpulkan informasi kepegawaian lengkap
    - Validasi semua field yang diperlukan untuk profil karyawan

11. **Buat profil karyawan**
    - Transfer data yang sesuai dari data pelamar
    - Input informasi spesifik kepegawaian

12. **Generate ID karyawan**
    - Sistem auto-generate identitas karyawan unik
    - Assign ID ke profil karyawan baru

13. **Update konversi status**
    - Ubah status pelamar menjadi "Dikonversi ke Karyawan"
    - Link data pelamar ke profil karyawan
    - Set status karyawan menjadi "Aktif"

14. **Selesaikan onboarding**
    - Profil karyawan aktif di sistem
    - Data pelamar tetap disimpan untuk audit trail

## Aturan Bisnis

- Setiap pelamar harus memiliki identifikasi unik (email atau nomor identitas)
- Status pelamar harus dilacak sepanjang proses: `Diajukan`, `Dalam Review`, `Interview Terjadwal`, `Sudah Interview`, `Penawaran Terkirim`, `Penawaran Diterima`, `Ditolak`, `Karyawan`
- ID karyawan harus unik dan di-generate oleh sistem
- Data pelamar tidak dapat dihapus; harus diarsipkan dengan update status
- Profil karyawan memerlukan semua field wajib sebelum aktivasi
- Satu data pelamar hanya dapat dikonversi menjadi satu profil karyawan
- Persetujuan hiring manager wajib sebelum generate surat penawaran
- Penjadwalan interview memerlukan persetujuan hiring manager terlebih dahulu
- Penerimaan penawaran diperlukan sebelum pembuatan profil karyawan
- Tanggal mulai karyawan harus ditentukan saat pembuatan profil

## Error Handling & Edge Case

**Lamaran Duplikat:**
- Sistem harus mendeteksi pengajuan pelamar duplikat berdasarkan email/ID
- Perilaku: (tidak dispesifikasikan - lihat Pertanyaan Terbuka)

**Data Karyawan Tidak Lengkap:**
- Jika field wajib hilang saat pembuatan profil, cegah aktivasi profil
- Return validation error ke staff HR

**Pelamar Mengundurkan Diri:**
- Pelamar dapat mengundurkan diri di tahap manapun sebelum penerimaan penawaran
- Update status sesuai dan arsipkan

**Surat Penawaran Kadaluarsa:**
- Penanganan surat penawaran kadaluarsa tanpa respon (tidak dispesifikasikan - lihat Pertanyaan Terbuka)

**Multiple Posisi:**
- Apakah pelamar dapat melamar untuk beberapa posisi secara bersamaan (tidak dispesifikasikan - lihat Pertanyaan Terbuka)

## Asumsi & Pertanyaan Terbuka

**Asumsi:**
- Generate ID karyawan mengikuti skema penomoran organisasi yang ada
- Sistem memiliki kontrol autentikasi dan otorisasi untuk staff HR dan hiring manager
- Mekanisme penyimpanan dan pengambilan dokumen sudah ada
- Sistem notifikasi ada untuk update status

**Pertanyaan Terbuka:**
- Apa format/pola spesifik untuk generate ID karyawan?
- Berapa lama data pelamar disimpan di sistem?
- Apakah ada batas waktu untuk respon surat penawaran?
- Apakah orang yang sama dapat melamar kembali setelah ditolak? Jika ya, setelah berapa lama?
- Field apa yang wajib vs opsional di profil karyawan?
- Apakah ada alur kerja berbeda untuk jenis atau level posisi yang berbeda?
- Bagaimana penanganan putaran interview (tunggal vs multiple)?
- Tingkat persetujuan apa yang ada selain hiring manager?
- Apakah ada integrasi dengan sistem background check eksternal?
- Apa yang terjadi pada profil karyawan parsial jika onboarding terganggu?

## Non-Goal

- Spesifikasi UI/UX detail
- Spesifikasi integrasi dengan sistem payroll atau time tracking
- Requirement pelaporan dan analytics
- Manajemen template dokumen
- Konten dan template notifikasi email
- Definisi permission dan role user
- Spesifikasi audit log
- Requirement optimasi performa