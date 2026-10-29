FSM CV Intan Pratama - Prototipe v37 (frontend statis)

Cara menjalankan:
1. Ekstrak zip ini.
2. Buka folder hasilnya di terminal, lalu jalankan:
     python3 -m http.server 8000
3. Buka http://localhost:8000/index.html di browser.

Catatan:
- Data disimpan di localStorage browser (kunci ip_customer_orders). Tidak ada backend.
- Halaman memakai Tailwind CDN, Google Fonts, dan Leaflet (cdnjs). Perlu koneksi internet.
- Peta OpenStreetMap membutuhkan internet.
- Login cepat: login.html?as=adrian (teknisi), as=budi, as=doni, as=slamet, atau pilih peran Admin / Pelanggan.
  Daftar teknisi di halaman login mengikuti roster aktif, jadi kru baru langsung bisa dipilih.
- QR invoice memakai qrcodejs (MIT) di assets/qrcode.min.js, lisensi di assets/qrcode.LICENSE.txt.

Perubahan v37 dibanding v36 (empat fitur lanjutan CSP/TSP; 98 cek end-to-end lulus):
- ETA kedatangan dari TSP di portal pelanggan. Saat dispatcher mengunci rute di Timeline Tim & Rute,
  solver TSP sudah menulis tspArrival per order; portal kini menampilkannya sebagai chip "Estimasi tiba
  HH:MM WIB" di kartu order dan catatan "Perkiraan tiba HH:MM WIB — hasil optimasi rute TSP" di panel
  Tim Teknisi. ETA hanya muncul pada status Dijadwalkan/Dikerjakan, dan otomatis dihapus saat penugasan
  dibatalkan (clearEta) atau order dijadwalkan ulang/di-assign batch, sehingga portal tidak pernah
  menampilkan janji kedatangan basi.
- Klaim garansi 30 hari dari portal. Order berstatus Selesai yang belum lewat 30 hari dari closedAt
  menampilkan tombol "Klaim garansi 30 hari". Sekali tap membuat order baru bertipe "Klaim Garansi",
  prioritas Darurat, durasi inspeksi 1 jam, mewarisi nama/alamat/zona/koordinat/paket, berstatus Baru,
  dan langsung masuk antrean CSP (garansiOf menunjuk order asal). Klaim ganda untuk order yang sama
  dicegah, dan kartu di antrean diberi badge "Garansi". Bila sudah diklaim, portal menampilkan
  "Klaim garansi sedang diproses dispatcher." bukan tombol lagi.
- Kalender hari libur (modul baru assets/ops-calendar.js, kunci ip_ops_calendar). Dispatcher menandai
  tanggal libur/cuti bersama lewat tombol "Libur" di Antrean & Penugasan (tanggal + keterangan, bisa
  dihapus). Tanggal tersebut hilang dari pilihan pelanggan (kartu BESOK/Lusa berubah menjadi "Libur"
  dan tidak bisa diklik; booking tanggal itu ditolak dengan pesan), dilewati solver CSP (semua tim
  dipangkas dari domain, Auto-Assign Batch dan Auto-Solver menolak), dan Timeline Tim & Rute
  menampilkan banner peringatan serta menolak mengunci rute di tanggal itu. Satu modul dipakai ketiga
  layar sehingga aturan libur konsisten.
- Pengingat servis berkala 90 hari. Validasi admin kini menyimpan nextServiceAt = closedAt + 90 hari.
  Portal menampilkan kartu pengingat servis berikutnya; bila tinggal ≤14 hari atau sudah jatuh tempo,
  judulnya berubah menjadi "Sudah waktunya servis berkala". Tombol "Pesan servis berikutnya" membuat
  order ulang (ulangDari) dengan paket/alamat lama berstatus Baru, dan kartu pengingat menandai
  "Sudah dipesan sebagai <ID>" agar tidak memesan dua kali. Order lama tanpa nextServiceAt tetap
  dihitung dari closedAt + 90 hari.
