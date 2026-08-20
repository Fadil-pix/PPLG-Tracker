# Cara Setup Database Bersama (Firebase Firestore)

Website ini memakai **Firebase Firestore** supaya data Tugas, Kas, dan Acara
Kalender dibagikan ke semua siswa (bukan cuma tersimpan di 1 HP/laptop).
Tema (gelap/terang) dan pilihan blok tetap disimpan lokal per-perangkat.

## 1. Buat project Firebase (gratis)

1. Buka https://console.firebase.google.com lalu login pakai akun Google.
2. Klik **Add project** → beri nama (mis. `xipplga-kelas`) → lanjutkan
   sampai selesai (boleh matikan Google Analytics, tidak perlu).

## 2. Aktifkan Firestore Database

1. Di sidebar kiri, buka **Build → Firestore Database**.
2. Klik **Create database**.
3. Pilih lokasi server, contoh: `asia-southeast2 (Jakarta)`.
4. Pilih **Start in test mode** (supaya cepat jalan dulu; nanti kita
   perketat aturan keamanannya di langkah 4).

## 3. Daftarkan Web App & ambil config

1. Klik ikon gerigi (⚙️) di sidebar kiri → **Project settings**.
2. Scroll ke bagian **Your apps** → klik ikon `</>` (Web).
3. Beri nickname app (mis. `web-kelas`) → **Register app**.
4. Firebase akan menampilkan kode `firebaseConfig` seperti ini:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "xipplga-kelas.firebaseapp.com",
     projectId: "xipplga-kelas",
     storageBucket: "xipplga-kelas.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcd1234"
   };
   ```
5. **Copy semua isinya**, lalu buka `script.js`, cari bagian:
   ```js
   const firebaseConfig = {
     apiKey: "GANTI_DENGAN_API_KEY_KAMU",
     ...
   };
   ```
   dan **ganti seluruhnya** dengan config asli dari Firebase Console kamu.
6. Simpan file, upload ulang website (lihat bagian Hosting di bawah).

## 4. Atur aturan keamanan (Firestore Rules) — PENTING

Mode "test mode" dari langkah 2 membuat database **terbuka untuk siapa saja
di internet** (bisa baca & tulis bebas) dan **otomatis kedaluwarsa dalam 30
hari** lalu terkunci total. Untuk website kelas, aturan berikut cukup aman
(siapapun yang tahu link web bisa baca/tulis data kelas, tapi tidak bisa
mengubah struktur project atau akses data project lain):

1. Di Firestore Database, klik tab **Rules**.
2. Ganti isinya dengan:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /tasks/{taskId} {
         allow read, write: if true;
       }
       match /calendarEvents/{eventId} {
         allow read, write: if true;
       }
       match /kasPayments/{monthId} {
         allow read, write: if true;
       }
     }
   }
   ```
3. Klik **Publish**.

> Catatan: `allow read, write: if true` berarti siapa pun yang membuka
> website bisa mengubah data (cocok untuk website kelas kecil yang saling
> percaya). Kalau nanti mau lebih aman (misal hanya siswa yang login yang
> boleh menulis), bisa ditambahkan Firebase Authentication — tapi itu di
> luar cakupan setup dasar ini.

## 5. Hosting (supaya bisa dibuka semua orang)

Karena `script.js` sekarang memuat Firebase dari internet (`import ...
gstatic.com`), situs **harus dibuka lewat server/hosting**, bukan dengan
klik-dua-kali file `index.html` dari folder lokal. Opsi gratis:

- **GitHub Pages** — upload folder ini ke repo GitHub, aktifkan Pages di
  Settings → Pages.
- **Netlify / Vercel** — drag & drop folder ini ke dashboard mereka.

## Struktur data di Firestore

| Collection        | Isi dokumen                                             |
|--------------------|----------------------------------------------------------|
| `tasks`            | `subject, title, deadline, type, desc, status`           |
| `calendarEvents`   | `date, title, type` (acara tambahan yang dibuat siswa)    |
| `kasPayments`       | 1 dokumen per bulan (id = `YYYY-MM`), field `paid, paidAt`|

Saat pertama kali dijalankan dan koleksi `tasks` masih kosong, situs akan
otomatis mengisi 6 contoh tugas (data awal) sekali saja.
