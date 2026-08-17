# PigeonMail — Dokumen Spesifikasi Proyek

**Tagline:** "Pesan tidak selalu harus instan."
**Versi Dokumen:** 1.0
**Status:** Draft untuk mulai pengembangan (Pre-Development Spec)
**Terinspirasi dari konsep:** aplikasi "delayed messaging via virtual courier" (bukan menyalin nama, brand, source code, atau aset dari aplikasi manapun)

---

## 1. Executive Summary

PigeonMail adalah aplikasi pesan mobile yang membalik asumsi dasar messaging modern: bukannya instan, pesan sengaja **ditunda** dan "diterbangkan" oleh merpati virtual dari lokasi pengirim ke lokasi penerima. Waktu tempuh dihitung dari jarak geografis nyata dan kecepatan merpati (dengan variasi acak dan atribut merpati), sehingga setiap pesan punya durasi perjalanan yang unik, bisa dilacak di peta, dan punya kemungkinan kecil "hilang di jalan".

Produk ini menggabungkan empat elemen: messaging, surat digital (ada rasa "menunggu"), light game (koleksi merpati dengan rarity), dan GPS/map tracking. Untuk MVP, fokus dipersempit ke: autentikasi, profil, pertemanan, kirim pesan 1-ke-1, sistem merpati sederhana, flight engine server-authoritative, peta real-time (dihitung, bukan disimpan tiap detik), inbox/outbox, dan notifikasi push.

Dokumen ini adalah spesifikasi teknis lengkap sebelum coding dimulai — mencakup arsitektur, database, keamanan, UX, dan roadmap yang dirancang agar bisa dieksekusi bertahap oleh AI coding agent (VS Code / OpenCode + model via OpenRouter) dengan risiko regresi rendah.

---

## 2. Product Vision

Menjadi aplikasi "slow messaging" paling nyaman dan menyenangkan yang membuat pengguna kembali merasakan antisipasi menerima surat — dibungkus pengalaman modern: peta interaktif, koleksi merpati, dan notifikasi yang membangun momen "pesan telah tiba".

Visi jangka panjang (di luar MVP): sistem cuaca yang memengaruhi kecepatan terbang, event musiman, breeding/upgrade merpati, group flock messaging, dan marketplace kosmetik merpati — namun tidak dikerjakan di fase awal.

## 3. Problem Statement

Messaging modern serba instan sehingga menghilangkan rasa "penantian" dan nilai emosional sebuah pesan. Tidak ada produk mainstream yang secara sengaja memperlambat pengiriman pesan sebagai fitur inti, dibungkus dengan mekanik visual dan gamifikasi ringan yang membuat penundaan itu terasa menyenangkan, bukan menyebalkan.

## 4. Target Users

- Mahasiswa/anak muda yang menyukai aplikasi dengan gimmick unik dan estetik (long-distance friends, pasangan LDR, pen-pal digital).
- Pengguna yang ingin pengalaman "menulis surat" digital tanpa tekanan balas instan.
- Niche gamer kasual yang suka koleksi (collectible pigeon) tanpa harus main game berat.
- Pengembangan awal: portofolio proyek solo developer/mahasiswa, dapat dipublikasikan sebagai APK terbatas (closed beta) sebelum ke Play Store.

## 5. Core Concept

1. Setiap pengguna punya lokasi (diambil dengan izin, bukan real-time terus-menerus).
2. Saat mengirim pesan, sistem menghitung jarak antara lokasi pengirim (saat pengiriman) dan lokasi penerima (posisi terakhir yang diketahui/dipilih).
3. Sistem menentukan merpati yang dipakai, kecepatan efektifnya, lalu menghitung `duration = distance / speed` dengan penyesuaian batas atas/bawah agar tetap masuk akal (tidak 0 detik, tidak berbulan-bulan).
4. Server membuat *flight record* yang authoritative — posisi merpati dihitung on-demand secara matematis dari `start_time + progress`, bukan disimpan tiap detik.
5. Ketika waktu tiba terlampaui, status berubah `arrived → delivered`, notifikasi terkirim, dan isi pesan baru bisa dibaca penerima.
6. Ada probabilitas kecil (default 0.2%, dapat dikonfigurasi) merpati "hilang" — ditentukan sekali oleh server saat keberangkatan, tidak bisa dimanipulasi client.

---

## 6. Feature List (MVP vs Production vs Future)

| Fitur | MVP | Production | Future |
|---|---|---|---|
| Register/Login/Logout (email+password) | ✅ | ✅ | — |
| Forgot password | ✅ | ✅ | — |
| Email verification | ✅ (Supabase bawaan) | ✅ | — |
| Profil (nama, username, foto, lokasi, status) | ✅ | ✅ | Custom cover, bio panjang |
| Cari & tambah teman | ✅ | ✅ | Import kontak |
| Terima/tolak/hapus teman, blokir | ✅ | ✅ | Mute, favorite friend |
| Kirim pesan 1-ke-1 dengan pilih merpati | ✅ | ✅ | Attachment (gambar/audio) |
| Status pesan lengkap (draft→lost) | ✅ | ✅ | — |
| Koleksi merpati (rarity sederhana: Common/Rare/Legendary) | ✅ (3 rarity saja) | ✅ (5 rarity) | Breeding, upgrade, shop |
| Flight engine server-authoritative | ✅ | ✅ | Cuaca dinamis, rute non-linear |
| Peta real-time (posisi dihitung) | ✅ | ✅ | Live weather overlay |
| Inbox / Outbox | ✅ | ✅ | Filter & search lanjutan |
| Bird Cage (riwayat & statistik) | ✅ (versi ringkas) | ✅ | Achievement system |
| Flock (daftar teman & aktivitas) | ✅ (versi list sederhana) | ✅ | Group chat / flock chat |
| Notifikasi push (7 jenis event) | ✅ | ✅ | Notifikasi berbasis lokasi |
| Settings (account, notif, privacy, lokasi, tema) | ✅ | ✅ | Multi-language penuh |
| Group messaging | ❌ | ❌ | ✅ |
| Marketplace merpati | ❌ | ❌ | ✅ |
| Voice/video | ❌ | ❌ | Mungkin tidak pernah |

---

## 7. Functional Requirements

**FR-1** Sistem harus dapat mendaftarkan pengguna baru dengan email unik dan username unik.
**FR-2** Sistem harus memvalidasi lokasi pengirim dan penerima sebelum membuat flight; jika lokasi tidak tersedia, sistem menggunakan lokasi terakhir yang tersimpan (fallback) atau menolak pengiriman dengan pesan jelas.
**FR-3** Sistem harus menghitung jarak menggunakan formula Haversine berbasis koordinat desimal (WGS84).
**FR-4** Sistem harus menghasilkan `estimated_duration` dan `arrival_timestamp` yang konsisten dan tidak berubah setelah flight dibuat (immutable setelah `preparing`).
**FR-5** Status flight harus bertransisi sesuai state machine yang telah ditentukan (lihat §17).
**FR-6** Probabilitas lost ditentukan sekali di server saat pembuatan flight, disimpan sebagai keputusan final (`is_lost` boolean + `lost_at_progress` float), tidak dihitung ulang.
**FR-7** Sistem harus mengirim push notification untuk 7 event yang telah didefinisikan.
**FR-8** Pengguna dapat menolak izin lokasi dan tetap menggunakan aplikasi dengan keterbatasan (lihat §22).
**FR-9** Sistem harus mencegah pengguna mengirim pesan ke non-teman (kecuali fitur "cari pengguna" untuk mengirim permintaan pertemanan).
**FR-10** Sistem harus membatasi jumlah pesan yang bisa "diberangkatkan" per periode waktu (rate limiting anti-spam).

## 8. Non-Functional Requirements

- **Performa:** waktu respons API < 500ms untuk operasi CRUD standar (P95), perhitungan posisi merpati harus O(1) (matematis, bukan query berat).
- **Skalabilitas:** arsitektur backend (Supabase) harus mampu menangani hingga beberapa ribu pengguna aktif pada tier gratis/starter tanpa perubahan arsitektur besar.
- **Keandalan:** flight yang sedang berjalan harus tetap valid meski server restart, app ditutup, atau device offline — karena posisi dihitung dari data statis (`start_time`, `duration`), bukan dari proses yang berjalan terus (lihat §17, §18).
- **Keamanan:** semua keputusan yang memengaruhi hasil (jarak, kecepatan, probabilitas lost) HARUS dihitung di server, tidak pernah dipercaya dari client.
- **Privasi:** lokasi presisi tinggi tidak boleh disimpan permanen dan dapat diblur; pengguna dapat menonaktifkan berbagi lokasi presisi (lihat §22).
- **Maintainability:** kode harus modular per fitur (feature-first), dengan dokumentasi dan naming convention konsisten agar mudah dilanjutkan AI coding agent.
- **Testability:** setiap fitur inti (khususnya flight engine) harus punya unit test murni tanpa dependency jaringan.
- **Portabilitas biaya:** seluruh stack harus bisa berjalan gratis/nyaris gratis untuk skala mahasiswa/hobi (< 1000 pengguna aktif).

---

## 9. Technology Stack

