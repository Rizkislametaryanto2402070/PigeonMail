# PigeonMail — Dokumen Spesifikasi Proyek

**Tagline:** "Pesan tidak selalu harus instan."
**Versi Dokumen:** 1.1 (revisi dari v1.0 — menutup 8 celah kritis/sedang hasil review teknis)
**Status:** Siap untuk mulai pengembangan (Pre-Development Spec — final)
**Terinspirasi dari konsep:** aplikasi "delayed messaging via virtual courier" (bukan menyalin nama, brand, source code, atau aset dari aplikasi manapun)

> **Catatan perubahan dari v1.0:** dokumen ini adalah versi yang sudah diperbaiki. Perubahan signifikan ditandai dengan **[REVISI v1.1]** di tiap bagian yang berubah, supaya jelas mana yang baru vs mana yang sama seperti draft awal.

---

## 1. Executive Summary

PigeonMail adalah aplikasi pesan mobile yang membalik asumsi dasar messaging modern: bukannya instan, pesan sengaja **ditunda** dan "diterbangkan" oleh merpati virtual dari lokasi pengirim ke lokasi penerima. Waktu tempuh dihitung dari jarak geografis nyata dan kecepatan merpati (dengan variasi acak dan atribut merpati), sehingga setiap pesan punya durasi perjalanan yang unik, bisa dilacak di peta, dan punya kemungkinan kecil "hilang di jalan".

Produk ini menggabungkan empat elemen: messaging, surat digital (ada rasa "menunggu"), light game (koleksi merpati dengan rarity), dan GPS/map tracking. Untuk MVP, fokus dipersempit ke: autentikasi, profil, pertemanan, kirim pesan 1-ke-1, sistem merpati sederhana (dengan siklus pemulihan agar tidak dead-end), flight engine server-authoritative, peta real-time (dihitung, bukan disimpan tiap detik), inbox/outbox, dan notifikasi push.

**[REVISI v1.1]** Prinsip tambahan yang sekarang wajib dipegang: *"waiting" adalah janji produk yang harus dijamin oleh database, bukan hanya oleh UI* — isi pesan secara teknis tidak bisa dibaca lewat jalur apa pun sebelum flight tiba, dan lokasi presisi pengguna lain secara teknis tidak bisa dibaca lewat jalur apa pun oleh sesama user biasa. Kedua hal ini di v1.0 hanya dijamin oleh konvensi client; di v1.1 dijamin oleh RLS/skema itu sendiri.

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
5. Ketika waktu tiba terlampaui, status berubah `arrived → delivered`, notifikasi terkirim, dan **[REVISI v1.1]** isi pesan baru **secara teknis bisa diambil** oleh penerima (bukan hanya "secara UI ditampilkan" — sebelum titik ini, query apa pun ke isi pesan akan kosong/ditolak di level database).
6. Ada probabilitas kecil (default 0.2%, dapat dikonfigurasi) merpati "hilang" — ditentukan sekali oleh server saat keberangkatan, tidak bisa dimanipulasi client. **[REVISI v1.1]** Merpati yang hilang tidak permanen mati — ia pulih otomatis setelah masa pemulihan (lihat §17.7), agar pengguna baru tidak terjebak tanpa merpati yang bisa dipakai.

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
| Batalkan pesan sebelum berangkat **[BARU v1.1]** | ✅ | ✅ | — |
| Status pesan lengkap (draft→lost) | ✅ | ✅ | — |
| Koleksi merpati (rarity sederhana: Common/Rare/Legendary) | ✅ (3 rarity saja) | ✅ (5 rarity) | Breeding, upgrade, shop |
| Pemulihan merpati lost/resting otomatis **[BARU v1.1]** | ✅ | ✅ | — |
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
**FR-5** Status flight harus bertransisi sesuai state machine yang telah ditentukan (lihat §17.1).
**FR-6** Probabilitas lost ditentukan sekali di server saat pembuatan flight, disimpan sebagai keputusan final (`is_lost` boolean + `lost_at_progress` float), tidak dihitung ulang.
**FR-7** Sistem harus mengirim push notification untuk 7 event yang telah didefinisikan.
**FR-8** Pengguna dapat menolak izin lokasi dan tetap menggunakan aplikasi dengan keterbatasan (lihat §22).
**FR-9** Sistem harus mencegah pengguna mengirim pesan ke non-teman (kecuali fitur "cari pengguna" untuk mengirim permintaan pertemanan).
**FR-10** Sistem harus membatasi jumlah pesan yang bisa "diberangkatkan" per periode waktu (rate limiting anti-spam).
**FR-11 [BARU v1.1]** Sistem tidak boleh mengizinkan isi pesan (`message_contents.body`) terbaca oleh siapa pun selain sender, kecuali flight terkait berstatus `arrived`, `delivered`, atau `read` — dijamin di level RLS, bukan hanya di client.
**FR-12 [BARU v1.1]** Sistem tidak boleh mengizinkan koordinat lokasi presisi (`profile_locations.latitude/longitude`) terbaca oleh siapa pun selain pemiliknya sendiri — dijamin di level RLS. User lain hanya mendapat representasi kota terblur via RPC.
**FR-13 [BARU v1.1]** Pengguna harus selalu memiliki minimal satu merpati berstatus `available` (dijamin oleh mekanisme pemulihan otomatis di §17.7), sehingga tidak pernah benar-benar terkunci dari mengirim pesan.
**FR-14 [BARU v1.1]** Pengguna dapat membatalkan pesan selama masih berstatus `preparing` (jendela waktu singkat sebelum resmi berangkat).

## 8. Non-Functional Requirements

- **Performa:** waktu respons API < 500ms untuk operasi CRUD standar (P95), perhitungan posisi merpati harus O(1) (matematis, bukan query berat).
- **Skalabilitas:** arsitektur backend (Supabase) harus mampu menangani hingga beberapa ribu pengguna aktif pada tier gratis/starter tanpa perubahan arsitektur besar.
- **Keandalan:** flight yang sedang berjalan harus tetap valid meski server restart, app ditutup, atau device offline — karena posisi dihitung dari data statis (`start_time`, `duration`), bukan dari proses yang berjalan terus.
- **Keamanan:** semua keputusan yang memengaruhi hasil (jarak, kecepatan, probabilitas lost) HARUS dihitung di server, tidak pernah dipercaya dari client. **[REVISI v1.1]** Semua jaminan privasi (isi pesan, lokasi presisi) HARUS dipaksakan di level RLS/skema, tidak boleh hanya bergantung pada UI client yang "sopan".
- **Privasi:** lokasi presisi tinggi tidak boleh disimpan permanen dan dapat diblur; pengguna dapat menonaktifkan berbagi lokasi presisi (lihat §22).
- **Maintainability:** kode harus modular per fitur (feature-first), dengan dokumentasi dan naming convention konsisten agar mudah dilanjutkan AI coding agent.
- **Testability:** setiap fitur inti (khususnya flight engine) harus punya unit test murni tanpa dependency jaringan. **[REVISI v1.1]** Setiap RLS policy sensitif (messages, profile_locations) wajib punya test manual "bypass attempt" (lihat §31).
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
| Geocoding **[BARU v1.1]** | Nominatim (OSM, gratis, fair-use policy) dipanggil dari Edge Function `get-profile-city` (bukan langsung dari client), hasil di-cache di kolom `profiles.city_cache` + `city_cache_updated_at` supaya tidak memanggil Nominatim berulang untuk lokasi yang sama |
| Push Notification | Firebase Cloud Messaging (FCM), dipicu dari Supabase Edge Function |
| Storage | Supabase Storage (bucket `avatars`, `pigeon-assets`) |
| State Management | Riverpod |
| Routing | go_router |
| Networking | supabase_flutter SDK (PostgREST + Realtime client) |
| Animation | Flutter implicit animation + `AnimationController`/`Tween` untuk marker; opsional Lottie untuk animasi merpati statis |
| Testing | flutter_test, mocktail, Supabase local (Docker) untuk integration test |
| CI/CD | GitHub Actions (build & analyze), manual `flutter build apk` untuk rilis awal |
| Deployment | APK sideload / internal testing track Google Play (bertahap) |

*(Bagian §10 Technology Selection Reasoning tidak berubah dari v1.0 — keputusan Supabase, Postgres tanpa PostGIS, flutter_map/OSM, FCM, Riverpod, dan go_router tetap berlaku dengan alasan yang sama.)*

---

## 11. System Architecture