- Folder pengujian ditambah e2e_v37.py (25 cek): ETA muncul di portal setelah kunci rute dan bersih saat
  penugasan dibatalkan, klaim garansi (order Darurat 1 jam, dicegah ganda, badge antrean), pengingat
  servis jatuh tempo + pesan ulang satu tap, serta kalender libur di antrean, portal, dan rute TSP.
  Total seluruh suite: 98 cek (42 regresi + 15 P1 + 11 kru + 5 v36 + 25 v37).
- Modal antrean (libur, usulkan jam, batal penugasan) memakai latar dan kartu eksplisit agar tetap
  terbaca meski halaman antrean tidak memuat tailwind.config seperti halaman admin lain.

Perubahan v36 dibanding v35 (tiga perbaikan pra-backend; 73 cek end-to-end lulus):
- Katalog material & harga kini satu sumber: assets/material-catalog.js (window.MaterialCatalog, 6 SKU dengan
  nama, satuan, stok awal, safety stock, HPP, harga jual, kategori). Sebelumnya daftar SKU/harga terduplikasi di
  antrean-penugasan, teknisi, master-inventaris, admin-v32.js, p1.js, dan spk-live.js sehingga bisa berbeda antar
  layar (contoh nyata: satuan ELE-MOD-450V "kelistrikan" vs "pcs"). Semua layar kini membaca MaterialCatalog;
  stok berjalan tetap di localStorage ip_inventory dan seed stok/reset demo memakai MaterialCatalog.stockSeed().
- Bukti bayar pelanggan dikompres sebelum disimpan (canvas, maks 1280 px, JPEG 0.6) seperti foto laporan teknisi.
  Batas unggah naik ke 8 MB karena hasil kompresi yang disimpan, dan data yang masih >1,5 MB setelah kompresi
  ditolak dengan pesan jelas. Sebelumnya file disimpan apa adanya sebagai base64 sehingga cepat menghabiskan
  kuota localStorage ~5 MB.
- Jejak audit data induk: kunci baru ip_audit (maks 500 entri) diisi lewat CrewMaster.audit() setiap kali teknisi
  ditambah/diubah/diaktifkan/nonaktifkan atau data tim diperbarui dari Master & Inventaris. Log Audit menampilkan
  entri tersebut dengan kategori "Master Data & Kru", konteks "Master Data", dan rantai hash sendiri (GENESIS MASTER),
  sehingga perubahan master tidak lagi luput dari jejak audit.
- Folder pengujian ditambah e2e_v36.py (5 cek): katalog tersedia di seluruh layar + harga konsisten, stok awal
  master dari katalog, kompresi bukti bayar 3,5 MB → ~530 KB, entri audit tambah/nonaktif teknisi, dan tampilan
  entri master di Log Audit beserta panel inspektornya. Total seluruh suite: 73 cek (42 regresi + 15 P1 + 11 kru + 5 v36).

Perubahan v35 dibanding v34 (tambah teknisi; 68 cek end-to-end lulus):
- Modul baru assets/crew-master.js: satu sumber data teknisi & tim (localStorage ip_master_crew) dengan
  fallback roster bawaan, sehingga aplikasi tetap berjalan bila kunci belum ada. API: crew(), allCrew(),
  teams(), find(), newId(), initials(), certsOf().
- Master & Inventaris → tab "Master Armada & Sertifikasi Teknisi":
  * Tombol "Tambah Teknisi" + modal (nama, peran TU/Helper, tim, status, WhatsApp, sertifikasi, keahlian).
  * Kartu ringkas per tim (armada, wilayah, depot, TU & Helper) dengan tombol "Atur" untuk mengubah
    nama tim, kendaraan, wilayah, dan depot.
  * Tabel teknisi dengan pencarian, filter tim & status, ubah data, aktif/nonaktif, serta empty state.
  * Aturan validasi: 1 TU aktif + maksimal 2 Helper aktif per tim, nama tidak boleh ganda, nomor WhatsApp
    divalidasi. Kru yang masih menjadi PIC order aktif tidak bisa dinonaktifkan.