| Layer | Pilihan |
|---|---|
| Frontend | Flutter (Dart), Riverpod, go_router |
| Backend | Supabase (Postgres + Auth + Realtime + Storage + Edge Functions) |
| Database | PostgreSQL (Supabase-managed); PostGIS tidak diaktifkan di MVP |
| Auth | Supabase Auth (email/password + email verification) |
| Realtime | Supabase Realtime (Postgres logical replication channel) |
| Map | flutter_map + tile OpenStreetMap (MVP); opsi upgrade ke Mapbox saat production |
| Push Notification | Firebase Cloud Messaging (FCM), dipicu dari Supabase Edge Function |
| Storage | Supabase Storage (bucket `avatars`, `pigeon-assets`) |
| State Management | Riverpod |
| Routing | go_router |
| Networking | supabase_flutter SDK (PostgREST + Realtime client) |
| Animation | Flutter implicit animation + `AnimationController`/`Tween` untuk marker; opsional Lottie untuk animasi merpati statis |
| Testing | flutter_test, mocktail, Supabase local (Docker) untuk integration test |
| CI/CD | GitHub Actions (build & analyze), manual `flutter build apk` untuk rilis awal |
| Deployment | APK sideload / internal testing track Google Play (bertahap) |

---

## 10. Technology Selection Reasoning

### 10.1 Backend: Supabase vs Firebase vs Custom Backend

| Kriteria | Supabase | Firebase | Custom (Node/Nest+Postgres) |
|---|---|---|---|
| Kecocokan data relasional (friendship, flight, pigeon) | Sangat baik — Postgres native, foreign key, transaksi | Kurang — Firestore NoSQL, join manual, rawan inkonsistensi | Baik, tapi effort tinggi |
| Row Level Security bawaan | Ya, native di Postgres | Firestore Rules (mirip, tapi model data beda) | Harus dibangun manual |
| Realtime | Ya (built-in) | Ya (built-in, sangat matang) | Harus dibangun manual (WebSocket) |
| Biaya untuk skala kecil | Gratis, generous free tier | Gratis, tapi Firestore read/write cepat mahal saat scale | Gratis jika self-host, tapi effort ops tinggi |
| Kemudahan dibantu AI agent | Tinggi — SQL standar, dokumentasi jelas, skema eksplisit | Sedang — NoSQL butuh pemahaman kontekstual | Tinggi secara kode, tapi effort setup besar |
| Server-side logic (flight engine) | Edge Functions (Deno/TypeScript) | Cloud Functions (Node) | Full control |
| Kecocokan dengan kebutuhan geolokasi & jarak | Sangat baik (SQL + bisa PostGIS kapan saja) | Kurang natural | Baik tapi manual |

**Keputusan: Supabase.** Alasan utama: model data PigeonMail sangat relasional (users–friendships–pigeons–flights–messages saling terhubung dengan foreign key dan butuh integritas transaksi), sehingga Postgres jauh lebih natural dibanding Firestore. Supabase memberi Auth, Realtime, Storage, dan Edge Functions dalam satu paket dengan SQL standar yang mudah dipahami dan dihasilkan oleh AI coding agent, serta RLS native yang cocok untuk model "privasi pesan per pengguna". Firebase unggul di kematangan push notification & realtime scaling ekstrem, tapi untuk skala MVP mahasiswa, keunggulan itu tidak signifikan dibanding kerugian model data NoSQL untuk kasus relasional ini. Custom backend memberi kontrol penuh tapi menambah beban DevOps (hosting, migrasi, auth dari nol) yang tidak realistis untuk solo developer dengan AI agent sebagai partner utama coding.

### 10.2 Database: PostgreSQL vs Firestore

Mengikuti pilihan backend, **PostgreSQL** (via Supabase) dipilih. PostGIS **tidak diaktifkan di MVP** — perhitungan jarak cukup dengan formula Haversine di Edge Function/SQL function biasa (`double precision` lat/lng), karena kebutuhan MVP hanya jarak titik-ke-titik, bukan query spasial kompleks (radius search, polygon, dsb). PostGIS dipertimbangkan untuk fase Production jika nanti dibutuhkan fitur seperti "cari teman di radius X km" atau rute non-linear.

### 10.3 Map: Google Maps vs Mapbox vs OpenStreetMap

| Kriteria | Google Maps | Mapbox | OpenStreetMap (flutter_map) |
|---|---|---|---|
| Biaya | Butuh billing account, kuota gratis terbatas | Butuh API key, free tier terbatas | Gratis, tanpa billing wajib |
| Kustomisasi visual (branding merpati) | Terbatas tanpa Cloud-based styling | Sangat fleksibel (style studio) | Fleksibel via tile custom/marker custom |
| Risiko akun mahasiswa (perlu kartu kredit) | Ya, wajib | Ya, wajib untuk tier tinggi | Tidak wajib |
| Kemudahan setup Flutter | Baik, plugin resmi tapi setup API key Android/iOS rumit | Baik, plugin resmi | Sangat mudah, murni Dart, tanpa native SDK config |

**Keputusan: flutter_map + OpenStreetMap untuk MVP.** Alasan: tidak memerlukan kartu kredit/billing account (penting untuk mahasiswa), setup lebih cepat karena murni package Dart tanpa konfigurasi native SDK yang rumit, dan kontrol penuh atas marker kustom (ikon merpati sendiri, bukan pin default) sehingga identitas visual tetap orisinal. Mapbox dicatat sebagai upgrade path untuk Production jika dibutuhkan styling peta bertema lebih kaya.

### 10.4 Push Notification

**Keputusan: Firebase Cloud Messaging (FCM).** FCM tetap dipakai walau backend utama Supabase — ini kombinasi umum dan didukung baik: Supabase Edge Function memanggil FCM HTTP v1 API saat event terjadi (pigeon departed/arriving/arrived, friend request, dsb). FCM dipilih karena gratis tanpa batas praktis untuk skala ini, dokumentasi Flutter resmi lengkap (`firebase_messaging`), dan tidak terikat pada satu platform backend.

### 10.5 State Management: Riverpod vs Bloc vs Provider

**Keputusan: Riverpod.** Riverpod dipilih karena: (1) tidak bergantung pada `BuildContext` sehingga logic bisa ditulis dan ditest terpisah dari UI — penting untuk flight engine yang butuh unit test murni; (2) boilerplate lebih sedikit dibanding Bloc sehingga lebih mudah "dibaca dan dilanjutkan" oleh AI coding agent; (3) dukungan `AsyncNotifier`/`StreamProvider` sangat cocok untuk mengonsumsi Supabase Realtime stream secara reaktif. Bloc lebih verbose dan cocok untuk tim besar dengan konvensi ketat — overkill untuk solo/AI-agent workflow. Provider dianggap sudah digantikan oleh Riverpod (dibuat oleh penulis yang sama, evolusi dari Provider).

### 10.6 Routing: go_router

Dipilih karena solusi routing resmi yang direkomendasikan Flutter team, mendukung deep link, nested navigation (untuk bottom nav + stack per tab), dan deklaratif sehingga mudah dipetakan AI agent ke `routes/app_router.dart` satu file sumber kebenaran.

---

## 11. System Architecture

```
[Flutter Mobile App]
     |  (HTTPS / REST via PostgREST, WebSocket via Realtime)
     v
[Supabase Platform]
   ├── Auth (JWT issuance, email verification, session)
   ├── PostgREST API (auto-generated REST dari skema Postgres + RLS)
   ├── Edge Functions (Deno/TS) — flight engine, lost-probability, cron jobs
   ├── Realtime (channel: flights, messages, notifications)
   ├── Storage (avatars, pigeon assets)
   └── PostgreSQL Database (source of truth)
     |
     v
[Firebase Cloud Messaging] <-- dipanggil oleh Edge Function saat event terjadi
     |
     v
[Push Notification ke device pengguna]
```

Alur ringkas: Client tidak pernah menulis langsung status flight yang sensitif (misal `is_lost`, `status`) — semua operasi kritikal lewat Edge Function yang bertindak sebagai "authoritative service layer" di atas Postgres, dilindungi RLS sebagai lapisan kedua.

## 12. Frontend Architecture

Pendekatan: **Feature-first + layer tipis (data/domain/presentation) per fitur**, bukan clean architecture penuh yang berlapis-lapis — supaya tetap mudah diikuti AI agent tanpa over-engineering untuk skala MVP.

Pola per fitur:
- `data/` — model (freezed/plain class), datasource (Supabase call).
- `domain/` (opsional, hanya untuk fitur dengan logic kompleks seperti flight) — pure logic, testable tanpa Flutter/Supabase.
- `presentation/` — screen (Widget), provider (Riverpod), komponen lokal.

Lihat §26 untuk struktur folder detail.

## 13. Backend Architecture

Backend "logic" dibagi dua:
1. **Declarative layer (RLS + SQL functions)** — aturan akses data, constraint dasar, trigger sederhana (misal `updated_at` otomatis).
2. **Imperative layer (Edge Functions, Deno/TypeScript)** — logic yang butuh keputusan sekali-jalan dan tidak boleh dipercaya ke client:
   - `create-flight` — hitung jarak, kecepatan, durasi, tentukan lost, buat record flight.
   - `resolve-flight-status` (dipanggil cron/scheduler tiap N menit) — cek flight yang sudah lewat `arrival_timestamp`, ubah status ke `arrived`, trigger notifikasi.
   - `send-friend-request`, `respond-friend-request` — validasi relasi, cegah duplikasi.
   - `notify` — helper generik pemanggil FCM.

Edge Function dipanggil via HTTPS dari Flutter menggunakan `supabase.functions.invoke()`, terautentikasi dengan JWT pengguna.

## 14. Database Architecture

Skema tunggal `public` di Postgres, dengan RLS aktif di **semua** tabel yang menyimpan data milik pengguna. Tidak ada tabel yang bisa diakses tanpa policy eksplisit (default deny). Lihat §16 untuk DDL lengkap dan §28 untuk daftar RLS policy per tabel.

---

## 15. ERD (Konseptual)