```
[Flutter Mobile App]
     |  (HTTPS / REST via PostgREST, WebSocket via Realtime)
     v
[Supabase Platform]
   ├── Auth (JWT issuance, email verification, session)
   ├── PostgREST API (auto-generated REST dari skema Postgres + RLS)
   ├── Edge Functions (Deno/TS) — flight engine, lost-probability, cron jobs, geocoding proxy
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

Alur ringkas: Client tidak pernah menulis langsung status flight yang sensitif (misal `is_lost`, `status`) — semua operasi kritikal lewat Edge Function yang bertindak sebagai "authoritative service layer" di atas Postgres, dilindungi RLS sebagai lapisan kedua. **[REVISI v1.1]** Untuk `message_contents` dan `profile_locations`, RLS bukan lagi "lapisan kedua" tapi lapisan **utama** yang menjamin data sensitif tidak bisa dibaca lewat jalur mana pun — termasuk kalau suatu saat ada bug di Edge Function.

## 12. Frontend Architecture

Pendekatan: **Feature-first + layer tipis (data/domain/presentation) per fitur**, bukan clean architecture penuh yang berlapis-lapis — supaya tetap mudah diikuti AI agent tanpa over-engineering untuk skala MVP. Struktur folder detail sama seperti §26.

## 13. Backend Architecture

Backend "logic" dibagi dua:
1. **Declarative layer (RLS + SQL functions)** — aturan akses data, constraint dasar, trigger sederhana (misal `updated_at` otomatis, pemulihan status pigeon).
2. **Imperative layer (Edge Functions, Deno/TypeScript)** — logic yang butuh keputusan sekali-jalan dan tidak boleh dipercaya ke client:
   - `create-flight` — hitung jarak, kecepatan, durasi, tentukan lost, buat record flight berstatus `preparing`.
   - `cancel-flight` **[BARU v1.1]** — batalkan flight selama masih `preparing`.
   - `resolve-flight-status` (dipanggil cron/scheduler tiap 1 menit) — cek flight yang sudah lewat `arrival_timestamp`, ubah status ke `arrived`/`lost`, trigger notifikasi.
   - `resolve-flight-progress` **[BARU v1.1]** (dipanggil cron/scheduler tiap 1 menit, terpisah dari resolver status) — cek flight yang progress-nya sudah ≥90% dan belum dinotifikasi, kirim `pigeon_arriving`.
   - `resolve-pigeon-recovery` **[BARU v1.1]** (dipanggil cron/scheduler tiap 5 menit) — kembalikan pigeon `resting`→`available` dan `lost`→`available` sesuai jadwal pemulihan masing-masing.
   - `send-friend-request`, `respond-friend-request` — validasi relasi, cegah duplikasi.
   - `get-profile-city` **[BARU v1.1]** — proxy ke Nominatim, hasil di-cache, tidak pernah mengembalikan koordinat mentah ke client selain pemilik profil.
   - `notify` — helper generik pemanggil FCM.

Edge Function dipanggil via HTTPS dari Flutter menggunakan `supabase.functions.invoke()`, terautentikasi dengan JWT pengguna.

## 14. Database Architecture

Skema tunggal `public` di Postgres, dengan RLS aktif di **semua** tabel yang menyimpan data milik pengguna. Tidak ada tabel yang bisa diakses tanpa policy eksplisit (default deny). **[REVISI v1.1]** Dua tabel baru (`message_contents`, `profile_locations`) sengaja dipisah dari tabel induknya (`messages`, `profiles`) supaya kolom sensitif punya RLS sendiri yang lebih ketat daripada RLS tabel induk — Postgres RLS bersifat row-level, bukan column-level, jadi pemisahan tabel adalah cara paling aman untuk membedakan "siapa boleh lihat metadata" vs "siapa boleh lihat isi sensitif dan kapan".

---

## 15. ERD (Konseptual)

```
users (auth.users, dikelola Supabase Auth)
   │ 1:1
   ▼
profiles ──< friendships >── profiles
   │
   │ 1:1
   ▼
profile_locations   [BARU v1.1 — dipisah dari profiles, RLS ketat]
   │
profiles ──1:N── pigeons
   │ 1:N (dipakai dalam)
   ▼
flights ──1:1── messages ──1:1── message_contents   [BARU v1.1 — dipisah dari messages, RLS ketat]
   │
   │ (opsional, tidak dipakai MVP)
   ▼
flight_positions  (TIDAK dibuat — posisi dihitung matematis, lihat §17.3)

profiles ──1:N── devices
profiles ──1:N── notifications
profiles ──1:1── settings
profiles ──1:N── reports (opsional, moderasi)
rate_limits   [BARU v1.1 — counter eksplisit per user per jenis aksi]
```

---

## 16. Database Schema (DDL PostgreSQL) — v1.1

Konvensi: `snake_case`, primary key `uuid` (`gen_random_uuid()`), timestamp `timestamptz`, semua tabel punya `created_at`; tabel yang berubah punya `updated_at` + trigger.

```sql
-- Ekstensi
create extension if not exists "pgcrypto";

-- =========================
-- profiles (1:1 dengan auth.users) — metadata publik saja
-- =========================
create table profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  username text not null unique,
  display_name text not null,
  avatar_url text,
  status_message text,
  city_cache text,                       -- [BARU v1.1] hasil reverse-geocode, boleh dibaca publik
  city_cache_updated_at timestamptz,      -- [BARU v1.1]
  location_sharing_enabled boolean not null default true,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
create index idx_profiles_username on profiles (lower(username));

-- =========================
-- profile_locations (1:1 dengan profiles) [BARU v1.1]
-- Dipisah dari profiles agar koordinat presisi punya RLS sendiri yang jauh lebih
-- ketat daripada metadata profil publik.
-- =========================
create table profile_locations (
  profile_id uuid primary key references profiles(id) on delete cascade,
  latitude double precision,
  longitude double precision,
  accuracy_meters numeric(8,2),
  location_updated_at timestamptz,
  updated_at timestamptz not null default now()
);

-- =========================
-- friendships (undirected, direpresentasikan directional + status)
-- =========================
create type friendship_status as enum ('pending', 'accepted', 'blocked');

create table friendships (
  id uuid primary key default gen_random_uuid(),
  requester_id uuid not null references profiles(id) on delete cascade,
  addressee_id uuid not null references profiles(id) on delete cascade,
  status friendship_status not null default 'pending',
  blocked_by uuid references profiles(id),   -- [BARU v1.1] siapa yang menekan blokir
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
  resting_until timestamptz,             -- [BARU v1.1]
  recovers_at timestamptz,               -- [BARU v1.1] kapan pigeon 'lost' pulih otomatis
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
create index idx_pigeons_owner on pigeons (owner_id, status);
create index idx_pigeons_resting on pigeons (resting_until) where status = 'resting';
create index idx_pigeons_recovery on pigeons (recovers_at) where status = 'lost';

-- =========================
-- messages (metadata pesan; isi/body dipisah ke message_contents)
-- =========================
create type message_status as enum (
  'draft','queued','preparing','flying','arrived',
  'delivered','read','failed','lost','cancelled'
);

create table messages (
  id uuid primary key default gen_random_uuid(),
  sender_id uuid not null references profiles(id) on delete cascade,
  recipient_id uuid not null references profiles(id) on delete cascade,
  status message_status not null default 'draft',
  read_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint chk_no_self_message check (sender_id <> recipient_id)
);
create index idx_messages_recipient on messages (recipient_id, status, created_at desc);
create index idx_messages_sender on messages (sender_id, created_at desc);

-- =========================
-- message_contents (1:1 dengan messages) [BARU v1.1]
-- Dipisah dari messages agar isi pesan punya RLS berbasis status flight,
-- bukan sekadar "kamu peserta pesan ini".
-- =========================
create table message_contents (
  message_id uuid primary key references messages(id) on delete cascade,
  body text not null check (char_length(body) between 1 and 2000),
  created_at timestamptz not null default now()
);

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
  departure_at timestamptz not null,          -- [REVISI v1.1] diisi saat status → 'flying', bukan saat insert
  prepared_at timestamptz not null default now(), -- [BARU v1.1] saat record dibuat (status 'preparing')
  estimated_duration_seconds int not null,
  arrival_at timestamptz,                     -- [REVISI v1.1] nullable selama masih 'preparing'
  status flight_status not null default 'preparing',
  is_lost boolean not null default false,
  lost_at_progress numeric(4,3),              -- 0.000 - 1.000, hanya terisi jika is_lost = true
  arriving_notified_at timestamptz,           -- [BARU v1.1] anti double-notify untuk pigeon_arriving
  resolved_at timestamptz,                    -- kapan status final ditetapkan oleh resolver
  cancelled_at timestamptz,                   -- [BARU v1.1]
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint chk_duration_positive check (estimated_duration_seconds > 0)
);
create index idx_flights_sender on flights (sender_id, status);
create index idx_flights_recipient on flights (recipient_id, status);
create index idx_flights_arrival on flights (arrival_at) where status in ('flying');
create index idx_flights_preparing on flights (prepared_at) where status = 'preparing';
create index idx_flights_arriving_check on flights (arrival_at) where status = 'flying' and arriving_notified_at is null;

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
  is_valid boolean not null default true,     -- [BARU v1.1] di-set false saat FCM mengembalikan error "unregistered"
  created_at timestamptz not null default now(),
  constraint uq_device_token unique (fcm_token)
);
create index idx_devices_owner on devices (owner_id) where is_valid = true;

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
-- rate_limits [BARU v1.1 — konkret, bukan opsional]
-- =========================
create table rate_limits (
  id uuid primary key default gen_random_uuid(),
  profile_id uuid not null references profiles(id) on delete cascade,
  action_type text not null,      -- 'create_flight' | 'friend_request'
  window_start timestamptz not null,
  count int not null default 1,
  constraint uq_rate_limit_window unique (profile_id, action_type, window_start)
);
create index idx_rate_limits_lookup on rate_limits (profile_id, action_type, window_start desc);

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
  ('speed_variance_pct', '0.15'),
  ('preparing_window_seconds', '8'),          -- [BARU v1.1]
  ('pigeon_resting_minutes', '15'),            -- [BARU v1.1]
  ('pigeon_lost_recovery_hours', '24'),        -- [BARU v1.1]
  ('rate_limit_create_flight_per_hour', '20'), -- [BARU v1.1]
  ('rate_limit_friend_request_per_day', '10'); -- [BARU v1.1]