- Data master dipakai bersama: SPK & Invoice (kartu tim, jumlah kru, nama PIC/Helper), Antrean & Penugasan
  (roster CSP + kualifikasi sertifikasi), Timeline Tim & Rute (nama tim, armada, helper), konsol teknisi
  (identitas, rekan tim), presensi clock-in, dan halaman login.
- Kru nonaktif hilang dari pilihan login, tidak bisa membuka konsol teknisi, dan tidak dihitung pada alert
  clock-in. Reset data demo tidak menghapus roster teknisi (hanya order, stok, dan presensi).
- Folder pengujian ditambah e2e_crew.py (11 cek): roster awal, simpan teknisi baru, propagasi ke SPK,
  TU ganda ditolak, Helper ke-3 ditolak, PIC order aktif tidak bisa dinonaktifkan, perubahan bertahan
  setelah reload, kru nonaktif keluar dari login/konsol, roster bertahan saat reset demo, dan antrean
  + timeline membaca roster master. Total seluruh suite: 68 cek (42 regresi + 15 P1 + 11 kru).

Perubahan v34 dibanding v33 (revisi UI/UX 7 menu admin; 57 cek end-to-end tetap lulus):
- Ringkasan Hari: Aksi Cepat dipangkas menjadi satu tombol utama "Panggil Solver CSP Otomatis" + menu
  "Aksi lain (6)" yang bisa dibuka. Metrik duplikat (KPI strip vs laporan owner vs kartu pendapatan) dihapus,
  angka contoh statis diganti nilai live, dan footer aksi tidak lagi menutupi konten saat digulir.
- Pengajuan Pelanggan: tab memakai angka nyata, empty state menyebutkan langkah berikutnya, dan badge sidebar
  ringkas ("0"/"N") sehingga tidak lagi menembus lebar sidebar.
- Antrean & Penugasan: kartu terpilih memakai bar aksi melekat (Setujui / Usulkan jam / Tolak order) yang ikut
  tergulir, status dipindah ke header, salinan kapasitas "N/2 tim · kapasitas X jam · Y jam terpakai" dihitung
  dari data, dan panel notifikasi ringkas saat tidak ada pesan.
- Timeline Tim & Rute: label KPI yang tertukar diperbaiki, chip "Hemat" jadi netral saat nilai 0,
  dan penjelasan algoritma dipisah dari ringkasan operasional.
- SPK & Invoice: satu tombol primer per layar (Cetak Batch SPK), tombol Export dan Verifikasi jadi sekunder
  dengan badge angka live, tab memakai jumlah nyata, kartu KPI "Laporan Menunggu Verifikasi" konsisten dengan
  badge tab, empty state material, serta pencarian No. Invoice/Pelanggan + filter status bayar pada arsip.
  Tabel invoice 10 kolom dipadatkan (padding 8 px, min-width 900 px, sel satu baris) plus petunjuk geser,
  sehingga kolom AKSI tidak lagi terpotong di 1440 px.
- Master & Inventaris: tombol "Simpan tab ini" menjadi aksi utama, "Tambah Komponen / Item" jadi sekunder.
- Log Audit: filter ringkas, status pasif (bukan tombol) untuk baris yang tidak bisa ditindak, teks KPI tidak
  lagi bertumpuk, stepper tanggal tetap satu baris, "Reset filter" kini ikut mengosongkan kolom pencarian,
  dan label tanggal kembali ke "Semua tgl" saat filter tanggal dilepas. Sidebar, header, dan skala tipografi
  halaman ini disamakan dengan enam menu admin lain (label 12–14 px, bukan lagi 16 px), sehingga tampilan tidak
  lagi terasa membesar saat menu dibuka.
- Lintas halaman: badge sidebar tidak terhimpit label, helper anti-overflow tidak lagi memaksa baris aksi
  terlipat, dan tidak ada gulir horizontal pada 1440 px maupun 390 px.