```
users (auth.users, dikelola Supabase Auth)
   │ 1:1
   ▼
profiles ──< friendships >── profiles
   │                              
   │ 1:N                          
   ▼                              
pigeons                          
   │ 1:N (dipakai dalam)          
   ▼                              
flights ──1:1── messages
   │
   │ (opsional, tidak dipakai MVP)
   ▼
flight_positions  (TIDAK dibuat — posisi dihitung matematis, lihat §17)

profiles ──1:N── devices
profiles ──1:N── notifications
profiles ──1:1── settings
profiles ──1:N── reports (opsional, moderasi)
```

Catatan: `flight_positions` sengaja **tidak** dibuat sebagai tabel tersimpan — dijelaskan alasannya di §17.3.

---

## 16. Database Schema (DDL PostgreSQL)

Konvensi: `snake_case`, primary key `uuid` (`gen_random_uuid()`), timestamp `timestamptz`, semua tabel punya `created_at`; tabel yang berubah punya `updated_at` + trigger.

```sql
-- Ekstensi
create extension if not exists "pgcrypto";

-- =========================
-- profiles (1:1 dengan auth.users)
-- =========================
create table profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  username text not null unique,
  display_name text not null,
  avatar_url text,
  status_message text,
  last_latitude double precision,
  last_longitude double precision,
  location_updated_at timestamptz,
  location_sharing_enabled boolean not null default true,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
create index idx_profiles_username on profiles (lower(username));

-- =========================
-- friendships (undirected, direpresentasikan directional + status)
-- =========================
create type friendship_status as enum ('pending', 'accepted', 'blocked');

create table friendships (
  id uuid primary key default gen_random_uuid(),
  requester_id uuid not null references profiles(id) on delete cascade,
  addressee_id uuid not null references profiles(id) on delete cascade,
  status friendship_status not null default 'pending',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint chk_no_self_friend check (requester_id <> addressee_id),
  constraint uq_friend_pair unique (requester_id, addressee_id)
);
create index idx_friendships_addressee on friendships (addressee_id, status);
create index idx_friendships_requester on friendships (requester_id, status);

-- =========================
-- pigeons (koleksi milik user)
-- =========================
create type pigeon_rarity as enum ('common', 'uncommon', 'rare', 'epic', 'legendary');
create type pigeon_status as enum ('available', 'flying', 'resting', 'lost');

create table pigeons (
  id uuid primary key default gen_random_uuid(),
  owner_id uuid not null references profiles(id) on delete cascade,
  name text not null,
  species text not null default 'Merpati Pos',
  rarity pigeon_rarity not null default 'common',
  base_speed_kmh numeric(6,2) not null default 40.00,
  stamina int not null default 100,
  health int not null default 100,
  total_flights int not null default 0,
  successful_flights int not null default 0,
  failed_flights int not null default 0,
  status pigeon_status not null default 'available',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
create index idx_pigeons_owner on pigeons (owner_id, status);

-- =========================
-- messages (isi pesan; flight = pembawa)
-- =========================
create type message_status as enum (
  'draft','queued','preparing','flying','arrived',
  'delivered','read','failed','lost','cancelled'
);

create table messages (
  id uuid primary key default gen_random_uuid(),
  sender_id uuid not null references profiles(id) on delete cascade,
  recipient_id uuid not null references profiles(id) on delete cascade,
  body text not null check (char_length(body) between 1 and 2000),
  status message_status not null default 'draft',
  read_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint chk_no_self_message check (sender_id <> recipient_id)
);
create index idx_messages_recipient on messages (recipient_id, status, created_at desc);
create index idx_messages_sender on messages (sender_id, created_at desc);

-- =========================
-- flights (satu flight = satu pengiriman satu pesan dengan satu merpati)
-- =========================
create type flight_status as enum (
  'preparing','flying','arrived','delivered','failed','lost','cancelled'
);

create table flights (
  id uuid primary key default gen_random_uuid(),
  message_id uuid not null unique references messages(id) on delete cascade,
  pigeon_id uuid not null references pigeons(id) on delete restrict,
  sender_id uuid not null references profiles(id) on delete cascade,
  recipient_id uuid not null references profiles(id) on delete cascade,
  origin_latitude double precision not null,
  origin_longitude double precision not null,
  destination_latitude double precision not null,
  destination_longitude double precision not null,
  distance_km numeric(10,3) not null,
  actual_speed_kmh numeric(6,2) not null,
  departure_at timestamptz not null default now(),
  estimated_duration_seconds int not null,
  arrival_at timestamptz not null,
  status flight_status not null default 'preparing',
  is_lost boolean not null default false,
  lost_at_progress numeric(4,3), -- 0.000 - 1.000, hanya terisi jika is_lost = true
  resolved_at timestamptz, -- kapan status final ditetapkan oleh resolver
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint chk_duration_positive check (estimated_duration_seconds > 0)
);
create index idx_flights_sender on flights (sender_id, status);
create index idx_flights_recipient on flights (recipient_id, status);
create index idx_flights_arrival on flights (arrival_at) where status in ('flying','preparing');

-- Catatan: TIDAK ADA tabel flight_positions — lihat §17.3

-- =========================
-- notifications
-- =========================
create type notification_type as enum (
  'friend_request','friend_accepted','pigeon_departed',
  'pigeon_arriving','pigeon_arrived','message_delivered','pigeon_lost'
);

create table notifications (
  id uuid primary key default gen_random_uuid(),
  recipient_id uuid not null references profiles(id) on delete cascade,
  type notification_type not null,
  payload jsonb not null default '{}',
  is_read boolean not null default false,
  created_at timestamptz not null default now()
);
create index idx_notifications_recipient on notifications (recipient_id, is_read, created_at desc);

-- =========================
-- devices (untuk FCM token)
-- =========================
create table devices (
  id uuid primary key default gen_random_uuid(),
  owner_id uuid not null references profiles(id) on delete cascade,
  fcm_token text not null,
  platform text not null default 'android',
  last_active_at timestamptz not null default now(),
  created_at timestamptz not null default now(),
  constraint uq_device_token unique (fcm_token)
);
create index idx_devices_owner on devices (owner_id);

-- =========================
-- settings (1:1)
-- =========================
create table settings (
  profile_id uuid primary key references profiles(id) on delete cascade,
  notifications_enabled boolean not null default true,
  location_precision text not null default 'city', -- 'precise' | 'city' | 'off'
  language text not null default 'id',
  theme text not null default 'system', -- 'light' | 'dark' | 'system'
  updated_at timestamptz not null default now()
);

-- =========================
-- reports (moderasi dasar, opsional MVP-akhir)
-- =========================
create table reports (
  id uuid primary key default gen_random_uuid(),
  reporter_id uuid not null references profiles(id) on delete cascade,
  target_user_id uuid references profiles(id) on delete cascade,
  target_message_id uuid references messages(id) on delete cascade,
  reason text not null,
  status text not null default 'open', -- 'open' | 'reviewed' | 'dismissed'
  created_at timestamptz not null default now()
);

-- =========================
-- app_config (agar probabilitas lost & parameter lain mudah dikonfigurasi)
-- =========================
create table app_config (
  key text primary key,
  value jsonb not null,
  updated_at timestamptz not null default now()
);
insert into app_config (key, value) values
  ('pigeon_lost_probability', '0.002'),
  ('min_flight_duration_seconds', '30'),
  ('max_flight_duration_seconds', '86400'),
  ('speed_variance_pct', '0.15');
```

### 16.1 Ringkasan kolom penting per tabel

| Tabel | PK | FK penting | Unique | Index tambahan |
|---|---|---|---|---|
| profiles | id (=auth.users.id) | — | username | lower(username) |
| friendships | id | requester_id, addressee_id → profiles | (requester_id, addressee_id) | addressee_id+status |
| pigeons | id | owner_id → profiles | — | owner_id+status |
| messages | id | sender_id, recipient_id → profiles | — | recipient_id+status+created_at |
| flights | id | message_id (unique) → messages; pigeon_id → pigeons; sender/recipient → profiles | message_id | arrival_at (partial, status aktif) |
| notifications | id | recipient_id → profiles | — | recipient_id+is_read |
| devices | id | owner_id → profiles | fcm_token | owner_id |
| settings | profile_id | profile_id → profiles | — | — |
| reports | id | reporter_id, target_user_id, target_message_id | — | — |
| app_config | key | — | — | — |

---

## 17. Flight Engine

### 17.1 Pendekatan: Server-Authoritative Penuh

Perhitungan posisi, jarak, kecepatan, dan probabilitas lost **100% dilakukan di server (Edge Function + Postgres)**. Client hanya menampilkan hasil. Ini wajib karena:
- Client bisa dimodifikasi (root/jailbreak, proxy request) — jika kecepatan/jarak dihitung di client, pengguna bisa memanipulasi waktu pengiriman atau menghindari "lost".
- Konsistensi: dua device (pengirim & penerima) harus melihat status & ETA yang sama persis.

### 17.2 Input & Output

**Input** (ke Edge Function `create-flight`):
- `sender_id`, `recipient_id`, `message_body`, `pigeon_id`
- `origin_latitude`, `origin_longitude` (dari GPS device saat submit, atau fallback last-known)

**Proses:**
```
1. Ambil origin dari payload (device pengirim), validasi rentang (-90..90, -180..180)
2. Ambil destination dari profiles.last_latitude/longitude milik recipient
   - Jika recipient tidak punya lokasi tersimpan → tolak dengan pesan jelas
     ("Penerima belum membagikan lokasi, tidak bisa membuat penerbangan")
3. distance_km = haversine(origin, destination)
4. Ambil pigeon milik sender (validasi kepemilikan & status = available)
5. base_speed = pigeon.base_speed_kmh (dimodifikasi rarity, lihat §17.4)
6. variance = random_server(-speed_variance_pct, +speed_variance_pct)
7. actual_speed_kmh = base_speed * (1 + variance)
8. duration_seconds = clamp(
        (distance_km / actual_speed_kmh) * 3600,
        min_flight_duration_seconds,
        max_flight_duration_seconds
     )
9. departure_at = now()
10. arrival_at = departure_at + duration_seconds
11. lost_roll = crypto_random_server() // 0.0 - 1.0
    is_lost = lost_roll < pigeon_lost_probability_config
    lost_at_progress = is_lost ? random_server(0.2, 0.9) : null
12. Insert row messages (status='queued') dan flights (status='preparing') dalam SATU transaksi
13. Set messages.status='flying', flights.status='flying'
14. Set pigeon.status='flying'
15. Trigger notifikasi 'pigeon_departed' ke sender (opsional) & simpan event
16. Return flight object ke client
```