```

### 16.1 Ringkasan tabel baru/berubah di v1.1

| Tabel | Perubahan |
|---|---|
| `profile_locations` | **Baru.** Menyimpan koordinat presisi, RLS hanya untuk pemilik. |
| `message_contents` | **Baru.** Menyimpan `body`, RLS bergantung status flight terkait. |
| `rate_limits` | **Baru.** Counter eksplisit, menggantikan opsi "count(*) query" yang ambigu di v1.0. |
| `pigeons` | + `resting_until`, `recovers_at` untuk siklus pemulihan otomatis. |
| `flights` | + `arriving_notified_at`, `cancelled_at`, `prepared_at`; `departure_at`/`arrival_at` jadi nullable sebelum `flying`. |
| `friendships` | + `blocked_by`. |
| `devices` | + `is_valid` untuk pembersihan token FCM mati. |
| `profiles` | `last_latitude/longitude` **dihapus** dari sini, pindah ke `profile_locations`; + `city_cache`. |

---

## 17. Flight Engine

### 17.1 State Machine Flight — konkret **[REVISI v1.1]**

```
preparing --(user cancel, dalam preparing_window_seconds)--> cancelled
preparing --(preparing_window_seconds terlampaui, resolver)--> flying
flying    --(arrival_at terlampaui & bukan is_lost, resolver)--> arrived --(otomatis)--> delivered
flying    --(waktu lost_at_progress terlampaui & is_lost=true, resolver)--> lost
delivered --(recipient membuka pesan, panggil mark-read)--> read
```

`preparing` sekarang punya arti nyata: window singkat (default 8 detik, `app_config.preparing_window_seconds`) di mana `create-flight` sudah menyimpan semua hasil kalkulasi (jarak, kecepatan, durasi, keputusan lost) tapi **belum** mengurangi ketersediaan pigeon secara final dan user masih bisa membatalkan lewat `cancel-flight`. Setelah window lewat, resolver mengubah status ke `flying` dan mengisi `departure_at`/`arrival_at` yang sesungguhnya (dihitung dari saat window berakhir, bukan saat insert), lalu mengunci pigeon ke status `flying`.

### 17.2 Pendekatan: Server-Authoritative Penuh

Perhitungan posisi, jarak, kecepatan, dan probabilitas lost **100% dilakukan di server (Edge Function + Postgres)**. Client hanya menampilkan hasil. Ini wajib karena:
- Client bisa dimodifikasi (root/jailbreak, proxy request) — jika kecepatan/jarak dihitung di client, pengguna bisa memanipulasi waktu pengiriman atau menghindari "lost".
- Konsistensi: dua device (pengirim & penerima) harus melihat status & ETA yang sama persis.

### 17.3 Input & Output (`create-flight`)

**Input:**
- `sender_id`, `recipient_id`, `message_body`, `pigeon_id`, `idempotency_key`
- `origin_latitude`, `origin_longitude` (dari GPS device saat submit, atau fallback last-known)

**Proses:**
```
1. Cek idempotency_key — jika sudah pernah diproses, kembalikan hasil sebelumnya (bukan buat baru).
2. Cek rate_limits (create_flight) — tolak dengan 429 jika limit tercapai.
3. Ambil origin dari payload, validasi rentang (-90..90, -180..180).
4. Ambil destination dari profile_locations milik recipient
   - Jika kosong → tolak: "Penerima belum membagikan lokasi, tidak bisa membuat penerbangan"
5. distance_km = haversine(origin, destination)
6. Ambil pigeon milik sender (validasi kepemilikan & status = 'available')
   - Jika tidak ada pigeon available → cek recovers_at/resting_until terdekat,
     kembalikan pesan jelas "merpati X akan tersedia dalam Y menit" (bukan error generik).
7. base_speed = pigeon.base_speed_kmh (dimodifikasi rarity, §17.5)
8. variance = random_server(-speed_variance_pct, +speed_variance_pct)
9. actual_speed_kmh = base_speed * (1 + variance)
10. duration_seconds = clamp((distance_km / actual_speed_kmh) * 3600, min, max)
11. lost_roll = crypto_random_server()
    is_lost = lost_roll < pigeon_lost_probability_config
    lost_at_progress = is_lost ? random_server(0.2, 0.9) : null
12. Insert dalam SATU transaksi:
    - messages (status='queued')
    - message_contents (body)
    - flights (status='preparing', departure_at=NULL, arrival_at=NULL,
               distance_km, actual_speed_kmh, estimated_duration_seconds,
               is_lost, lost_at_progress diisi di sini agar immutable)
13. pigeon.status tetap 'available' selama masih 'preparing' (baru dikunci 'flying'
    saat resolver mengeksekusi transisi preparing→flying, supaya cancel tidak
    perlu "mengembalikan" pigeon).