Perubahan v33 dibanding v32 (P1, 57 cek end-to-end lulus):
- Mesin status terpusat (assets/p1.js): transisi sah Baru → Dijadwalkan → Dikerjakan → Review → Selesai.
  Transisi mustahil ditolak dan dicatat "ditolak" di ip_state_log; setiap perubahan status menulis audit trail.
- Pembayaran end-to-end: status belum/dp, menunggu verifikasi, piutang (umur H+), dan lunas dipisah dari status kerja.
  Bukti bayar dari portal pelanggan masuk antrean verifikasi admin; tombol Verifikasi menutup saldo dan mengirim notifikasi lunas.
- Invoice A4 siap cetak: kop CV Intan Pratama, data pelanggan/unit, rincian jasa & material, PPN 11%, instruksi transfer
  BCA/QRIS, QR verifikasi portal, dan kolom tanda tangan. Cetak via tombol print pada tabel invoice (body.ip-print-inv).
- AR aging: piutang dikelompokkan H+0, H+1–2, dan H+≥3 beserta jumlah invoice dan nominal pada halaman SPK & Invoice.
- Notifikasi proaktif pelanggan (antrean WhatsApp manual) + SLA: order Baru >15 menit, usulan jadwal kedaluwarsa 24 jam,
  usulan balik pelanggan >2 jam, dan kru belum clock-in. Panel notifikasi ada di antrean & SPK.
- Reschedule dua arah: pelanggan mengusulkan jadwal (dengan dampak & tenggat), admin melihat kartu usulan di antrean
  dan dapat Setujui & jadwalkan atau Tolak tanpa kehilangan jadwal semula.
- Privasi portal: verifikasi ID order + 4 digit terakhir nomor WhatsApp, tombol keluar sesi. Data pesanan tersembunyi
  sampai verifikasi berhasil.
- Operasional vs teknis: ringkasan bahasa operasional tampil lebih dulu; jejak CSP, constraint, dan skor ada di
  panel "Lihat detail teknis" (details) yang bisa dibuka saat dibutuhkan.
- Aksesibilitas & perangkat: label teks minimal 12 px, focus ring jelas, baris checklist teknisi ≥48 px (label penuh),
  tabel dengan petunjuk geser, dan sidebar menjadi drawer di tablet ≤1279 px.
- Teknisi desktop: dock clock-in melayang + QR untuk membuka tampilan ponsel; di ponsel dock disembunyikan.
- Booking pelanggan lebih ringkas: ringkasan melekat (sticky) di ponsel dan detail tambahan tertutup secara bawaan.

Perubahan v32 dibanding v31 (P0 prioritas, 42 cek end-to-end lulus):
- Satu sumber data: tanggal/jam header, badge navigasi, KPI, dan panel SPK membaca store yang sama. Tanggal mengikuti hari operasional, bukan tanggal tetap.
- Mode demo/live di banner admin: muat data demo (4 order contoh) atau hapus seluruh data perangkat. Panel SPK tidak lagi memakai order contoh Bpk. Dimas.
- Uang & waktu konsisten: nomor invoice INV-YYMM-NNN di semua halaman, label jasa/material/PPN/total/DP/sisa tagihan seragam, tombol validasi menampilkan grand total dan sisa, durasi pekerjaan memakai satu sumber (mis. 1,5 Jam / 45 Menit).
- Status kerja dan status bayar dipisah. "Selesai" tidak lagi berarti lunas; piutang tampil sebagai label terpisah.
- Auto-assign berbasis kejadian: order baru dan clock-in kru langsung memicu penjadwalan ulang (dapat dimatikan lewat toggle Auto-assign). Antrean pengecualian menampilkan order darurat, tanpa tim layak, dan menunggu persetujuan pelanggan.
- Visibilitas clock-in: alert admin "0 kru clock-in · N order menunggu" dengan tautan ke antrean, CTA clock-in di kartu tugas teknisi, dan penjadwalan ulang otomatis saat kru clock-in.
- Cut-off 06.00 WIB: tombol "Hari Ini" dinonaktifkan beserta alasannya; slot yang tidak tersedia (ISHOMA, jam lampau) tidak bisa dipilih.
- Aksi destruktif (batalkan penugasan, kembalikan laporan, tarik usulan, tolak order) memakai modal konfirmasi.
- Validasi form per kolom dengan pesan inline, scroll dan fokus otomatis ke kolom bermasalah; empty state sesuai konteks halaman.
- Konsol teknisi: teks salin foto konsisten, input DP berformat ribuan dengan batas total tagihan, label tombol aksi berikutnya lebih eksplisit.
- Folder pengujian/e2e_extra.py diperbarui: E6 mengonfirmasi modal pengembalian laporan sebelum status berubah.