**Output** (disimpan & dikembalikan):
`distance_km, actual_speed_kmh, estimated_duration_seconds, arrival_at, status='flying'`

### 17.3 Posisi Merpati: Dihitung, Bukan Disimpan Per Detik

**Keputusan: posisi dihitung on-demand (matematis), tidak ada tabel `flight_positions`.**

Alasan:
- Menyimpan posisi tiap detik untuk ribuan flight simultan = write amplification besar, tidak perlu secara fungsional.
- Posisi merpati bisa 100% direkonstruksi dari data statis:
  ```
  progress = clamp((now() - departure_at) / duration_seconds, 0, 1)
  current_lat = origin_lat + (destination_lat - origin_lat) * progress
  current_lng = origin_lng + (destination_lng - origin_lng) * progress
  ```
  (Interpolasi linear lat/lng cukup akurat untuk jarak yang relevan pada aplikasi ini; great-circle interpolation bisa ditambahkan di Production jika ingin visual rute melengkung mengikuti kelengkungan bumi untuk jarak sangat jauh.)
- Client (Flutter) menghitung ulang posisi ini secara lokal setiap frame animasi menggunakan `origin`, `destination`, `departure_at`, `duration_seconds` yang didapat sekali dari server — sangat murah secara komputasi dan tidak butuh polling terus-menerus.
- Untuk update status (bukan posisi visual), client cukup subscribe Supabase Realtime pada tabel `flights` (filter by `sender_id`/`recipient_id`) untuk tahu kapan status berubah (`arrived`, `lost`, dll), tanpa perlu tahu posisi presisi dari server tiap detik.

Ini adalah hybrid yang efisien: **status = server-push (realtime), posisi visual = client-computed (matematis)**.

### 17.4 Rarity Memengaruhi Kecepatan (MVP sederhana)

| Rarity | Speed multiplier | Catatan |
|---|---|---|
| Common | 1.0x | default |
| Uncommon | 1.1x | — |
| Rare | 1.25x | — |
| Epic | 1.4x | Production saja |
| Legendary | 1.6x | Production saja |

MVP hanya memakai 3 rarity pertama untuk menyederhanakan game loop awal.

### 17.5 Pseudocode Ringkas (Edge Function, TypeScript)

```ts
async function createFlight(input: CreateFlightInput, ctx: AuthContext) {
  assertOwnsPigeon(ctx.userId, input.pigeonId);
  const origin = validateCoords(input.originLat, input.originLng);
  const recipientProfile = await getProfile(input.recipientId);
  if (!recipientProfile.lastLatitude) {
    throw new AppError('RECIPIENT_LOCATION_UNAVAILABLE');
  }
  const destination = { lat: recipientProfile.lastLatitude, lng: recipientProfile.lastLongitude };

  const distanceKm = haversine(origin, destination);
  const pigeon = await getPigeon(input.pigeonId);
  const config = await getAppConfig();

  const speedMultiplier = RARITY_MULTIPLIER[pigeon.rarity];
  const variance = randomBetween(-config.speedVariancePct, config.speedVariancePct);
  const actualSpeed = pigeon.baseSpeedKmh * speedMultiplier * (1 + variance);

  let durationSeconds = (distanceKm / actualSpeed) * 3600;
  durationSeconds = clamp(durationSeconds, config.minFlightDurationSeconds, config.maxFlightDurationSeconds);

  const departureAt = new Date();
  const arrivalAt = new Date(departureAt.getTime() + durationSeconds * 1000);

  const lostRoll = cryptoRandom(); // 0..1, server-side, tidak bisa ditebak/diulang client
  const isLost = lostRoll < config.pigeonLostProbability;
  const lostAtProgress = isLost ? randomBetween(0.2, 0.9) : null;

  return await db.transaction(async (tx) => {
    const message = await tx.insertMessage({ ...input, status: 'queued' });
    const flight = await tx.insertFlight({
      messageId: message.id, pigeonId: pigeon.id,
      senderId: ctx.userId, recipientId: input.recipientId,
      originLat: origin.lat, originLng: origin.lng,
      destinationLat: destination.lat, destinationLng: destination.lng,
      distanceKm, actualSpeedKmh: actualSpeed,
      departureAt, estimatedDurationSeconds: durationSeconds, arrivalAt,
      status: 'flying', isLost, lostAtProgress,
    });
    await tx.updateMessageStatus(message.id, 'flying');
    await tx.updatePigeonStatus(pigeon.id, 'flying');
    return flight;
  });
}
```

### 17.6 Resolver (Cron / Scheduled Edge Function)

Berjalan tiap 1 menit (Supabase Scheduled Function / pg_cron):
```sql
select id from flights
where status = 'flying' and arrival_at <= now();
```
Untuk tiap flight:
- Jika `is_lost = true` dan `now() >= departure_at + duration*lost_at_progress` → set status `lost`, `pigeon.status='lost'`, `pigeon.failed_flights += 1`, kirim notifikasi `pigeon_lost` ke sender.
- Jika tidak lost dan `now() >= arrival_at` → set flight `arrived`, message `arrived`, kirim notifikasi `pigeon_arrived` ke recipient, `pigeon.status='resting'`, `pigeon.successful_flights += 1`, `pigeon.total_flights += 1`.
- Delivery (`arrived → delivered`) terjadi otomatis saat resolver set status arrived (delivery = pesan tersedia untuk dibuka), status `read` diset saat recipient membuka detail pesan (client call `mark-read` function).

Resolver bersifat **idempotent** — dijalankan pada baris dengan `status='flying'` saja sehingga aman dijalankan berulang atau terlambat (misal server sempat mati beberapa menit).

---

## 18. GPS Architecture