14. Return flight object (status='preparing', cancel_deadline) ke client.
```

Resolver kecil (bagian dari `resolve-flight-status`, jalan tiap ~5 detik atau memakai `pg_cron` interval pendek khusus untuk transisi ini) mengeksekusi:
```sql
select id from flights
where status = 'preparing'
  and prepared_at + (select (value#>>'{}')::int from app_config where key='preparing_window_seconds') * interval '1 second' <= now();
```
Untuk tiap baris: set `departure_at = now()`, `arrival_at = now() + estimated_duration_seconds`, `status = 'flying'`, `messages.status='flying'`, `pigeon.status='flying'`, trigger notifikasi `pigeon_departed`.

### 17.4 `cancel-flight` **[BARU v1.1]**

Input: `flight_id`. Hanya bisa dipanggil oleh `sender_id` dan hanya jika `status = 'preparing'`. Mengubah `flights.status='cancelled'`, `messages.status='cancelled'`, `cancelled_at=now()`. Pigeon tidak perlu dikembalikan karena belum pernah dikunci (lihat §17.3 poin 13).

### 17.5 Posisi Merpati: Dihitung, Bukan Disimpan Per Detik

**Keputusan: posisi dihitung on-demand (matematis), tidak ada tabel `flight_positions`.**

```
progress = clamp((now() - departure_at) / duration_seconds, 0, 1)
current_lat = origin_lat + (destination_lat - origin_lat) * progress
current_lng = origin_lng + (destination_lng - origin_lng) * progress
```

Client (Flutter) menghitung ulang posisi ini secara lokal setiap frame animasi menggunakan `origin`, `destination`, `departure_at`, `duration_seconds` yang didapat sekali dari server. Untuk update status, client subscribe Supabase Realtime pada tabel `flights`. **Hybrid: status = server-push (realtime), posisi visual = client-computed (matematis).**

### 17.6 Rarity Memengaruhi Kecepatan (MVP sederhana)

| Rarity | Speed multiplier | Catatan |
|---|---|---|
| Common | 1.0x | default |
| Uncommon | 1.1x | — |
| Rare | 1.25x | — |
| Epic | 1.4x | Production saja |
| Legendary | 1.6x | Production saja |

MVP hanya memakai 3 rarity pertama.

### 17.7 Siklus Hidup Pigeon & Pemulihan Otomatis **[BARU v1.1 — menutup celah "resting stuck" & "soft-lock"]**

```
available --(dipakai create-flight, dikunci saat preparing→flying)--> flying
flying --(flight sukses arrived)--> resting (resting_until = now() + pigeon_resting_minutes)
resting --(resting_until terlampaui, resolve-pigeon-recovery)--> available
flying --(flight is_lost dieksekusi)--> lost (recovers_at = now() + pigeon_lost_recovery_hours)
lost --(recovers_at terlampaui, resolve-pigeon-recovery)--> available (dengan stamina/health direset sebagian, mis. 50%, sebagai konsekuensi ringan tanpa mematikan gameplay)
```

**Jaminan minimal 1 pigeon available:** setiap kali `create-flight` gagal karena tidak ada pigeon `available`, Edge Function mengecek: jika **semua** pigeon milik user dalam status `lost`/`resting` DAN tidak ada satu pun yang `recovers_at`/`resting_until` dalam 5 menit ke depan, sistem memberi satu "merpati darurat" gratis berumur pendek (rarity `common`, dipakai sekali, otomatis terhapus setelah dipakai) — supaya user tidak pernah benar-benar buntu, tapi tetap terasa sebagai fallback darurat, bukan cara utama dapat merpati.

`resolve-pigeon-recovery` (cron tiap 5 menit) menjalankan dua UPDATE idempotent berbasis `resting_until <= now()` dan `recovers_at <= now()`.

### 17.8 Resolver Status (arrived/lost) — Idempotent

Berjalan tiap 1 menit:
```sql
select id from flights
where status = 'flying' and arrival_at <= now();
```
Untuk tiap flight:
- Jika `is_lost = true` dan `now() >= departure_at + duration*lost_at_progress` → status `lost`, `pigeon.status='lost'`, `pigeon.recovers_at` diisi, `pigeon.failed_flights += 1`, notifikasi `pigeon_lost`.
- Jika tidak lost dan `now() >= arrival_at` → flight `arrived`, message `arrived`, notifikasi `pigeon_arrived`, `pigeon.status='resting'`, `pigeon.resting_until` diisi, `pigeon.successful_flights += 1`, `pigeon.total_flights += 1`.
- Delivery (`arrived → delivered`) otomatis bersamaan dengan set `arrived` (delivery = pesan tersedia untuk dibuka). Status `read` diset saat recipient memanggil `mark-read`.

### 17.9 Resolver Progress (notifikasi "arriving") **[REVISI v1.1 — menutup celah query yang tidak konsisten]**

Query **terpisah**, berjalan tiap 1 menit bersamaan dengan §17.8 tapi sebagai langkah berbeda:
```sql
select id, departure_at, estimated_duration_seconds from flights
where status = 'flying'
  and arriving_notified_at is null
  and now() >= departure_at + (estimated_duration_seconds * 0.9) * interval '1 second';
```
Untuk tiap baris: kirim notifikasi `pigeon_arriving`, set `arriving_notified_at = now()` (mencegah double-notify).

---

## 18. GPS Architecture

- Lokasi diambil **hanya saat dibutuhkan**: (a) saat pengguna membuka app pertama kali/refresh manual di Home, (b) saat akan mengirim pesan (opsional refresh), (c) saat pengguna menekan "Update Lokasi" di Profile/Settings.
- **Tidak ada background location tracking** di MVP.
- **[REVISI v1.1]** Lokasi disimpan di `profile_locations` (bukan lagi kolom langsung di `profiles`), di-overwrite tiap update, tidak ada histori presisi tinggi tersimpan permanen.
- Precision setting pengguna (`settings.location_precision`): `precise`, `city` (city_cache saja yang dibagikan ke teman via `get-profile-city`), `off` (tidak berbagi).

## 19. Realtime Architecture

Client subscribe ke Supabase Realtime channel dengan filter RLS-aware:
- `flights` — filter `sender_id=eq.<uid> OR recipient_id=eq.<uid>`.
- `messages` — filter serupa (hanya metadata status, bukan `message_contents`).
- `notifications` — filter `recipient_id=eq.<uid>`.
- `friendships` — filter `addressee_id=eq.<uid>` (dan `requester_id=eq.<uid>` untuk melihat perubahan status blokir yang dilakukan lawan bicara).

Fallback: pull-to-refresh manual + polling ringan (~60 detik) sebagai jaring pengaman.

## 20. Notification Architecture

Alur: Edge Function → insert row `notifications` → panggil FCM HTTP v1 API dengan token dari tabel `devices` (`is_valid=true`) milik `recipient_id`. Jika FCM mengembalikan error "unregistered"/"invalid-registration-token", Edge Function men-set `devices.is_valid=false` untuk token tersebut **[BARU v1.1]**.

7 jenis notifikasi: `friend_request`, `friend_accepted`, `pigeon_departed`, `pigeon_arriving` (dikirim oleh `resolve-flight-progress`, §17.9), `pigeon_arrived`, `message_delivered`, `pigeon_lost`.

---

## 21. Security Architecture

### 21.1 Prinsip Umum
- Semua keputusan yang berdampak hasil (jarak, kecepatan, lost-roll) → **wajib di server**.
- **[REVISI v1.1]** Semua data yang secara naratif "belum boleh diketahui" (isi pesan sebelum tiba, koordinat presisi orang lain) → **wajib ditolak di RLS**, bukan hanya disembunyikan di UI.
- Client hanya boleh: menampilkan data, mengirim intent, tidak pernah mengirim hasil kalkulasi.

### 21.2 Boleh di Client vs HARUS di Server

| Boleh di Client | HARUS di Server |
|---|---|
| Menampilkan posisi merpati (interpolasi dari data terverifikasi) | Menghitung distance, speed, duration final |
| Memilih merpati mana yang dipakai | Validasi kepemilikan merpati & status available |
| Mengirim koordinat GPS device saat ini | Validasi rentang koordinat & fallback logic |
| Menampilkan status flight | Menentukan transisi status (state machine) |
| UI probabilitas/animasi "lost" | Rolling dadu probabilitas lost (crypto-random server) |
| Cache lokal untuk offline viewing | Sumber kebenaran data (Postgres) |
| — | **[BARU v1.1]** Menentukan kapan `message_contents.body` boleh terbaca (via RLS) |
| — | **[BARU v1.1]** Menentukan siapa yang boleh baca `profile_locations` mentah (via RLS) |

### 21.3 Rate Limiting & Anti-Spam **[REVISI v1.1 — konkret]**

- `create-flight`: maksimum **20 pesan/jam** per user (`app_config.rate_limit_create_flight_per_hour`), dicek via tabel `rate_limits` (window per jam, upsert counter, bukan `count(*)` yang lebih mahal).
- Friend request: maksimum **10 request pending baru/hari** per user (`app_config.rate_limit_friend_request_per_day`).
- Validasi panjang pesan di DB (`check` constraint pada `message_contents.body`) + Edge Function.

### 21.4 Secret Management
- Supabase URL & anon key boleh ada di client. Service role key, FCM server key **hanya** di Edge Function environment variables (Supabase Secrets).
- `.env` masuk `.gitignore`; gunakan `supabase secrets set` untuk produksi.

### 21.5 Validasi & Sanitasi
- Semua input Edge Function divalidasi skema (Zod) sebelum diproses.
- `message_contents.body` disanitasi dari karakter kontrol berbahaya, dibatasi panjang di DB dan aplikasi.

---

## 22. Privacy (Location Privacy) **[REVISI v1.1]**

- **Permission:** hanya `foreground location`, tidak ada background location.
- Lokasi disimpan sebagai **satu nilai terakhir** di `profile_locations`, bukan histori berkelanjutan.
- Pengguna dapat mengatur `location_precision`: `precise`, `city`, atau `off`.
- Jika `off` atau permission ditolak: pengguna **tidak bisa menjadi penerima** flight baru sampai diaktifkan kembali; tetap bisa mengirim pakai lokasi lama miliknya sendiri jika ada.
- **Koordinat mentah secara teknis tidak bisa dibaca oleh user lain** — `profile_locations` RLS hanya mengizinkan `auth.uid() = profile_id`. Nama kota untuk ditampilkan ke teman didapat lewat RPC `get-profile-city` yang mengembalikan string kota dari `profiles.city_cache`, bukan koordinat.

---

## 23. UI/UX Specification

*(Tidak berubah dari v1.0 — identitas visual §23.1, navigasi §23.2–23.3, spesifikasi 19 layar §23.5 tetap berlaku apa adanya. Tambahan satu elemen UI baru: tombol "Batalkan" muncul di layar Flight Tracking selama status `preparing`, dengan countdown mengikuti `preparing_window_seconds`.)*

### 23.1 Identitas Visual (ringkas)
- **Palet:** dusty blue `#5B7B9A`, cream parchment `#F4EFE6`, aksen tembaga/rose `#C97B63`.
- **Tipografi:** serif humanis untuk judul (kesan surat), sans-serif modern untuk body.
- **Ikon:** siluet merpati bersayap terbuka, digambar sendiri (SVG/vector orisinal).

### 23.2 Navigation Structure

Bottom Navigation (4 tab): **Home**, **Inbox**, **Bird Cage**, **Flock**. Profile & Settings dari ikon avatar di top bar.

### 23.3 Screen Hierarchy (ringkas)
```
Splash → Onboarding (first-run only) → Login/Register
   → Shell (Bottom Nav: Home / Inbox / Bird Cage / Flock)
        Home → Compose Message → Select Pigeon → Flight Tracking (dengan tombol Batalkan selama 'preparing')
        Inbox → Message Detail (→ Message Arrived state)
        Bird Cage → Pigeon Detail
        Flock → Friend Request list, Friend Detail
   → Profile (dari avatar) → Settings
```

### 23.5 Spesifikasi ringkas per layar

| # | Layar | Elemen kunci |
|---|---|---|
| 1 | Splash | Logo + tagline, auto-check session |
| 2 | Onboarding | 3 slide konsep (tulis pesan → merpati terbang → pesan tiba) |
| 3 | Login | Email, password, lupa password, link ke register |
| 4 | Register | Nama, username (cek unik realtime), email, password, konfirmasi |
| 5 | Home | Peta ringkas flight aktif, tombol compose besar (FAB), ringkasan status |
| 6 | Friends (Flock) | Search bar, list teman, badge friend request |
| 7 | Friend Request | List incoming/outgoing, tombol terima/tolak/blokir |
| 8 | Compose Message | Pilih penerima (dari teman), text area, counter karakter |
| 9 | Select Pigeon | Grid koleksi merpati available, info speed/rarity, tombol pilih; **jika kosong, tampilkan estimasi kapan merpati berikutnya tersedia** |
| 10 | Flight Tracking | Peta full-screen, marker origin/destination, garis rute, marker merpati bergerak, ETA countdown, **tombol Batalkan selama status 'preparing'** |
| 11 | Message Arrived | Modal/full-screen "amplop terbuka" animasi ringan saat pertama dibaca |
| 12 | Inbox | List pesan, indikator status merpati, unread dot |
| 13 | Message Detail | Isi pesan, info pengirim, riwayat penerbangan singkat |
| 14 | Pigeon Collection | Grid merpati dengan badge rarity & status, **termasuk indikator "resting X menit lagi" / "lost, pulih dalam Y jam"** |
| 15 | Pigeon Detail | Statistik lengkap, riwayat flight merpati tsb |
| 16 | Flock | List teman + status kota (dari `get-profile-city`) + jumlah pesan terkirim |
| 17 | Bird Cage | Tab: Riwayat pesan / Statistik / Koleksi ringkas |
| 18 | Profile | Foto, nama, username, statistik ringkas, tombol edit |
| 19 | Settings | Account, Notification, Privacy, Location, Language, Theme, Logout, Delete Account |

---

## 24. Screen-by-Screen (Interaksi Detail Kritis)

- **Compose → Select Pigeon → Flight Tracking** satu alur linear. Tombol "Kirim" di Select Pigeon memicu `create-flight` (status awal `preparing`).
- **Flight Tracking** saat `preparing` menampilkan countdown batal; setelah `preparing_window_seconds` lewat, otomatis pindah tampilan ke mode tracking penuh (server sudah mengubah status ke `flying`).
- **Message Detail** memanggil `mark-read` saat dibuka pertama kali (`delivered → read`).

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

---

## 26. Project Folder Structure

```
lib/
├── main.dart
├── app/
│   ├── app.dart
│   ├── router/app_router.dart
│   └── theme/
├── core/
│   ├── constants/
│   ├── utils/                    # haversine (preview client, non-authoritative), formatter tanggal
│   ├── errors/
│   └── network/supabase_client.dart
├── features/
│   ├── auth/
│   ├── profile/                  # termasuk profile_locations datasource
│   ├── friends/
│   ├── messages/                 # message metadata + message_contents datasource
│   ├── pigeons/                  # termasuk tampilan resting/lost recovery
│   ├── flights/
│   │   ├── data/                 # FlightsDatasource (create-flight, cancel-flight, subscribe realtime)
│   │   ├── domain/                # flight_position_calculator.dart (PURE, unit-tested)
│   │   └── presentation/
│   ├── notifications/
│   └── settings/
├── shared/
│   ├── widgets/
│   └── models/
└── l10n/

supabase/
├── migrations/
├── functions/
│   ├── create-flight/index.ts
│   ├── cancel-flight/index.ts              # [BARU v1.1]
│   ├── resolve-flight-status/index.ts
│   ├── resolve-flight-progress/index.ts    # [BARU v1.1]
│   ├── resolve-pigeon-recovery/index.ts    # [BARU v1.1]
│   ├── get-profile-city/index.ts           # [BARU v1.1]
│   ├── friend-request/index.ts
│   ├── mark-read/index.ts
│   └── _shared/                   # haversine.ts, config.ts, fcm.ts, geocode.ts
└── config.toml

test/
├── unit/flights/flight_position_calculator_test.dart
├── unit/flights/haversine_test.dart
└── widget/...
```

---

## 27. API Specification (Ringkas)

| Endpoint | Method | Auth | Deskripsi |
|---|---|---|---|
| `/functions/v1/create-flight` | POST | JWT user | Body: `{recipientId, pigeonId, messageBody, originLat, originLng, idempotencyKey}` → flight (status `preparing`) |
| `/functions/v1/cancel-flight` | POST | JWT user | **[BARU]** Body: `{flightId}` → hanya jika masih `preparing` |
| `/functions/v1/resolve-flight-status` | POST (cron) | Service role | Tidak dipanggil client |
| `/functions/v1/resolve-flight-progress` | POST (cron) | Service role | **[BARU]** Tidak dipanggil client |
| `/functions/v1/resolve-pigeon-recovery` | POST (cron) | Service role | **[BARU]** Tidak dipanggil client |
| `/functions/v1/get-profile-city` | POST | JWT user | **[BARU]** Body: `{profileId}` → `{city: string}` |
| `/functions/v1/friend-request` | POST | JWT user | Body: `{action: 'send'|'accept'|'reject'|'remove'|'block', targetUserId}` |
| `/functions/v1/mark-read` | POST | JWT user | Body: `{messageId}` → set status `read` |
| `/functions/v1/register-device` | POST | JWT user | Body: `{fcmToken, platform}` → upsert devices |

Response error terstandarisasi: `{ "error": { "code": "RECIPIENT_LOCATION_UNAVAILABLE", "message": "..." } }`.

---

## 28. Database Policies / RLS — v1.1 (Lengkap untuk Tabel Sensitif)

```sql
-- profiles: metadata publik boleh dibaca semua user terautentikasi
alter table profiles enable row level security;
create policy "profiles_select_all_authenticated"
  on profiles for select using (auth.role() = 'authenticated');
create policy "profiles_update_own"
  on profiles for update using (auth.uid() = id);

-- profile_locations: [BARU v1.1] HANYA pemilik yang boleh baca koordinat mentah
alter table profile_locations enable row level security;
create policy "profile_locations_select_own"
  on profile_locations for select using (auth.uid() = profile_id);
create policy "profile_locations_modify_own"
  on profile_locations for all using (auth.uid() = profile_id) with check (auth.uid() = profile_id);
-- Tidak ada policy select untuk user lain. get-profile-city (service role) yang
-- membaca tabel ini untuk menghasilkan city_cache, lalu menulis hasilnya ke
-- profiles.city_cache yang memang publik.

alter table pigeons enable row level security;
create policy "pigeons_select_own"
  on pigeons for select using (auth.uid() = owner_id);
create policy "pigeons_modify_own"
  on pigeons for all using (auth.uid() = owner_id) with check (auth.uid() = owner_id);

-- messages: metadata (status) boleh dilihat kedua peserta kapan pun
alter table messages enable row level security;
create policy "messages_select_participant"
  on messages for select
  using (auth.uid() = sender_id or auth.uid() = recipient_id);
-- insert/update HANYA lewat Edge Function (service role)

-- message_contents: [BARU v1.1] body HANYA boleh dibaca:
--   - sender, kapan pun (dia penulis aslinya)
--   - recipient, HANYA jika flight terkait sudah arrived/delivered/read
alter table message_contents enable row level security;
create policy "message_contents_select_sender"
  on message_contents for select
  using (
    exists (
      select 1 from messages m
      where m.id = message_contents.message_id
        and m.sender_id = auth.uid()
    )
  );
create policy "message_contents_select_recipient_after_arrival"
  on message_contents for select
  using (
    exists (
      select 1 from messages m
      join flights f on f.message_id = m.id
      where m.id = message_contents.message_id
        and m.recipient_id = auth.uid()
        and f.status in ('arrived', 'delivered', 'read')
    )
  );
-- Tidak ada policy insert/update untuk client biasa.

alter table flights enable row level security;
create policy "flights_select_participant"
  on flights for select
  using (auth.uid() = sender_id or auth.uid() = recipient_id);
-- update/insert flights: hanya service role (Edge Function)
-- catatan: kolom di flights (jarak, origin/destination) TIDAK dianggap
-- sensitif karena hanya milik dua peserta flight itu sendiri, bukan publik.

alter table friendships enable row level security;
create policy "friendships_select_participant"
  on friendships for select
  using (auth.uid() = requester_id or auth.uid() = addressee_id);
create policy "friendships_insert_own_request"
  on friendships for insert with check (auth.uid() = requester_id);
create policy "friendships_update_addressee_accept_reject"
  on friendships for update
  using (auth.uid() = addressee_id and status = 'pending');
create policy "friendships_update_either_side_block"     -- [REVISI v1.1]
  on friendships for update
  using (auth.uid() = requester_id or auth.uid() = addressee_id)
  with check (status = 'blocked' and blocked_by = auth.uid());

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

alter table rate_limits enable row level security;
-- rate_limits TIDAK punya policy select/insert untuk client — murni service role.
```

**Prinsip inti v1.1:** `messages`, `message_contents`, `flights`, `profile_locations`, dan `rate_limits` **tidak punya policy INSERT/UPDATE untuk role `authenticated`** — hanya bisa ditulis oleh Edge Function (`service_role`). `message_contents` bahkan membatasi **SELECT** berdasarkan status flight, bukan cuma keanggotaan peserta.

---

## 29. Error Handling

- Format seragam: `{ error: { code, message } }` dengan HTTP status sesuai (400/401/403/404/409/429/500).
- Client memetakan `code` ke pesan Bahasa Indonesia via `error_messages.dart` terpusat.
- Exception tak terduga ditangkap, dicatat, dikembalikan sebagai `code: 'INTERNAL_ERROR'` tanpa membocorkan stack trace.
- **[BARU v1.1]** Kode error baru: `NO_PIGEON_AVAILABLE` (menyertakan `retryAfterSeconds`), `FLIGHT_NOT_CANCELLABLE`, `RATE_LIMIT_EXCEEDED` (menyertakan `retryAfterSeconds`).

---

## 30. Edge Cases

| # | Kasus | Penanganan |
|---|---|---|
| 1 | Pengirim offline saat submit | Idempotency key, retry aman saat online |
| 2 | Penerima offline | Flight tetap dibuat dari `profile_locations` tersimpan; notifikasi dikirim saat device online |
| 3 | GPS mati | Fallback ke lokasi tersimpan; jika tidak ada, minta input kota manual atau tolak dengan pesan jelas |
| 4 | GPS tidak akurat | Terima `accuracy_meters`; jika > threshold, tetap dipakai + indikator "kurang akurat" |
| 5 | Permission lokasi ditolak | App tetap bisa dipakai; user tidak bisa jadi *destination* baru sampai diaktifkan |
| 6 | Penerima pindah lokasi saat merpati terbang | Sengaja tidak memengaruhi flight berjalan — `destination` immutable saat keberangkatan |
| 7 | Aplikasi ditutup | Flight state 100% di server, dihitung ulang saat dibuka lagi |
| 8 | HP restart | Sama seperti #7 |
| 9 | Internet terputus saat tracking | Indikator offline, tetap animasi posisi lokal, sinkron ulang saat online |
| 10 | Server error saat create-flight | Rollback penuh transaksi, client tampilkan error & retry |
| 11 | Duplicate message (double tap) | Idempotency key mencegah dua flight dari satu aksi |
| 12 | Duplicate flight | `flights.message_id` unique constraint |
| 13 | User kirim banyak pesan cepat | Rate limiting via `rate_limits` (§21.3), UI tampilkan cooldown |
| 14 | Spam | Rate limit + `reports` table + blokir |
| 15 | Merpati hilang | **[REVISI v1.1]** Tidak permanen — pulih otomatis setelah `pigeon_lost_recovery_hours` (§17.7) |
| 16 | Flight terlalu lama (jarak sangat jauh) | `max_flight_duration_seconds` meng-cap durasi maksimum |
| 17 | Jarak sangat dekat | `min_flight_duration_seconds` mencegah durasi nyaris-instan |
| 18 | User menghapus akun | **[REVISI v1.1 — diputuskan final]** Data milik sendiri (pigeons, sent messages sbg sender, profile_locations) dihapus cascade. Untuk flight yang user tsb jadi **recipient**-nya: `messages`/`message_contents`/`flights` tetap ada demi integritas histori sender, tapi `recipient_id` di-set ke satu akun sistem placeholder `deleted_user` (dibuat khusus saat migrasi awal) sehingga tidak ada FK menggantung dan sender tetap bisa melihat riwayat kirimnya sendiri. |
| 19 | User memblokir teman | `friendships.status='blocked'` (bisa dari requester **atau** addressee, lihat RLS §28); `create-flight` menolak pengiriman baru antara dua user blocked di kedua arah |
| 20 | Timezone berbeda | Semua timestamp `timestamptz` (UTC); konversi ke lokal murni di UI client |
| 21 | Perubahan koordinat selama penerbangan | Lihat #6 — snapshot immutable |
| 22 | Flight sedang berjalan saat deploy backend baru | State di DB, deploy tidak memengaruhi flight aktif |
| 23 **[BARU v1.1]** | User membatalkan setelah preparing_window lewat | Ditolak dengan `FLIGHT_NOT_CANCELLABLE` — window sudah tertutup, flight sudah `flying` |
| 24 **[BARU v1.1]** | Semua pigeon user dalam status non-available | Mekanisme merpati darurat (§17.7) memberi 1 pigeon common sekali pakai |
| 25 **[BARU v1.1]** | Recipient mencoba baca `message_contents` langsung via PostgREST sebelum arrived | RLS menolak — baris tidak muncul sama sekali (bukan error, tapi empty result, sesuai perilaku default-deny RLS) |
| 26 **[BARU v1.1]** | User lain mencoba baca `profile_locations` langsung via PostgREST | RLS menolak — baris tidak muncul sama sekali |

---

## 31. Testing Strategy

- **Unit test (wajib, prioritas tertinggi):** `flight_position_calculator.dart`, `haversine.dart`, state machine transisi status pesan/flight — pure function tanpa Flutter/network.
- **Edge Function test:** Supabase CLI lokal — create-flight normal, recipient tanpa lokasi, rate limit terlampaui, idempotency key duplikat, cancel-flight sebelum/sesudah window, pigeon recovery.
- **RLS bypass test [BARU v1.1, wajib sebelum rilis]:** dengan dua akun uji A (sender) dan B (recipient):
  1. Sebelum flight `arrived`, login sebagai B, `curl` langsung ke PostgREST `message_contents?message_id=eq.<id>` → harus mengembalikan array kosong.
  2. Login sebagai user C (bukan A atau B), coba SELECT `profile_locations` milik A → harus kosong.
  3. Sebagai requester yang belum jadi addressee, coba UPDATE `friendships.status='blocked'` → harus berhasil (mengonfirmasi fix §28).
- **Widget test:** Compose form validation, Select Pigeon list (termasuk state "kosong"), Flight Tracking marker render, tombol Batalkan.
- **Integration test (opsional Production):** register → add friend → send message → preparing→flying→arrived → inbox update.
- **Manual QA checklist** sebelum tiap rilis APK: 26 edge case di §30 dicek manual minimal sekali.

---

## 32. Development Roadmap

> Setiap fase: Objective, Fitur, File dibuat, DB, Dependency, Backend, Testing, DoD.
> **[REVISI v1.1]** Phase 7 (Flight Engine) sekarang eksplisit mencakup `cancel-flight`, `resolve-flight-progress`, `resolve-pigeon-recovery`, dan RLS `message_contents`/`profile_locations` — karena fitur-fitur ini saling bergantung dan sebaiknya tidak dipecah lintas fase.

### Phase 0 — Environment Setup
- **Objective:** siapkan tooling agar AI agent bisa langsung produktif.
- **File:** `pubspec.yaml`, `.env.example`, `README.md`, `analysis_options.yaml`
- **DB:** buat project Supabase, catat URL & anon key
- **Dependency:** Flutter SDK, Supabase CLI, Docker (opsional)
- **DoD:** `flutter run` menampilkan halaman kosong tanpa error; Supabase project bisa diakses dari dashboard.

### Phase 1 — Project Initialization
- **Fitur:** shell navigasi (4 tab placeholder)
- **File:** `app/app.dart`, `app/router/app_router.dart`, `app/theme/*`
- **Dependency:** `go_router`, `flutter_riverpod`
- **DoD:** app bisa berpindah 4 tab dengan tema PigeonMail.

### Phase 2 — Authentication
- **Fitur:** Register, Login, Forgot Password, Session persistence
- **File:** `features/auth/**`
- **DB:** RLS `profiles`, trigger auto-insert `profiles` **dan** `profile_locations` (kosong) **dan** `settings` (default) saat user baru daftar
- **DoD:** user baru bisa daftar, verifikasi email, login, logout, sesi bertahan.

### Phase 3 — Profile
- **Fitur:** view/edit profil, upload avatar, update lokasi manual (menulis ke `profile_locations`)
- **File:** `features/profile/**`
- **DB:** `profiles`, `profile_locations`, bucket Storage `avatars`
- **Dependency:** `image_picker`, `geolocator`
- **DoD:** user bisa ubah nama/username/foto/lokasi dan tersimpan permanen; koordinat tidak bisa dibaca user lain (uji manual dengan akun kedua).

### Phase 4 — Friend System
- **Fitur:** search user, friend request flow lengkap, blokir dua arah
- **File:** `features/friends/**`, `supabase/functions/friend-request/`
- **DB:** `friendships` + RLS (§28, termasuk fix blokir)
- **DoD:** dua akun uji bisa berteman, dan **kedua arah** bisa saling memblokir.

### Phase 5 — Messaging (skeleton, tanpa flight dulu)
- **Fitur:** Compose screen, Inbox list, Outbox list (status manual/dummy)
- **File:** `features/messages/**`
- **DB:** `messages`, `message_contents` (tanpa flight dulu, status manual untuk UI dev)
- **DoD:** UI compose, inbox, outbox berfungsi dengan data dummy/lokal.

### Phase 6 — Pigeon System
- **Fitur:** Pigeon Collection grid, Pigeon Detail, seed 2-3 merpati awal per user baru
- **File:** `features/pigeons/**`
- **DB:** `pigeons`, trigger seed default pigeon saat register
- **DoD:** user baru otomatis punya minimal 1 merpati common; bisa lihat koleksi & detail.

### Phase 7 — Flight Engine (INTI) **[REVISI v1.1 — cakupan diperluas]**
- **Objective:** implementasi penuh §17, termasuk siklus preparing/cancel dan pemulihan pigeon.
- **Fitur:** create-flight, cancel-flight, resolve-flight-status, resolve-flight-progress, resolve-pigeon-recovery, state machine lengkap
- **File:** `supabase/functions/create-flight/`, `cancel-flight/`, `resolve-flight-status/`, `resolve-flight-progress/`, `resolve-pigeon-recovery/`, `features/flights/domain/flight_position_calculator.dart`
- **DB:** `flights`, `message_contents`, `rate_limits`, `app_config`, semua RLS sensitif di §28
- **Testing:** unit test kalkulator posisi & haversine (wajib coverage tinggi), Edge Function test skenario §30, **RLS bypass test §31 wajib lulus sebelum lanjut ke Phase 8**
- **DoD:** kirim pesan sungguhan → `preparing` → (bisa dibatalkan atau) → `flying` → `arrived` otomatis via resolver; pigeon pulih otomatis; RLS bypass test lulus semua.

### Phase 8 — Map
- **Fitur:** Flight Tracking screen dengan marker origin/destination/merpati bergerak + garis rute + tombol Batalkan saat preparing
- **File:** `features/flights/presentation/screens/flight_tracking_screen.dart`, `widgets/pigeon_map_marker.dart`
- **Dependency:** `flutter_map`, `latlong2`
- **DoD:** posisi merpati bergerak halus sesuai progress, tombol batalkan berfungsi selama window preparing.

### Phase 9 — Realtime
- **Fitur:** subscribe channel `flights`, `messages`, `friendships`, `notifications`
- **DB:** aktifkan Realtime publication pada tabel terkait
- **DoD:** perubahan status di satu device langsung tercermin di device lain tanpa reload.

### Phase 10 — Notification & Geocoding
- **Fitur:** register device token, kirim FCM untuk 7 event, `get-profile-city` untuk tampilan kota di Flock
- **File:** `features/notifications/**`, `supabase/functions/_shared/fcm.ts`, `_shared/geocode.ts`, `register-device`, `get-profile-city`
- **DB:** `devices` (+`is_valid`), `notifications`, `profiles.city_cache`
- **Dependency:** `firebase_messaging`, Firebase project setup
- **DoD:** semua 7 event menghasilkan push notification; nama kota tampil di Flock tanpa membocorkan koordinat.

### Phase 11 — UI Polish
- **Fitur:** empty/loading/error state seragam, animasi transisi halus, ilustrasi custom, indikator resting/lost di Pigeon Collection
- **DoD:** semua 19 layar memenuhi standar empty/loading/error dan identitas visual konsisten.

### Phase 12 — Security
- **Fitur:** rate limiting final, review semua RLS policy (termasuk yang baru), audit secret
- **Testing:** uji manual bypass (curl langsung ke PostgREST tanpa lewat function) untuk `message_contents` dan `profile_locations` khususnya
- **DoD:** tidak ada tabel sensitif yang bisa ditulis/dibaca langsung tanpa lewat jalur yang seharusnya; semua secret di environment variable.

### Phase 13 — Testing
- **File:** lengkapi folder `test/`
- **Testing:** seluruh unit/widget/edge-function test, checklist 26 edge case manual (§30)
- **DoD:** seluruh test hijau; checklist edge case lulus manual.

### Phase 14 — Build APK
- **File:** `android/app/build.gradle` (signing config), `key.properties`
- **DoD:** `flutter build apk --release` sukses, APK terinstal dan berjalan penuh di device fisik.

### Phase 15 — Production Deployment
- **File:** listing metadata (deskripsi, ikon, screenshot)
- **DB:** migration terkunci/versioned, backup policy Supabase diaktifkan, akun placeholder `deleted_user` dibuat
- **DoD:** aplikasi berjalan stabil untuk kelompok penguji nyata selama minimal beberapa hari tanpa insiden kritikal.

---

## 33. MVP Scope (Ringkasan Final)

Termasuk: Auth, Profile (dengan lokasi terpisah aman), Friend System (blokir dua arah), Messaging 1-ke-1 (isi pesan terkunci sampai tiba), Pigeon collection dengan siklus pemulihan otomatis (3 rarity), Flight Engine penuh termasuk preparing/cancel, Map tracking, Inbox/Outbox, Bird Cage ringkas, Flock (list saja), Notifikasi 7 jenis (termasuk arriving yang query-nya benar), Settings dasar, rate limiting konkret.

Tidak termasuk: group messaging, attachment media, breeding/upgrade merpati, marketplace, cuaca dinamis, rute non-linear/great-circle, multi-bahasa penuh.

## 34. Future Features

Group/Flock chat, attachment (gambar/voice note terlampir pada surat), sistem cuaca yang memengaruhi kecepatan & risiko lost, achievement/badge, breeding merpati, marketplace kosmetik, mode "pen-pal acak" (kirim ke stranger), widget home-screen menampilkan flight aktif.

## 35. Cost Estimation

| Komponen | Tier | Estimasi biaya (skala < 1.000 MAU) |
|---|---|---|
| Supabase | Free/Pro | $0 (Free tier) hingga ~$25/bln jika naik ke Pro |
| Firebase Cloud Messaging | Free | $0 |
| Map tiles (OSM via flutter_map) | Free | $0 (perhatikan fair-use policy) |
| Nominatim geocoding **[BARU v1.1]** | Free | $0 (fair-use ~1 req/detik; sudah di-cache di `city_cache` sehingga trafik ke Nominatim sangat kecil) |
| Domain (opsional) | — | ~$10-15/tahun |
| Google Play Console | One-time | $25 sekali bayar |
| **Total estimasi awal** | | **$0–$25 sekali bayar + $0/bln selama masih dalam batas free tier** |

## 36. Deployment Strategy

1. Development: Supabase local (Docker) + `flutter run` debug.
2. Staging: Supabase project terpisah (`pigeonmail-staging`).
3. Production: Supabase project (`pigeonmail-prod`), migration via `supabase db push` terversi di `supabase/migrations`.
4. Mobile release: APK sideload internal → Google Play Internal Testing → Closed Testing → (opsional) rilis publik.
5. CI: GitHub Actions menjalankan `flutter analyze` + `flutter test` di setiap PR.

---

## 37. AI Coding Agent Rules

1. **Satu fase = satu unit kerja.** Ikuti urutan Phase 0–15 di §32; jangan mengerjakan fitur di luar fase aktif tanpa diminta eksplisit.
2. **Jangan mengubah skema DB tanpa file migration baru.** Semua perubahan schema wajib lewat file baru di `supabase/migrations/`.
3. **Flight engine logic hanya boleh berubah di** `supabase/functions/create-flight`, `cancel-flight`, `resolve-flight-status`, `resolve-flight-progress`, `resolve-pigeon-recovery`, plus counterpart pure function di `flight_position_calculator.dart`. Tidak ada logic kalkulasi jarak/kecepatan/lost di layer presentation.
4. **Setiap fungsi pure baru di `domain/` wajib disertai unit test pada PR yang sama.**
5. **Naming convention:** file `snake_case.dart`, class `PascalCase`, provider `xxxProvider` (Riverpod), Edge Function folder `kebab-case`.
6. **Setiap fitur baru wajib punya:** deskripsi singkat di komentar header, TODO eksplisit untuk bagian yang sengaja belum diimplementasikan, acceptance criteria sebagai komentar test.
7. **Tidak boleh menyalin ikon, aset, atau teks dari aplikasi manapun.**
8. **Perubahan satu fitur tidak boleh menyentuh file fitur lain** kecuali `shared/`/`core/`, dijelaskan alasannya di commit message.
9. **Semua secret hanya lewat environment variable / Supabase Secrets.**
10. **Sebelum menandai Phase selesai, jalankan Definition of Done** di §32 untuk fase tersebut.
11. **[BARU v1.1] Perubahan pada RLS policy tabel `message_contents` atau `profile_locations` wajib disertai RLS bypass test baru di §31 pada PR yang sama** — dua tabel ini adalah jaminan privasi inti produk, bukan tabel biasa.

## 38. Definition of Done (Global)

Sebuah fitur dianggap selesai jika: (1) memenuhi DoD spesifik fase-nya di §32, (2) tidak menimbulkan regresi pada test yang sudah ada, (3) RLS/security review untuk tabel yang disentuh sudah dicek ulang terhadap §21 & §28, (4) tidak ada TODO kritikal tersisa tanpa catatan di README, (5) sudah diuji manual minimal sekali di device/emulator nyata.

## 39. Risks and Mitigation

| Risiko | Dampak | Mitigasi |
|---|---|---|
| Free tier Supabase terlampaui saat demo/testing ramai | App down sementara | Monitor usage dashboard, siapkan upgrade Pro tier sebelum demo penting |
| Ketergantungan pada tile server OSM publik (rate limit) | Peta gagal load saat traffic tinggi | Cache tile lokal ringan, siapkan fallback provider berbayar murah |
| Ketergantungan Nominatim untuk geocoding **[BARU v1.1]** | Nama kota gagal muncul saat traffic tinggi | Cache di `city_cache`, fallback tampilkan "Lokasi dirahasiakan" jika gagal, bukan blocking error |
| Resolver cron gagal jalan (down time) | Flight menumpuk melebihi `arrival_at` | Semua resolver idempotent & bisa dijalankan manual/backfill kapan saja |
| Solo developer kehabisan waktu | Fitur tidak selesai sesuai roadmap | Roadmap granular per fase, tiap fase punya DoD independen |
| Kesalahan RLS policy membocorkan data pengguna lain | Pelanggaran privasi serius | **[REVISI v1.1]** Sudah ditutup secara struktural lewat pemisahan `message_contents`/`profile_locations`; tetap wajib RLS bypass test di §31 sebelum tiap rilis |
| Kebingungan AI agent lintas sesi | Kode inkonsisten antar fase | Dokumen ini + §37 + komentar header tiap file sebagai single source of truth |

---

## 40. Final Recommended Architecture

**Flutter (Riverpod + go_router) sebagai client tipis, Supabase sebagai backend-as-a-service tunggal** (Postgres relasional untuk integritas data sosial/pesan, Edge Functions sebagai lapisan authoritative untuk flight engine, Realtime untuk sinkronisasi status, Storage untuk aset, Auth untuk identitas), dilengkapi **FCM** untuk push notification, **flutter_map/OSM** untuk peta bebas biaya, dan **Nominatim** untuk reverse-geocoding kota.

Prinsip arsitektural inti yang tidak boleh dilanggar:
1. *Server-authoritative* untuk semua hasil yang berdampak game/kepercayaan (jarak, kecepatan, probabilitas lost).
2. Posisi visual dihitung matematis di client dari data statis (bukan disimpan tiap detik).
3. RLS sebagai pertahanan berlapis di atas logic Edge Function.
4. **[BARU v1.1]** Untuk data yang janji produknya bergantung padanya (isi pesan sebelum tiba, koordinat presisi orang lain) — RLS bukan pertahanan tambahan, tapi **sumber kebenaran utama** yang berdiri sendiri meski Edge Function ada bug.

---

## FINAL TECH STACK

```
Frontend        : Flutter (Dart) + Riverpod + go_router
Backend         : Supabase (Auth, PostgREST, Realtime, Storage, Edge Functions)
Database        : PostgreSQL (Supabase-managed), tanpa PostGIS di MVP
Authentication  : Supabase Auth (email/password + email verification)
Realtime        : Supabase Realtime (channel: flights, messages, notifications, friendships)
Maps            : flutter_map + OpenStreetMap tiles (upgrade opsional ke Mapbox di Production)
Geocoding       : Nominatim (via Edge Function get-profile-city, hasil di-cache)
Push Notification : Firebase Cloud Messaging (FCM HTTP v1), dipicu dari Edge Function
Storage         : Supabase Storage (bucket: avatars, pigeon-assets)
State Management: Riverpod
Routing         : go_router
Testing         : flutter_test, mocktail, Supabase CLI local testing untuk Edge Function
CI/CD           : GitHub Actions (flutter analyze + flutter test per PR)
Deployment      : APK internal → Google Play Internal/Closed Testing track
```

## RECOMMENDED BUILD ORDER

1. Environment Setup (Phase 0).
2. Project skeleton + tema + navigasi 4 tab kosong (Phase 1).
3. Authentication penuh + auto-create profile/profile_locations/settings (Phase 2).
4. Profile (view/edit + avatar + lokasi manual ke `profile_locations`) (Phase 3).
5. Friend System penuh termasuk blokir dua arah (Phase 4).
6. Messaging UI skeleton dengan data dummy (Phase 5).
7. Pigeon System dasar + seed default (Phase 6).
8. **Flight Engine inti — paling kritikal.** Kerjakan create-flight, cancel-flight, kedua resolver, pemulihan pigeon, dan RLS sensitif dalam satu fase yang sama, dengan RLS bypass test lulus sebelum lanjut (Phase 7).
9. Hubungkan Messaging UI ke Flight Engine nyata.
10. Map tracking + tombol batalkan (Phase 8).
11. Realtime wiring (Phase 9).
12. Notification (FCM) + geocoding kota (Phase 10).
13. UI Polish menyeluruh (Phase 11).
14. Security hardening & review RLS/rate-limit, ulangi RLS bypass test (Phase 12).
15. Testing menyeluruh + checklist 26 edge case (Phase 13).
16. Build APK release & smoke test device fisik (Phase 14).
17. Production deployment bertahap (Phase 15).

**Catatan penting:** Flight Engine (langkah 8) adalah fondasi yang menentukan seluruh pengalaman produk sekaligus jaminan privasi inti — jangan lanjut ke langkah berikutnya sampai unit test kalkulator posisi, Edge Function `create-flight`/`cancel-flight`, dan **ketiga skenario RLS bypass test di §31** benar-benar lulus.