Perubahan v31 dibanding v30 (hasil uji end-to-end, 42 cek lulus):
- Portal pelanggan menampilkan PPN sesuai invoice admin (sebelumnya Rp 0 karena membaca kolom yang salah).
- Laporan yang dikembalikan Admin bisa dikirim ulang teknisi (status kirim sebelumnya tidak direset).
- Kartu order langsung diperbarui setelah Auto-Assign Batch tanpa perlu reload halaman.
- Layout Antrean & Penugasan diperbaiki: matriks CSP, konsol solver, dan manual override kembali berada di kolom kanan (sebelumnya menjadi item grid selebar 1 kolom).
- Folder pengujian ditambah skenario e2e_extra.py (24 cek): tanggal lampau, booking besok, multi-unit, material + DP + PPN, teknisi non-PIC, tolak/kirim ulang laporan, usulan jadwal, dan pembatalan pelanggan.

Perubahan v30 dibanding v24 (hasil uji end-to-end, lihat folder pengujian/):
- Booking untuk hari ini setelah pukul 06.00 WIB ditolak (sebelumnya diterima).
- CSP tidak menempatkan pekerjaan di jeda ISHOMA 12.00-13.00.
- CSP tidak lagi memulai pekerjaan lebih awal dari jam yang dipilih pelanggan.

Perubahan v29 dibanding v24:
- Slot booking per jam: pelanggan memilih jam mulai 08:00 sampai 15:00. Jam mulai terakhir 15:00 agar layanan terpendek selesai sebelum 17.00. Admin menentukan jam final lewat CSP.
- Form ganti jadwal memakai pilihan jam yang sama (08:00–15:00).

Perubahan v28 dibanding v24:
- Label status pelanggan seragam dengan teknisi dan admin: Menunggu Penjadwalan, Terjadwal, Usulan jadwal baru, Sedang Dikerjakan, Menunggu Validasi, Selesai, Dibatalkan.
- Rentang slot Siang diperbaiki menjadi 13.00-15.00 WIB, sesuai jam mulai yang disimpan (13:00).

Perubahan v27 dibanding v24:
- Konsol teknisi: indikator sinkron membaca status jaringan asli. Saat offline, label OFFLINE; perubahan yang disimpan saat offline dihitung sebagai belum tersinkron, dan otomatis disinkronkan saat koneksi kembali.

Perubahan v26 dibanding v24:
- customer-portal.html dinonaktifkan dan diarahkan ke customer.html. Satu-satunya portal pelanggan sekarang customer.html (booking, lacak, ganti jadwal, dan pembatalan dengan data nyata).

Perubahan v25 dibanding v24:
- customer-portal.html: booking menyimpan order nyata ke store yang dibaca dispatcher.
- Lacak mencari order berdasarkan ID + nomor WhatsApp, menampilkan status dan tahap nyata.
- Ganti jadwal dan pembatalan memakai modal dengan alasan, bukan alert/confirm.
- Data contoh dihapus dari form dan kolom lacak; tahun footer diperbarui ke 2026.
- Tanggal kunjungan memakai input tanggal dengan cut-off 06.00 WIB.