- Lokasi diambil **hanya saat dibutuhkan**: (a) saat pengguna membuka app pertama kali/refresh manual di Home, (b) saat akan mengirim pesan (opsional refresh), (c) saat pengguna menekan "Update Lokasi" di Profile/Settings.
- **Tidak ada background location tracking** di MVP — tidak sesuai kebutuhan (destination dihitung dari `last known location`, bukan real-time actual movement penerima).
- Lokasi disimpan sebagai `last_latitude/last_longitude` + `location_updated_at` di `profiles`, di-overwrite tiap update (tidak ada histori presisi tinggi tersimpan permanen — lihat §22 privasi).
- Precision setting pengguna (`settings.location_precision`): `precise` (lat/lng asli), `city` (dibulatkan ~2 desimal, ±1km), `off` (tidak berbagi — lihat dampaknya di §21 edge case #5).

## 19. Realtime Architecture

Client subscribe ke Supabase Realtime channel dengan filter RLS-aware:
- `flights` — filter `sender_id=eq.<uid> OR recipient_id=eq.<uid>` → update UI status/peta tanpa polling.
- `messages` — filter serupa → update badge inbox/outbox.
- `notifications` — filter `recipient_id=eq.<uid>` → update badge notifikasi & lonceng.
- `friendships` — filter `addressee_id=eq.<uid>` → update daftar friend request masuk secara langsung.

Fallback: jika Realtime terputus (network issue), client melakukan **pull-to-refresh** manual dan polling ringan (misal tiap 60 detik saat screen aktif) sebagai jaring pengaman — bukan primary mechanism.

## 20. Notification Architecture

Alur: Edge Function (create-flight / resolver / friend-request handler) → insert row `notifications` → panggil FCM HTTP v1 API dengan token dari tabel `devices` milik `recipient_id` → device menerima push. Client juga menyimpan token FCM ke tabel `devices` saat login (`upsert` by `fcm_token`).

7 jenis notifikasi (sesuai `notification_type` enum): `friend_request`, `friend_accepted`, `pigeon_departed`, `pigeon_arriving` (dikirim saat progress ≥ 90%, dicek oleh resolver yang sama), `pigeon_arrived`, `message_delivered`, `pigeon_lost`.

---

## 21. Security Architecture

### 21.1 Prinsip Umum
- Semua keputusan yang berdampak hasil (jarak, kecepatan, lost-roll) → **wajib di server**.
- Client hanya boleh: menampilkan data, mengirim intent (mis. "kirim pesan ke X pakai merpati Y"), tidak pernah mengirim hasil kalkulasi (mis. tidak pernah mengirim `distance_km` dari client — server hitung ulang).
- RLS aktif di semua tabel sebagai lapisan pertahanan kedua meskipun akses utama lewat Edge Function dengan service role.

### 21.2 Apa yang boleh di Client vs HARUS di Server

| Boleh di Client | HARUS di Server |
|---|---|
| Menampilkan posisi merpati (hasil interpolasi dari data yang sudah diverifikasi server) | Menghitung distance, speed, duration final |
| Memilih merpati mana yang dipakai (dari koleksi miliknya) | Validasi kepemilikan merpati & status available |
| Mengirim koordinat GPS device saat ini | Validasi rentang koordinat & fallback logic |
| Menampilkan status flight | Menentukan transisi status (state machine) |
| UI probabilitas/animasi "lost" | Rolling dadu probabilitas lost (crypto-random server) |
| Cache lokal untuk offline viewing | Sumber kebenaran data (Postgres) |

### 21.3 Rate Limiting & Anti-Spam
- Edge Function `create-flight`: maksimum N pesan per user per jam (misal 20), disimpan counter di `app_config`/Redis-like via Postgres (tabel `rate_limits` sederhana atau cek `count(*) from flights where sender_id=... and created_at > now()-interval '1 hour'`).
- Friend request: maksimum M request pending yang bisa dikirim per hari untuk mencegah spam pertemanan.
- Validasi panjang pesan di DB (`check` constraint) + Edge Function.

### 21.4 Secret Management
- Supabase URL & anon key boleh ada di client (by design, dilindungi RLS).
- Service role key, FCM server key **hanya** di Edge Function environment variables (Supabase Secrets), tidak pernah masuk ke repo atau client bundle.
- `.env` masuk `.gitignore`; gunakan `supabase secrets set` untuk produksi.

### 21.5 Validasi & Sanitasi
- Semua input Edge Function divalidasi skema (misal Zod) sebelum diproses.
- `message.body` disanitasi dari karakter kontrol berbahaya, dibatasi panjang di DB dan aplikasi.

---

## 22. Privacy (Location Privacy)

- **Permission:** hanya `foreground location` (izin "saat aplikasi digunakan"), **tidak meminta background location** — sesuai kebutuhan MVP (lihat §18).
- Lokasi disimpan sebagai **satu nilai terakhir** (`last_latitude/longitude`), bukan histori berkelanjutan — mengurangi risiko pelacakan pergerakan pengguna dari waktu ke waktu.
- Pengguna dapat mengatur `location_precision` di Settings: `precise`, `city` (dibulatkan, radius blur ~1km), atau `off`.
- Jika `off` atau permission ditolak: pengguna **tidak bisa menjadi penerima** flight baru (sender akan melihat pesan "X belum membagikan lokasi") sampai mereka mengaktifkan kembali; pengguna tetap bisa mengirim pesan menggunakan `last_latitude` lama miliknya sendiri jika ada, atau diminta memasukkan lokasi kota secara manual (opsional fallback non-GPS).
- Lokasi tidak pernah ditampilkan presisi tinggi ke pengguna lain — hanya dipakai untuk kalkulasi jarak di server; UI menampilkan nama kota (reverse-geocode) ke teman, bukan koordinat mentah.

---

## 23. UI/UX Specification

### 23.1 Identitas Visual
- **Nama produk:** PigeonMail. **Palet warna:** dominan warna langit senja (dusty blue `#5B7B9A`, cream parchment `#F4EFE6` sebagai warna dasar "surat", aksen tembaga/rose `#C97B63` untuk CTA) — kesan surat lama + langit, berbeda dari palet cerah messaging app umum.
- **Tipografi:** judul menggunakan serif humanis hangat (kesan surat), body menggunakan sans-serif modern untuk keterbacaan (mis. kombinasi seperti Fraunces/Literata untuk judul + Inter/Manrope untuk body — pilihan font akhir bebas asal tidak menyalin font berlisensi eksklusif kompetitor).
- **Ikon:** line-icon minimalis dengan satu ikon custom khas produk: siluet merpati bersayap terbuka sebagai logo dan pigeon marker di peta (digambar sendiri sebagai SVG/vector, bukan aset pihak lain).
- **Komponen:** card bergaya "amplop"/kertas dengan sudut membulat halus (radius 16), elevation lembut, bukan flat material default.

### 23.2 Navigation Structure

Bottom Navigation (4 tab utama):
1. **Home** (peta ringkasan + flight aktif)
2. **Inbox** (pesan masuk)
3. **Bird Cage** (koleksi merpati + riwayat + statistik)
4. **Flock** (daftar teman + aktivitas)

Profile & Settings diakses dari ikon avatar di top bar (bukan tab kelima) agar navigasi tetap ringkas.

### 23.3 Screen Hierarchy (ringkas)
```
Splash → Onboarding (first-run only) → Login/Register
   → Shell (Bottom Nav: Home / Inbox / Bird Cage / Flock)
        Home → Compose Message → Select Pigeon → Flight Tracking
        Inbox → Message Detail (→ Message Arrived state)
        Bird Cage → Pigeon Detail
        Flock → Friend Request list, Friend Detail
   → Profile (dari avatar) → Settings
```

### 23.4 Empty / Loading / Error State (standar di semua layar list)
- **Empty:** ilustrasi merpati sederhana + copy ramah ("Belum ada surat, kirim yang pertama!") + CTA.
- **Loading:** skeleton shimmer card sesuai bentuk konten (bukan spinner generik untuk list).
- **Error:** copy jelas + tombol "Coba lagi", bedakan pesan untuk offline vs server error.

### 23.5 Spesifikasi ringkas per layar (19 layar diminta)

| # | Layar | Elemen kunci |
|---|---|---|
| 1 | Splash | Logo + tagline, auto-check session |
| 2 | Onboarding | 3 slide konsep (tulis pesan → merpati terbang → pesan tiba) |
| 3 | Login | Email, password, lupa password, link ke register |
| 4 | Register | Nama, username (cek unik realtime), email, password, konfirmasi |
| 5 | Home | Peta ringkas flight aktif, tombol compose besar (FAB), ringkasan status |
| 6 | Friends (Flock) | Search bar, list teman, badge friend request |
| 7 | Friend Request | List incoming/outgoing, tombol terima/tolak |
| 8 | Compose Message | Pilih penerima (dari teman), text area, counter karakter |
| 9 | Select Pigeon | Grid koleksi merpati available, info speed/rarity, tombol pilih |
| 10 | Flight Tracking | Peta full-screen, marker origin/destination, garis rute, marker merpati bergerak, ETA countdown |
| 11 | Message Arrived | Modal/full-screen "amplop terbuka" animasi ringan saat pertama dibaca |
| 12 | Inbox | List pesan, indikator status merpati, unread dot |
| 13 | Message Detail | Isi pesan, info pengirim, riwayat penerbangan singkat |
| 14 | Pigeon Collection | Grid merpati dengan badge rarity & status |
| 15 | Pigeon Detail | Statistik lengkap, riwayat flight merpati tsb |
| 16 | Flock | List teman + status online/lokasi kota + jumlah pesan terkirim |
| 17 | Bird Cage | Tab: Riwayat pesan / Statistik / Koleksi ringkas |
| 18 | Profile | Foto, nama, username, statistik ringkas, tombol edit |
| 19 | Settings | Account, Notification, Privacy, Location, Language, Theme, Logout, Delete Account |

---

## 24. Screen-by-Screen (Interaksi Detail Kritis)

- **Compose → Select Pigeon → Flight Tracking** adalah satu alur linear (tidak bisa skip Select Pigeon) agar user selalu sadar merpati mana yang dipakai sebelum konfirmasi kirim (ada tombol "Kirim" konfirmasi final di Select Pigeon, memicu `create-flight`).
- **Flight Tracking** memakai data yang didapat sekali dari server (`origin, destination, departure_at, duration`) lalu menghitung posisi via `Ticker`/`AnimationController` lokal — hemat baterai & data, tidak polling posisi tiap detik (lihat §17.3).
- **Message Detail** memanggil `mark-read` Edge Function saat dibuka pertama kali (transisi `delivered → read`).

## 25. Navigation Structure (Detail Routing)

```
/splash
/onboarding
/auth/login
/auth/register
/auth/forgot-password
/home                (shell route)
  /home/compose
  /home/compose/select-pigeon
  /home/flight/:flightId
/inbox
  /inbox/message/:messageId
/birdcage
  /birdcage/pigeon/:pigeonId
/flock
  /flock/requests
  /flock/friend/:friendId
/profile
/profile/settings
```
Diimplementasikan dengan `go_router` `ShellRoute` untuk bottom nav + `GoRoute` nested untuk detail.

---

## 26. Project Folder Structure

```
lib/
├── main.dart
├── app/
│   ├── app.dart                 # MaterialApp.router setup
│   ├── router/app_router.dart   # semua definisi route (go_router)
│   └── theme/                   # color, typography, component theme
├── core/
│   ├── constants/                # nama tabel, enum, config default
│   ├── utils/                    # haversine (untuk preview client, non-authoritative), formatter tanggal, dsb
│   ├── errors/                   # AppException, error mapper
│   └── network/supabase_client.dart
├── features/
│   ├── auth/
│   │   ├── data/                 # datasource: SupabaseAuthDatasource
│   │   ├── domain/                # (opsional) validasi form
│   │   └── presentation/
│   │       ├── screens/          # login_screen.dart, register_screen.dart
│   │       └── providers/        # auth_provider.dart (Riverpod)
│   ├── profile/
│   ├── friends/
│   ├── messages/
│   │   ├── data/
│   │   ├── domain/                # message state machine helper (client-side display only)
│   │   └── presentation/
│   ├── pigeons/
│   ├── flights/
│   │   ├── data/                  # FlightsDatasource (call Edge Function, subscribe realtime)
│   │   ├── domain/                # flight_position_calculator.dart (PURE, unit-tested)
│   │   └── presentation/
│   │       ├── screens/flight_tracking_screen.dart
│   │       └── widgets/pigeon_map_marker.dart
│   ├── notifications/
│   └── settings/
├── shared/
│   ├── widgets/                   # button, card, envelope_card, empty_state, loading_skeleton
│   └── models/                    # shared DTO jika dipakai lintas fitur
└── l10n/                          # bahasa (id, en)

supabase/
├── migrations/                    # SQL migration files (schema §16)
├── functions/
│   ├── create-flight/index.ts
│   ├── resolve-flight-status/index.ts
│   ├── friend-request/index.ts
│   ├── mark-read/index.ts
│   └── _shared/                   # haversine.ts, config.ts, fcm.ts
└── config.toml

test/
├── unit/flights/flight_position_calculator_test.dart
├── unit/flights/haversine_test.dart
└── widget/...
```

**Penjelasan singkat:** struktur *feature-first* dipilih agar satu fitur = satu folder mandiri (mudah dipetakan AI agent ke satu task/phase tanpa menyentuh fitur lain). Layer `domain/` hanya dibuat untuk fitur dengan logic non-trivial (flight) agar bisa diuji murni tanpa Flutter/Supabase — bagian lain cukup `data/` + `presentation/` agar tidak over-engineered.

---

## 27. API Specification (Ringkas)

Sebagian besar CRUD memakai PostgREST otomatis dari Supabase (`supabase.from('table')...`) yang tunduk pada RLS. Endpoint kustom (Edge Functions):

| Endpoint | Method | Auth | Deskripsi |
|---|---|---|---|
| `/functions/v1/create-flight` | POST | JWT user | Body: `{recipientId, pigeonId, messageBody, originLat, originLng}` → return flight object |
| `/functions/v1/resolve-flight-status` | POST (cron) | Service role | Tidak dipanggil client; dijadwalkan |
| `/functions/v1/friend-request` | POST | JWT user | Body: `{action: 'send'|'accept'|'reject'|'remove'|'block', targetUserId}` |
| `/functions/v1/mark-read` | POST | JWT user | Body: `{messageId}` → set status `read` |
| `/functions/v1/register-device` | POST | JWT user | Body: `{fcmToken, platform}` → upsert devices |

Response error terstandarisasi: `{ "error": { "code": "RECIPIENT_LOCATION_UNAVAILABLE", "message": "..." } }`.

---

## 28. Database Policies / RLS (Contoh Kunci)

```sql
alter table profiles enable row level security;
create policy "profiles_select_all_authenticated"
  on profiles for select using (auth.role() = 'authenticated');
create policy "profiles_update_own"
  on profiles for update using (auth.uid() = id);

alter table pigeons enable row level security;
create policy "pigeons_select_own"
  on pigeons for select using (auth.uid() = owner_id);
create policy "pigeons_modify_own"
  on pigeons for all using (auth.uid() = owner_id) with check (auth.uid() = owner_id);

alter table messages enable row level security;
create policy "messages_select_participant"
  on messages for select
  using (auth.uid() = sender_id or auth.uid() = recipient_id);
-- insert message HANYA lewat Edge Function (service role) → tidak ada policy insert untuk client biasa

alter table flights enable row level security;
create policy "flights_select_participant"
  on flights for select
  using (auth.uid() = sender_id or auth.uid() = recipient_id);
-- update/insert flights: tidak ada policy untuk client → hanya service role (Edge Function)

alter table friendships enable row level security;
create policy "friendships_select_participant"
  on friendships for select
  using (auth.uid() = requester_id or auth.uid() = addressee_id);
create policy "friendships_insert_own_request"
  on friendships for insert with check (auth.uid() = requester_id);
create policy "friendships_update_addressee"
  on friendships for update using (auth.uid() = addressee_id);

alter table notifications enable row level security;
create policy "notifications_select_own"
  on notifications for select using (auth.uid() = recipient_id);
create policy "notifications_update_own_read"
  on notifications for update using (auth.uid() = recipient_id);

alter table devices enable row level security;
create policy "devices_owner_only"
  on devices for all using (auth.uid() = owner_id) with check (auth.uid() = owner_id);

alter table settings enable row level security;
create policy "settings_owner_only"
  on settings for all using (auth.uid() = profile_id) with check (auth.uid() = profile_id);
```

**Prinsip inti:** tabel `messages` dan `flights` **tidak punya policy INSERT/UPDATE untuk role `authenticated`** — hanya bisa ditulis oleh Edge Function yang berjalan dengan `service_role` key (bypass RLS by design, tapi logic-nya sudah divalidasi di kode function). Ini mencegah client menulis langsung status/hasil flight.

---

## 29. Error Handling

- **Format seragam** di semua Edge Function: `{ error: { code, message } }` dengan HTTP status sesuai (400 validasi, 401 auth, 403 forbidden, 404 not found, 409 conflict, 429 rate limit, 500 server).
- Client memetakan `code` ke pesan Bahasa Indonesia ramah pengguna via satu file `error_messages.dart` terpusat (memudahkan AI agent menambah kode baru tanpa menyentuh UI tiap layar).
- Semua exception tak terduga di Edge Function ditangkap, dicatat (log), dan dikembalikan sebagai `code: 'INTERNAL_ERROR'` tanpa membocorkan detail stack trace ke client.

---

## 30. Edge Cases

| # | Kasus | Penanganan |
|---|---|---|
| 1 | Pengirim offline saat submit | Request ditahan di client queue lokal, dikirim ulang saat online (retry dengan idempotency key agar tidak duplikat) |
| 2 | Penerima offline | Tidak masalah — flight tetap dibuat dari `last_latitude` tersimpan; notifikasi dikirim saat device online (FCM handle ini) |
| 3 | GPS mati | Fallback ke `last_latitude/longitude` tersimpan; jika tidak ada sama sekali, minta input kota manual atau tolak kirim dgn pesan jelas |
| 4 | GPS tidak akurat | Terima accuracy dari platform; jika accuracy > threshold (mis. 500m), tetap dipakai tapi beri indikator "lokasi kurang akurat" di UI (tidak memblokir) |
| 5 | Permission lokasi ditolak | App tetap bisa dipakai; user tidak bisa jadi *destination* baru sampai lokasi diaktifkan, bisa tetap kirim pakai lokasi lama jika ada |
| 6 | Penerima pindah lokasi saat merpati terbang | **Sengaja tidak memengaruhi flight yang sedang berjalan** — `destination` di tabel `flights` sudah snapshot immutable saat keberangkatan; ini realistis secara narasi (merpati sudah terlanjur menuju titik lama) |
| 7 | Aplikasi ditutup | Tidak masalah — flight state 100% di server, saat app dibuka lagi, posisi dihitung ulang dari `departure_at`/`duration` yang tersimpan |
| 8 | HP restart | Sama seperti #7, tidak ada state penting di client |
| 9 | Internet terputus saat tracking | Client tampilkan indikator offline, tetap animasikan posisi via kalkulasi lokal (data sudah didapat sebelumnya), sinkron ulang status saat online |
| 10 | Server error saat create-flight | Transaksi DB rollback penuh (tidak ada partial message/flight tersimpan); client tampilkan error & tombol retry |
| 11 | Duplicate message (double tap kirim) | Idempotency key per submit (UUID dibuat client, dicek di server sebelum insert) mencegah dua flight dari satu aksi |
| 12 | Duplicate flight | `flights.message_id` unique constraint mencegah lebih dari satu flight per message |
| 13 | User kirim banyak pesan cepat | Rate limiting server-side (lihat §21.3), UI menampilkan cooldown jika limit tercapai |
| 14 | Spam | Rate limit + laporan (`reports` table) + kemampuan blokir user |
| 15 | Merpati hilang | Ditentukan sekali saat create (lihat §17.5), resolver mengeksekusi status `lost` pada waktunya, pigeon status → `lost` (tidak bisa dipakai lagi di MVP; opsi "cari merpati" jadi future feature) |
| 16 | Flight terlalu lama (jarak sangat jauh) | `max_flight_duration_seconds` di `app_config` meng-cap durasi maksimum (misal 24 jam) agar tetap masuk akal untuk UX, meski jarak lintas negara |
| 17 | Jarak sangat dekat | `min_flight_duration_seconds` mencegah durasi 0/nyaris-instan (misal minimum 30 detik) supaya tetap terasa "flight", bukan instan |
| 18 | User menghapus akun | Cascade delete via FK `on delete cascade` untuk data milik sendiri (pigeons, sent messages sbg sender); untuk flight yang recipient-nya dihapus, `recipient_id` di-set ke placeholder "pengguna terhapus" atau flight diarsipkan (kebijakan: soft handling agar histori pengirim tidak korup — detail final ditentukan saat implementasi Phase Security) |
| 19 | User memblokir teman | `friendships.status='blocked'`; RLS & Edge Function `create-flight` menolak pengiriman baru antara dua user berstatus blocked di kedua arah |
| 20 | Timezone berbeda | Semua timestamp disimpan `timestamptz` (UTC) di DB; konversi ke timezone lokal device dilakukan murni di UI client saat render — tidak pernah memengaruhi kalkulasi durasi |
| 21 | Perubahan koordinat selama penerbangan | Lihat #6 — snapshot immutable by design |
| 22 | Flight yang sedang berjalan saat deploy backend baru | Karena state di DB (bukan proses in-memory), deploy/redeploy Edge Function tidak memengaruhi flight aktif; resolver cron tetap jalan berdasar data DB |

---

## 31. Testing Strategy

- **Unit test (wajib, prioritas tertinggi):** `flight_position_calculator.dart` (interpolasi posisi), `haversine.dart`, state machine transisi status pesan/flight — semua pure function, di-test tanpa Flutter widget/network.
- **Edge Function test:** test lokal dengan Supabase CLI (`supabase start`), skenario: create-flight normal, recipient tanpa lokasi, rate limit terlampaui, idempotency key duplikat.
- **Widget test:** komponen kritis (Compose form validation, Select Pigeon list, Flight Tracking marker render) menggunakan `flutter_test` + mock provider Riverpod.
- **Integration test (opsional Production):** alur end-to-end register → add friend → send message → resolver → inbox update, dijalankan terhadap Supabase local Docker.
- **Manual QA checklist** sebelum tiap rilis APK: daftar 20 edge case di §30 dicek manual minimal sekali.

---

## 32. Development Roadmap

> Setiap fase: Objective, Fitur, File dibuat, DB, Dependency, Backend, Testing, Definition of Done (DoD).

### Phase 0 — Environment Setup
- **Objective:** siapkan tooling agar AI agent bisa langsung produktif.
- **Fitur:** —
- **File:** `pubspec.yaml`, `.env.example`, `README.md`, `analysis_options.yaml`
- **DB:** buat project Supabase, catat URL & anon key
- **Dependency:** Flutter SDK, Supabase CLI, Docker (untuk local dev opsional)
- **Backend:** inisialisasi project Supabase (kosong)
- **Testing:** `flutter doctor` clean
- **DoD:** `flutter run` menampilkan halaman kosong tanpa error; Supabase project bisa diakses dari dashboard.

### Phase 1 — Project Initialization
- **Objective:** kerangka app + tema + router kosong.
- **Fitur:** shell navigasi (4 tab placeholder)
- **File:** `app/app.dart`, `app/router/app_router.dart`, `app/theme/*`
- **DB:** —
- **Dependency:** `go_router`, `flutter_riverpod`
- **Backend:** —
- **Testing:** widget test navigasi antar tab
- **DoD:** app bisa berpindah 4 tab dasar dengan tema PigeonMail (warna/font sesuai §23.1).

### Phase 2 — Authentication
- **Objective:** register/login/logout/forgot-password berjalan penuh.
- **Fitur:** Register, Login, Forgot Password, Session persistence
- **File:** `features/auth/**`
- **DB:** RLS `profiles` (§28), trigger auto-insert `profiles` row saat user baru daftar (Postgres trigger on `auth.users`)
- **Dependency:** `supabase_flutter`
- **Backend:** Supabase Auth (email/password + email confirm)
- **Testing:** widget test form validasi, manual test alur email verifikasi
- **DoD:** user baru bisa daftar, verifikasi email, login, logout, dan sesi bertahan setelah app ditutup-buka.

### Phase 3 — Profile
- **Objective:** lihat & edit profil termasuk update lokasi manual.
- **Fitur:** view/edit profil, upload avatar, update lokasi (tombol manual)
- **File:** `features/profile/**`
- **DB:** tabel `profiles` (sudah ada), bucket Storage `avatars`
- **Dependency:** `image_picker`, `geolocator`
- **Backend:** Storage policy avatar (user hanya bisa upload ke folder miliknya sendiri)
- **Testing:** unit test validasi form profil
- **DoD:** user bisa ubah nama/username/foto/lokasi dan tersimpan permanen.

### Phase 4 — Friend System
- **Objective:** cari, request, terima/tolak, hapus, blokir teman.
- **Fitur:** search user, friend request flow lengkap
- **File:** `features/friends/**`, `supabase/functions/friend-request/`
- **DB:** tabel `friendships` + RLS (§28)
- **Dependency:** —
- **Backend:** Edge Function `friend-request`
- **Testing:** Edge Function test (kirim, terima, tolak, duplikat, blokir)
- **DoD:** dua akun uji bisa berteman penuh melalui alur request → accept, dan bisa saling blokir/unfriend.

### Phase 5 — Messaging (skeleton, tanpa flight dulu)
- **Objective:** UI compose & inbox/outbox dengan data dummy status, sebelum flight engine nyata.
- **Fitur:** Compose screen, Inbox list, Outbox list (status masih manual/dummy)
- **File:** `features/messages/**`
- **DB:** tabel `messages` (tanpa flight dulu, status manual untuk UI dev)
- **Dependency:** —
- **Backend:** —
- **Testing:** widget test compose form
- **DoD:** UI compose, inbox, outbox berfungsi dengan data dummy/lokal.

### Phase 6 — Pigeon System
- **Objective:** koleksi merpati dasar.
- **Fitur:** Pigeon Collection grid, Pigeon Detail, seed 2-3 merpati awal per user baru
- **File:** `features/pigeons/**`
- **DB:** tabel `pigeons`, trigger seed default pigeon saat register
- **Dependency:** —
- **Backend:** —
- **Testing:** widget test grid & detail
- **DoD:** user baru otomatis punya minimal 1 merpati common; bisa lihat koleksi & detail.

### Phase 7 — Flight Engine (INTI)
- **Objective:** implementasi penuh §17.
- **Fitur:** create-flight, resolver cron, state machine
- **File:** `supabase/functions/create-flight/`, `supabase/functions/resolve-flight-status/`, `features/flights/domain/flight_position_calculator.dart`
- **DB:** tabel `flights`, `app_config`, index arrival_at
- **Dependency:** —
- **Backend:** Edge Functions + Scheduled Function (cron)
- **Testing:** unit test kalkulator posisi & haversine (wajib coverage tinggi), Edge Function test skenario §30
- **DoD:** kirim pesan sungguhan menghasilkan flight dengan ETA benar, status berubah otomatis via resolver tanpa aksi manual.

### Phase 8 — Map
- **Objective:** visualisasi flight di peta.
- **Fitur:** Flight Tracking screen dengan marker origin/destination/merpati bergerak + garis rute
- **File:** `features/flights/presentation/screens/flight_tracking_screen.dart`, `widgets/pigeon_map_marker.dart`
- **DB:** —
- **Dependency:** `flutter_map`, `latlong2`
- **Backend:** —
- **Testing:** widget test render marker sesuai progress mock
- **DoD:** posisi merpati di peta bergerak halus sesuai progress kalkulasi lokal, update tiap beberapa detik via `Ticker`.

### Phase 9 — Realtime
- **Objective:** update UI otomatis tanpa refresh manual.
- **Fitur:** subscribe channel `flights`, `messages`, `friendships`, `notifications`
- **File:** provider realtime per fitur (`*_realtime_provider.dart`)
- **DB:** aktifkan Realtime publication pada tabel terkait
- **Dependency:** (sudah termasuk `supabase_flutter`)
- **Backend:** konfigurasi Realtime di Supabase dashboard
- **Testing:** manual test dua device/emulator paralel
- **DoD:** perubahan status di satu device langsung tercermin di device lain tanpa reload.

### Phase 10 — Notification
- **Objective:** push notification 7 jenis event.
- **Fitur:** register device token, kirim FCM dari Edge Function
- **File:** `features/notifications/**`, `supabase/functions/_shared/fcm.ts`, `register-device` function
- **DB:** tabel `devices`, `notifications`
- **Dependency:** `firebase_messaging`, Firebase project setup
- **Backend:** integrasi FCM HTTP v1 di semua Edge Function relevan
- **Testing:** manual test tiap 7 jenis notifikasi di device fisik
- **DoD:** semua 7 event menghasilkan push notification yang diterima device.

### Phase 11 — UI Polish
- **Objective:** finalisasi visual sesuai §23.
- **Fitur:** empty/loading/error state seragam, animasi transisi halus, ilustrasi custom
- **File:** `shared/widgets/**`
- **DB:** —
- **Dependency:** opsional `lottie`
- **Backend:** —
- **Testing:** review visual manual per layar (19 layar)
- **DoD:** semua 19 layar memenuhi standar empty/loading/error dan identitas visual konsisten.

### Phase 12 — Security
- **Objective:** hardening sesuai §21.
- **Fitur:** rate limiting final, review semua RLS policy, audit secret
- **File:** —
- **DB:** review ulang seluruh policy (§28), tambah index yang kurang
- **Dependency:** —
- **Backend:** review Edge Function validasi input (Zod schema)
- **Testing:** uji manual mencoba bypass (curl langsung ke PostgREST tanpa lewat function) untuk pastikan ditolak
- **DoD:** tidak ada tabel sensitif yang bisa ditulis langsung tanpa lewat Edge Function; semua secret di environment variable.

### Phase 13 — Testing
- **Objective:** coverage & QA menyeluruh.
- **Fitur:** —
- **File:** lengkapi folder `test/`
- **DB:** —
- **Dependency:** `mocktail`
- **Backend:** —
- **Testing:** jalankan seluruh unit/widget/edge-function test, checklist 20 edge case manual
- **DoD:** seluruh test hijau; checklist edge case §30 lulus manual.

### Phase 14 — Build APK
- **Objective:** hasilkan APK yang bisa diinstal.
- **Fitur:** —
- **File:** `android/app/build.gradle` (signing config), `key.properties`
- **DB:** —
- **Dependency:** keystore rilis
- **Backend:** pastikan pakai Supabase project production (bukan local)
- **Testing:** install APK di device fisik, smoke test seluruh alur utama
- **DoD:** `flutter build apk --release` sukses, APK terinstal dan berjalan penuh di device fisik.

### Phase 15 — Production Deployment
- **Objective:** rilis terbatas (closed testing) & siap iterasi.
- **Fitur:** —
- **File:** listing metadata (deskripsi, ikon, screenshot)
- **DB:** migration terkunci/versioned, backup policy Supabase diaktifkan
- **Dependency:** akun Google Play Console (jika lanjut ke Play Store)
- **Backend:** monitoring dasar (Supabase logs/Edge Function logs)
- **Testing:** closed testing dengan beberapa pengguna nyata, kumpulkan feedback
- **DoD:** aplikasi berjalan stabil untuk kelompok penguji nyata selama minimal beberapa hari tanpa insiden kritikal.

---

## 33. MVP Scope (Ringkasan Final)

Termasuk: Auth, Profile, Friend System (tanpa group), Messaging 1-ke-1, Pigeon collection sederhana (3 rarity), Flight Engine penuh (server-authoritative), Map tracking (posisi terhitung), Inbox/Outbox, Bird Cage ringkas, Flock (list saja), Notifikasi 7 jenis, Settings dasar.

Tidak termasuk (lihat §7 Feature List kolom Future): group messaging, attachment media, breeding/upgrade merpati, marketplace, cuaca dinamis, rute non-linear/great-circle, multi-bahasa penuh.

## 34. Future Features

Group/Flock chat, attachment (gambar/voice note terlampir pada surat), sistem cuaca yang memengaruhi kecepatan & risiko lost, achievement/badge, breeding merpati, marketplace kosmetik, mode "pen-pal acak" (kirim ke stranger), widget home-screen menampilkan flight aktif.

## 35. Cost Estimation

| Komponen | Tier | Estimasi biaya (skala < 1.000 MAU) |
|---|---|---|
| Supabase | Free/Pro | $0 (Free tier) hingga ~$25/bln jika naik ke Pro saat traffic bertambah |
| Firebase Cloud Messaging | Free | $0 (tidak ada biaya untuk push notification) |
| Map tiles (OSM via flutter_map) | Free | $0 (perhatikan fair-use policy tile server publik; pertimbangkan tile provider berbayar murah jika trafik besar di Production) |
| Domain (opsional, untuk landing page) | — | ~$10-15/tahun jika diperlukan |
| Google Play Console (jika rilis publik) | One-time | $25 sekali bayar |
| **Total estimasi awal** | | **$0–$25 sekali bayar (Play Console) + $0/bln selama masih dalam batas free tier** |

## 36. Deployment Strategy

1. Development: Supabase local (Docker) + `flutter run` debug.
2. Staging: Supabase project terpisah (`pigeonmail-staging`) untuk testing Edge Function & migration sebelum ke production.
3. Production: Supabase project (`pigeonmail-prod`), migration dijalankan via `supabase db push` terversi di `supabase/migrations`.
4. Mobile release: mulai dari APK sideload untuk internal testing → Google Play **Internal Testing track** → **Closed Testing** → (opsional) rilis publik setelah stabil.
5. CI: GitHub Actions menjalankan `flutter analyze` + `flutter test` di setiap PR sebagai gate minimal sebelum merge.

---

## 37. AI Coding Agent Rules

Karena proyek dikerjakan bertahap oleh AI coding agent (VS Code / OpenCode, model via OpenRouter), berlaku aturan berikut:

1. **Satu fase = satu unit kerja.** Ikuti urutan Phase 0–15 di §32; jangan mengerjakan fitur di luar fase aktif tanpa diminta eksplisit.
2. **Jangan mengubah skema DB tanpa file migration baru.** Semua perubahan schema wajib lewat file baru di `supabase/migrations/`, tidak pernah mengedit migration lama yang sudah dijalankan.
3. **Flight engine logic hanya boleh berubah di `supabase/functions/create-flight` dan `.../resolve-flight-status`, plus counterpart pure function di `flight_position_calculator.dart`.** Tidak ada logic kalkulasi jarak/kecepatan/lost yang boleh ditambahkan di layer presentation.
4. **Setiap fungsi pure baru di `domain/` wajib disertai unit test pada PR yang sama.**
5. **Naming convention:** file `snake_case.dart`, class `PascalCase`, provider `xxxProvider` (Riverpod), Edge Function folder `kebab-case`.
6. **Setiap fitur baru wajib punya:** deskripsi singkat di komentar header file utama, TODO eksplisit untuk bagian yang sengaja belum diimplementasikan (jangan silent-skip), dan acceptance criteria dicantumkan sebagai komentar test.
7. **Tidak boleh menyalin ikon, aset, atau teks dari aplikasi manapun** — semua aset visual harus original atau dari sumber lisensi bebas dan dibuat sesuai palet §23.1.
8. **Perubahan satu fitur tidak boleh menyentuh file fitur lain** kecuali file bersama di `shared/` atau `core/` — dan itu pun harus dijelaskan alasannya di commit message.
9. **Semua secret (service role key, FCM server key) hanya lewat environment variable / Supabase Secrets**, tidak pernah dihardcode atau di-commit.
10. **Sebelum menandai sebuah Phase selesai, jalankan Definition of Done (DoD)** yang tercantum di §32 untuk fase tersebut.

## 38. Definition of Done (Global)

Sebuah fitur dianggap selesai jika: (1) memenuhi DoD spesifik fase-nya di §32, (2) tidak menimbulkan regresi pada test yang sudah ada (`flutter test` hijau penuh), (3) RLS/security review untuk tabel yang disentuh sudah dicek ulang terhadap §21 & §28, (4) tidak ada TODO kritikal tersisa tanpa catatan di README, (5) sudah diuji manual minimal sekali di device/emulator nyata.

## 39. Risks and Mitigation

| Risiko | Dampak | Mitigasi |
|---|---|---|
| Free tier Supabase terlampaui saat demo/testing ramai | App down sementara | Monitor usage dashboard, siapkan upgrade Pro tier sebelum demo penting |
| Ketergantungan pada tile server OSM publik (rate limit) | Peta gagal load saat traffic tinggi | Cache tile lokal ringan, siapkan fallback provider berbayar murah untuk Production |
| Resolver cron gagal jalan (down time) | Flight menumpuk di status `flying` melebihi `arrival_at` | Resolver idempotent & bisa dijalankan manual/backfill kapan saja tanpa efek samping (lihat §17.6) |
| Solo developer kehabisan waktu (proyek mahasiswa) | Fitur tidak selesai sesuai roadmap | Roadmap dipecah granular per fase agar bisa berhenti di fase manapun dengan produk yang tetap "utuh secara fungsional" (setiap fase punya DoD independen) |
| Kesalahan RLS policy membocorkan data pengguna lain | Pelanggaran privasi serius | Review §28 di Phase 12 (Security) sebelum rilis; test manual bypass |
| Kebingungan AI agent lintas sesi (context hilang) | Kode inkonsisten antar fase | Dokumen ini + aturan §37 + komentar header tiap file sebagai "single source of truth" yang bisa dibaca ulang tiap sesi baru |

---

## 40. Final Recommended Architecture

Arsitektur akhir yang direkomendasikan: **Flutter (Riverpod + go_router) sebagai client tipis, Supabase sebagai backend-as-a-service tunggal** (Postgres relasional untuk integritas data sosial/pesan, Edge Functions sebagai lapisan authoritative untuk flight engine, Realtime untuk sinkronisasi status, Storage untuk aset, Auth untuk identitas), dilengkapi **FCM** untuk push notification, dan **flutter_map/OSM** untuk peta bebas biaya. Prinsip arsitektural inti yang tidak boleh dilanggar sepanjang pengembangan: *server-authoritative untuk semua hasil yang berdampak game/kepercayaan (jarak, kecepatan, probabilitas lost), posisi visual dihitung matematis di client dari data statis (bukan disimpan tiap detik), dan RLS sebagai pertahanan berlapis di atas logic Edge Function.*

---

## FINAL TECH STACK

```
Frontend        : Flutter (Dart) + Riverpod + go_router
Backend         : Supabase (Auth, PostgREST, Realtime, Storage, Edge Functions)
Database        : PostgreSQL (Supabase-managed), tanpa PostGIS di MVP
Authentication  : Supabase Auth (email/password + email verification)
Realtime        : Supabase Realtime (channel: flights, messages, notifications, friendships)
Maps            : flutter_map + OpenStreetMap tiles (upgrade opsional ke Mapbox di Production)
Push Notification : Firebase Cloud Messaging (FCM HTTP v1), dipicu dari Edge Function
Storage         : Supabase Storage (bucket: avatars, pigeon-assets)
State Management: Riverpod
Routing         : go_router
Testing         : flutter_test, mocktail, Supabase CLI local testing untuk Edge Function
CI/CD           : GitHub Actions (flutter analyze + flutter test per PR)
Deployment      : APK internal → Google Play Internal/Closed Testing track
```

## RECOMMENDED BUILD ORDER

1. Environment Setup (Phase 0) — Flutter + Supabase project siap.
2. Project skeleton + tema + navigasi 4 tab kosong (Phase 1).
3. Authentication penuh (register/login/logout/forgot password + trigger auto-create profile) (Phase 2).
4. Profile (view/edit + upload avatar + update lokasi manual) (Phase 3).
5. Friend System penuh (search, request, accept/reject, remove, block) (Phase 4).
6. Messaging UI skeleton dengan data dummy (Compose/Inbox/Outbox tampil dulu tanpa flight nyata) (Phase 5).
7. Pigeon System dasar (koleksi + seed default pigeon per user baru) (Phase 6).
8. **Flight Engine inti** — bagian paling kritikal, kerjakan dengan hati-hati dan test coverage tinggi (create-flight + resolver cron + state machine) (Phase 7).
9. Hubungkan Messaging UI ke Flight Engine nyata (ganti data dummy dari langkah 6 dengan flight sungguhan).
10. Map tracking (visualisasi flight di peta dengan kalkulasi posisi lokal) (Phase 8).
11. Realtime wiring di semua fitur relevan (Phase 9).
12. Notification (FCM) untuk 7 jenis event (Phase 10).
13. UI Polish menyeluruh (empty/loading/error state, animasi, identitas visual final) (Phase 11).
14. Security hardening & review RLS/rate-limit (Phase 12).
15. Testing menyeluruh + checklist edge case manual (Phase 13).
16. Build APK release & smoke test device fisik (Phase 14).
17. Production deployment bertahap (internal → closed testing) (Phase 15).

**Catatan penting:** Flight Engine (langkah 8) adalah fondasi yang menentukan seluruh pengalaman produk — sebaiknya tidak dilanjutkan ke langkah berikutnya sampai unit test kalkulator posisi & Edge Function `create-flight` benar-benar stabil dan teruji terhadap skenario di §30.
