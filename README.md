# Lentera — Perpustakaan Digital (Django)

Website perpustakaan digital teknik berbasis **Django**: katalog buku, e-journal, pembaca PDF, peminjaman, bookmark, riwayat, notifikasi, leaderboard, achievement, streak membaca, dan dashboard admin. Situs: https://lentera.pbo.web.id/

README ini disusun sebagai bahan belajar dan presentasi PBO. **Konsep OOP dijelaskan per fitur website**, mulai dari halaman yang dilihat user sampai halaman admin, lengkap dengan file dan nomor baris di kode.

Isi:

1. Gambaran proyek
2. Peta fitur website (user dan admin)
3. Dasar OOP: pengertian dan analogi
4. Class inti proyek (`models.py`)
5. **OOP per fitur: User**
6. **OOP per fitur: Admin**
7. Relasi antarobjek (asosiasi, agregasi, komposisi, dependensi)
8. Alur program (ringkas)
9. Tanya jawab
10. Catatan jujur (cek sebelum presentasi)

> **Cakupan.** Semua nomor baris di README ini sudah dicocokkan dengan kode di `Lentera-main.zip`: `config/urls.py`, `perpustakaan/urls.py`, `models.py`, `views.py`, `views_admin.py`, `admin.py`, `tests.py`, dan template. Hal yang belum atau tidak cocok ditandai di bagian 10.

**Aturan membaca README ini**

- Semua *class* OOP ada di backend Python (`models.py`, `views_admin.py`, `admin.py`). JavaScript proyek tidak memakai `class`.
- Kebanyakan view adalah **fungsi biasa**. OOP-nya ada pada **object yang dipakai** fungsi itu (misalnya object `CustomUser`, `BukuDigital`, `Peminjaman`).
- Tiap fitur punya kolom **Bukti** (yang boleh diklaim) dan **Jangan klaim** (yang sering dikira OOP padahal bukan).

---

## 1. Gambaran Proyek

### Teknologi
- Python dan Django (pola MTV), SQLite
- HTML + Django Template Language, CSS, JavaScript
- PDF.js untuk membaca PDF, Font Awesome untuk ikon

### Struktur folder (ringkas)

```
Lentera-main/
├── config/                      # settings.py, urls.py utama
├── perpustakaan/                # app utama
│   ├── models.py                # struktur data + logika objek (inti OOP)
│   ├── views.py                 # logika halaman anggota
│   ├── views_admin.py           # logika halaman admin (termasuk AchievementForm)
│   ├── urls.py                  # peta alamat app
│   ├── admin.py                 # panel /admin bawaan Django
│   ├── tests.py                 # tes otomatis
│   ├── static/perpustakaan/     # css, js (script, buku, baca-ring, notifikasi, admin-dashboard), img
│   └── templates/perpustakaan/  # halaman user, base_admin.html, admin/
├── media/                       # upload (PDF buku, foto profil, gambar achievement)
├── manage.py
└── requirements.txt             # Django 6.1.1, Pillow, requests, dll.
```

### Menjalankan

```bash
python -m venv venv
venv\Scripts\activate            # Windows
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
python manage.py test            # tes otomatis (lihat bagian 10, ada 3 tes yang gagal)
```

---

## 2. Peta Fitur Website

Menu user (sidebar): Dashboard, Katalog Buku, E-journal, Peminjaman, Bookmarks, Riwayat Peminjaman, Top List Reader, Profil. Menu admin: Dashboard, Kelola Buku, Kelola Jurnal, Kelola Achievement, Kelola Anggota, Peminjaman, Laporan, Pengumuman.

### User

| # | Fitur | Alamat | View | Class utama yang dipakai |
|---|---|---|---|---|
| U1 | Login (+ lupa password OTP) | `login/` | `login_view` (`views.py:262`), `_proses_reset` (`:88`) | `CustomUser` |
| U2 | Register | `register/` | `register_view` (`:307`) | `CustomUser` |
| U3 | Dashboard | `/` | `dashboard` (`:375`) | `CustomUser`, `Peminjaman`, `BukuDigital`, `Bookmark` |
| U4 | Katalog Buku | `katalog/` | `katalog_view` (`:462`) | `BukuDigital`, `Bookmark` |
| U5 | Detail Buku dan Membaca | `buku/<olid>/`, `baca/<olid>/` | `buku_detail` (`:559`), `baca_buku` (`:599`), `simpan_progress` (`:624`) | `BukuDigital`, `Peminjaman` |
| U6 | Peminjaman (pinjam, perpanjang, kembalikan) | `peminjaman/`, `buku/<olid>/pinjam/` | `peminjaman_view` (`:794`), `pinjam_buku` (`:643`), `perpanjang_buku` (`:839`), `kembalikan_buku` (`:820`) | `Peminjaman` |
| U7 | E-Journal | `ejournal/`, `jurnal/<id>/baca/` | `ejournal_view` (`:684`), `baca_jurnal` (`:744`) | `Jurnal` |
| U8 | Bookmarks | `bookmarks/` | `bookmarks_view` (`:877`), `toggle_bookmark` (`:669`), `hapus_bookmark_api` (`:906`) | `Bookmark` |
| U9 | Riwayat Peminjaman | `riwayat/` | `riwayat_view` (`:916`) | `Peminjaman` |
| U10 | Top List Reader | `leaderboard/` | `leaderboard_view` (`:930`) | `CustomUser` |
| U11 | Profil (milik sendiri dan publik) | `profil/`, `profil/<username>/` | `profil_view` (`:957`), `profil_user_detail_view` (`:1190`) | `CustomUser` |
| U12 | Waktu baca, EXP, streak | `api/waktu-baca/` | `waktu_baca` (`:1262`) | `CustomUser`, `Peminjaman`, `Notifikasi` |
| U13 | Achievement | `achievement/`, `api/update-pinned/` | `achievement_view` (`:1327`), `update_pinned_achievement_api` (`:1342`) | `AchievementGambar` |
| U14 | Notifikasi | `notifikasi/` | `notifikasi_view` (`:216`), `buka_notifikasi` (`:255`) | `Notifikasi` |

### Admin (semua berawalan `dashboard-admin/`, di `views_admin.py`)

| # | Fitur | Alamat | View | Class utama yang dipakai |
|---|---|---|---|---|
| A1 | Dashboard Admin | `dashboard-admin/` | `dashboard_admin` (`:158`) | `BukuDigital`, `Jurnal`, `Peminjaman`, `CustomUser` |
| A2 | Kelola Buku | `.../buku/` (+ edit, hapus, hapus-terpilih) | `kelola_buku` (`:209`), `edit_buku` (`:278`), `hapus_buku` (`:271`), `hapus_buku_massal` (`:311`) | `BukuDigital`, `Notifikasi` |
| A3 | Kelola Jurnal | `.../jurnal/` (+ edit, hapus, hapus-terpilih) | `kelola_jurnal` (`:326`), `edit_jurnal` (`:418`), `hapus_jurnal` (`:411`), `hapus_jurnal_massal` (`:453`) | `Jurnal`, `Notifikasi` |
| A4 | Kelola Achievement | `.../achievement/` | `kelola_achievement` (`:540`), `edit_achievement` (`:575`), `AchievementForm` (`:471`) | `AchievementGambar` |
| A5 | Kelola Anggota | `.../anggota/` | `kelola_anggota` (`:674`), `setujui_anggota` (`:717`), `tolak_anggota` (`:739`), `edit_anggota` (`:761`), `hapus_anggota` (`:809`), `aktifkan_anggota` (`:823`) | `CustomUser`, `Notifikasi` |
| A6 | Peminjaman (admin) | `.../peminjaman/` | `peminjaman_admin` (`:896`) | `Peminjaman` |
| A7 | Laporan | `.../laporan/`, `.../laporan/unduh/` | `laporan` (`:923`), `unduh_laporan` (`:986`) | `CustomUser`, `Peminjaman`, `BukuDigital`, `Jurnal` |
| A8 | Pengumuman | `.../notifikasi/` | `buat_pengumuman` (`:389`) | `Notifikasi`, `CustomUser` |

Panel `/admin` bawaan Django juga dipakai untuk `AchievementGambar` (`admin.py`), dan model lain didaftarkan dengan tampilan bawaan.

---

## 3. Dasar OOP: Pengertian dan Analogi

### 3.0 Class dan Object

- **Class**: cetak biru berisi atribut (data) dan method (perilaku).
- **Object**: hasil nyata (instance) dari class, dengan nilai atribut sendiri.

**Analogi:** class = cetakan kue, object = kue-kuenya.

```python
class Kucing:
    def __init__(self, nama):
        self.nama = nama

k1 = Kucing("Mimi")     # object 1
k2 = Kucing("Tom")      # object 2
```

**Di proyek:** `BukuDigital` adalah class. Satu baris buku tertentu di database adalah object.

### 3.1 Encapsulation (Enkapsulasi)

Membungkus data dan method pengolahnya dalam satu class, supaya data diubah lewat method yang punya aturan.

**Analogi: mesin ATM.** Saldo tidak bisa diubah langsung. Kamu memakai menu tarik atau setor, dan menu itu menjalankan aturannya.

**Catatan Python:** tidak ada `private` yang ketat seperti Java. Enkapsulasi di Python berupa pengelompokan data dengan method, `@property`, dan konvensi awalan `_` untuk method internal.

```python
class Rekening:
    def __init__(self, saldo):
        self._saldo = saldo

    def tarik(self, jumlah):
        if jumlah > self._saldo:
            raise ValueError("Saldo tidak cukup")
        self._saldo -= jumlah
```

### 3.2 Abstraction (Abstraksi)

Menyembunyikan detail rumit dan hanya menampilkan yang perlu. Bentuknya bisa class kerangka yang tidak dibuat objeknya langsung, atau lapisan yang menyembunyikan proses di baliknya.

**Analogi: mengemudi mobil.** Pengemudi memakai setir dan pedal tanpa tahu cara mesin membakar bahan bakar.

```python
from abc import ABC, abstractmethod

class Bentuk(ABC):
    @abstractmethod
    def luas(self): ...

class Persegi(Bentuk):
    def __init__(self, sisi): self.sisi = sisi
    def luas(self): return self.sisi ** 2
```

### 3.3 Inheritance (Pewarisan)

Class anak mewarisi atribut dan method class induk, lalu boleh menambah atau mengubah. Hubungan **"is-a"**.

**Analogi:** anak mewarisi sifat orang tua tetapi punya ciri sendiri.

```python
class Hewan:
    def __init__(self, nama): self.nama = nama

class Kucing(Hewan):
    def mengeong(self): return f"{self.nama}: meong"
```

Jenis pewarisan di proyek:

| Jenis | Ada? | Contoh |
|---|---|---|
| Single | Ya | `CustomUser(AbstractUser)`, `AchievementForm(forms.ModelForm)` |
| Multilevel | Ya | `BukuDigital` → `ItemKatalog` → `models.Model` |
| Hierarchical | Ya | `ItemKatalog` → `BukuDigital` dan `Jurnal` |
| Multiple | Tidak ditulis sendiri | `AbstractUser` Django mewarisi `AbstractBaseUser` dan `PermissionsMixin` |
| Hybrid | Tidak ada | - |

### 3.4 Polymorphism (Polimorfisme)

"Banyak bentuk": satu nama method atau satu cara pemanggilan menghasilkan **perilaku berbeda** tergantung objeknya. Bentuk paling umum: **method overriding**.

**Analogi: tombol "Play"** di pemutar musik, video, dan podcast. Cara menekan sama, hasilnya beda.

**Catatan Python:** tidak ada method overloading seperti Java. Polimorfisme muncul lewat overriding dan duck typing.

```python
class Hewan:
    def suara(self): return "..."
class Kucing(Hewan):
    def suara(self): return "meong"
class Anjing(Hewan):
    def suara(self): return "guk"

for h in (Kucing(), Anjing()):
    print(h.suara())          # meong / guk
```

**Uji empat ciri** sebuah contoh dihitung polimorfisme: (1) ada hubungan class, (2) nama method sama, (3) dipanggil dengan cara sama, (4) hasilnya berbeda per objek.

### 3.5 Relasi antarobjek

| Relasi | Makna | Siklus hidup | Analogi |
|---|---|---|---|
| **Dependensi** | memakai sementara | tidak terikat | tukang memakai palu untuk satu pekerjaan |
| **Asosiasi** | berhubungan lewat referensi | masing-masing hidup sendiri | kartu peminjaman yang menunjuk anggota dan buku |
| **Agregasi** | has-a lemah | bagian **bisa hidup tanpa** induk | perpustakaan tutup, buku tetap ada |
| **Komposisi** | has-a kuat | bagian **ikut hilang** bersama induk | rumah dibongkar, kamarnya ikut hilang |

Pertanyaan pembeda agregasi dan komposisi: *"kalau induknya dihapus, apakah bagiannya masih bermakna?"* Masih → agregasi. Tidak → komposisi.

### 3.6 Ringkasan

| Pilar | Inti | Analogi |
|---|---|---|
| Encapsulation | data + method dibungkus, akses terkontrol | ATM |
| Abstraction | sembunyikan detail, tampilkan yang perlu | mengemudi mobil |
| Inheritance | anak mewarisi induk (is-a) | keluarga |
| Polymorphism | nama sama, perilaku beda | tombol Play |

---

## 4. Class Inti Proyek (`models.py`)

Bagian ini penting karena hampir semua fitur di bagian 5 dan 6 memakai class-class ini.

| Class | Letak | Peran OOP |
|---|---|---|
| `ItemKatalog` (abstract) | `models.py:33` | Abstraksi |
| `BukuDigital(ItemKatalog)` | `:49` | Pewarisan, property `kategori_label` (`:57`) |
| `Jurnal(ItemKatalog)` | `:69` | Pewarisan |
| `Peminjaman` | `:84` | Asosiasi (`ForeignKey` ke user, buku, jurnal) |
| `Bookmark` | `:111` | Asosiasi |
| `Achievement`, `UserAchievement` | `:129`, `:147` | Model lama, hanya terdaftar di `admin.py` |
| `Notifikasi` | `:157` | Asosiasi/komposisi ke `CustomUser`, `Meta.ordering` |
| `CustomUser(AbstractUser)` | `:178` | Pewarisan, property `nama_asli`, method perilaku |
| `Pengaturan` | `:262` | Singleton (`save()` dipaksa `pk = 1`), `@classmethod muat()` |
| `AchievementGambar` | `:331` | Property `ikon_tampil` (`:360`) |
| `AchievementForm(forms.ModelForm)` | `views_admin.py:471` | Pewarisan, overriding `__init__`, class `Meta` bersarang |
| `AchievementGambarAdmin(admin.ModelAdmin)` | `admin.py` | Pewarisan (panel `/admin`) |

### 4.1 Abstraksi: `ItemKatalog`

```python
class ItemKatalog(models.Model):
    id_item = models.CharField(max_length=20, unique=True, primary_key=True)
    judul = models.CharField(max_length=255)
    penulis = models.CharField(max_length=255)
    tahun_terbit = models.CharField(max_length=4, blank=True)
    cover = models.URLField(blank=True, null=True)
    tanggal_ditambahkan = models.DateTimeField(default=timezone.now)

    class Meta:
        abstract = True
```

- `abstract = True`: Django **tidak membuat tabel** untuk class ini dan class ini **tidak bisa dibuat objeknya langsung**. Field-nya disalin ke tabel `BukuDigital` dan `Jurnal`.
- Ini abstraksi gaya Django (abstract base model), bukan `abc.ABC` dengan `@abstractmethod`. Jelaskan seperti itu bila ditanya.
- `ItemKatalog` belum punya method, jadi ia **kerangka atribut**, bukan kerangka perilaku.
- Abstraksi lain: **ORM Django**. `BukuDigital.objects.filter(...)` menyembunyikan SQL.

### 4.2 Pewarisan

```python
class BukuDigital(ItemKatalog):
    kategori = models.CharField(max_length=50, default="", blank=True)
    isbn = models.CharField(max_length=20, blank=True, default="")
    link_unduh = models.URLField(default="https://example.com")
    ukuran_file = models.FloatField(default=2.5, help_text="dalam MB")
    format_file = models.CharField(max_length=10, default="PDF")
    deskripsi = models.TextField(blank=True, default="")

class Jurnal(ItemKatalog):
    kategori = models.CharField(max_length=50, default="", blank=True)
    issn = models.CharField(max_length=9, blank=True, default="")
    volume = models.CharField(max_length=50, blank=True)
    bidang_ilmu = models.CharField(max_length=100, blank=True)
    link_akses = models.URLField(default="https://example.com")
```

| Subclass | Induk | Yang diwarisi | Yang ditambah |
|---|---|---|---|
| `BukuDigital` | `ItemKatalog` | id_item, judul, penulis, tahun_terbit, cover, tanggal_ditambahkan | kategori, isbn, link_unduh, ukuran_file, format_file, deskripsi |
| `Jurnal` | `ItemKatalog` | atribut yang sama | kategori, issn, volume, bidang_ilmu, link_akses |
| `CustomUser` | `AbstractUser` | username, password, `check_password`, `set_password`, `is_active`, `is_staff` | is_admin, is_member, menunggu_persetujuan, nama_tampilan, foto_profil, OTP, exp, streak, max_loans, dan lainnya |
| `AchievementForm` | `forms.ModelForm` | `is_valid()`, `save()`, `cleaned_data` | class `Meta`, `__init__` yang ditimpa, `clean_*`, `_cek_ukuran` |
| `AchievementGambarAdmin` | `admin.ModelAdmin` | fitur admin bawaan | `list_display`, `list_filter`, `search_fields`, preview gambar |
| class tes | `django.test.TestCase` | client, database tes, assertion | tes khusus proyek |

### 4.3 Enkapsulasi: `CustomUser`

```python
def add_exp(self, points):
    self.exp += points
    self.save()

def update_streak(self):
    today = timezone.localdate()
    if (self.tanggal_baca_harian != today
        or self.detik_baca_hari_ini < TARGET_HARIAN_DETIK
        or self.last_read_date == today):
        return
    if self.last_read_date == today - timedelta(days=1):
        self.streak_days += 1
    elif not self.last_read_date or self.last_read_date < today - timedelta(days=1):
        self.streak_days = 1
    self.last_read_date = today
    self.save()

def reset_streak_if_expired(self):
    today = timezone.localdate()
    if (self.streak_days and self.last_read_date
            and self.last_read_date < today - timedelta(days=1)):
        self.streak_days = 0
        self.save(update_fields=["streak_days"])
```

Data (`exp`, `streak_days`, `last_read_date`) dan aturan yang mengubahnya berada di satu class. View cukup memanggil `user.update_streak()`.

**`@property` sebagai antarmuka baca**, dipakai lintas fitur:

```python
@property
def nama_asli(self):                 # models.py:210  (CustomUser)
    nama_lengkap = self.get_full_name().strip()
    if nama_lengkap:
        return nama_lengkap
    nama = (self.nama_tampilan or "").strip()
    if nama.startswith("User_") and nama[5:].isdigit():
        return self.username
    return nama or self.username

@property
def kategori_label(self):            # models.py:57  (BukuDigital)
    return label_kategori(self.kategori)
```

Template cukup menulis `{{ user.nama_asli }}` atau `{{ buku.kategori_label }}` tanpa tanda kurung dan tanpa tahu aturan di baliknya.

**Singleton `Pengaturan`:** `save()` memaksa `pk = 1` dan `muat()` (`@classmethod`) memakai `get_or_create(pk=1)`, jadi aturan "hanya satu baris pengaturan" dikunci di dalam class. (Lihat catatan jujur: `muat()` belum dipanggil di view mana pun.)

### 4.4 Polimorfisme: `__str__`

Semua model mewarisi `models.Model` dan menimpa `__str__`:

| Class | Hasil `str(objek)` |
|---|---|
| `BukuDigital` | `[Digital] <judul>` |
| `Jurnal` | `[Jurnal] <judul>` |
| `Peminjaman` | `<username> <item> (<status>)` |
| `Bookmark` | `<username> saved <item>` |
| `Notifikasi` | `<username>: <judul>` |
| `AchievementGambar` | `[<kategori>] <nama>` |
| `CustomUser` | `<username>` |
| `Pengaturan` | `Masa pinjam: <n> hari` |

Panggilannya sama (`str(obj)` atau `print(obj)`), hasilnya berbeda per class. Halaman `/admin` Django memanggilnya otomatis untuk menampilkan nama tiap data.

Polimorfisme yang terjadi **saat program berjalan** ada di `Peminjaman.__str__` (`models.py:104`):

```python
def __str__(self):
    item = self.buku_digital or self.jurnal
    return f"{self.user.username} {item} ({self.status})"
```

`item` bisa berisi `BukuDigital` **atau** `Jurnal`. `f"{item}"` memanggil `__str__` milik class yang sedang dipegang, tanpa memeriksa tipenya.

Contoh lain: `Pengaturan.save()` (`:272`) menimpa `save()` bawaan (memaksa satu baris, lalu `super().save()`), dan `AchievementForm.__init__` (bagian 6, fitur A4).

**Contoh yang lemah:** `CustomUser.save()` (`:207`) hanya berisi `super().save(*args, **kwargs)`. Secara teknis overriding, tetapi perilakunya belum berbeda. Jangan jadikan contoh utama.

---

## 5. OOP per Fitur: User

Format tiap fitur: **Di website**, **Bukti** (tabel konsep dan lokasi kode), **Kalimat presentasi**, **Jangan klaim**.

### U1. Login (dan Lupa Password)

**Di website:** form "NIM" dan "Password", "Ingat Saya", "Lupa Password?". Lupa password membuka alur OTP lewat email: masukkan email, terima kode, atur ulang password. Kata "NIM" di layar adalah field `username` di kode.

**Bukti:**

```python
user = authenticate(request, username=username_input, password=password_input)   # views.py:283
...
kandidat = CustomUser.objects.filter(username=username_input or "").first()
if kandidat and not kandidat.is_active and kandidat.check_password(password_input or ""):  # :297
```

| Konsep | Bukti |
|---|---|
| Object | `authenticate(...)` mengembalikan object `CustomUser`, karena `AUTH_USER_MODEL = 'perpustakaan.CustomUser'` (`config/settings.py:101`) |
| Pewarisan | `is_active`, `check_password()` (`views.py:297`) diwarisi dari `AbstractUser`, tidak ditulis di `models.py` |
| Atribut tambahan | `menunggu_persetujuan` untuk akun yang belum disetujui admin |
| Atribut tambahan | `otp_code`, `otp_created_at` di `CustomUser`; dipakai alur reset di `_proses_reset` (`views.py:88`), dengan konstanta `OTP_BERLAKU_DETIK = 300`, `OTP_JEDA_KIRIM_ULANG = 60`, `OTP_MAKS_SALAH = 5` (`:44-46`) |
| Method warisan | `user.set_password(baru)` saat reset (`views.py:194`) |

**Kalimat presentasi:** "Login memakai *inheritance*. `CustomUser` mewarisi `AbstractUser`, jadi autentikasi, `check_password`, dan `is_active` kami dapat dari kelas induk. Lalu kami menambah atribut seperti `menunggu_persetujuan` dan field OTP."

**Jangan klaim:** enkapsulasi, polimorfisme, atau abstraksi di Login. `login_view` dan `_proses_reset` adalah fungsi biasa.

### U2. Register

**Di website:** form NIM, nama tampilan, email, password, dan konfirmasi. Akun baru bisa berstatus menunggu persetujuan admin.

**Bukti** (`register_view`, `views.py:307`):

| Konsep | Bukti |
|---|---|
| Class dan object | `CustomUser.objects.filter(username=nim).first()` (`:332`) dan `filter(email__iexact=email).exists()` (`:341`) untuk cek duplikat |
| Pembuatan object | `CustomUser.objects.create_user(...)` (`:353`) |
| Pewarisan | `create_user` (hash password otomatis) berasal dari `AbstractUser`. `nama_tampilan` adalah atribut tambahan `CustomUser` |

**Kalimat presentasi:** "Register membuat object `CustomUser`. Penyimpanan password yang aman kami dapat dari `AbstractUser`, lalu kami tambah atribut `nama_tampilan`."

**Jangan klaim:** `validasi_password` (`views.py:55`) sebagai OOP. Itu fungsi biasa.

### U3. Dashboard

**Di website:** sapaan nama, peringkat, "Bacaanmu" (bacaan terakhir dan sisa hari), banner streak, jumlah sedang dipinjam, total dibaca, bookmarks, dan "Rekomendasi Hari Ini".

**Bukti** (`dashboard`, `views.py:375`):

| Konsep | Bukti |
|---|---|
| Enkapsulasi | `user.reset_streak_if_expired()` (`:377`, definisi `models.py:246`). `buku.kategori_label` (`:421`). `{{ user.nama_asli }}` di template |
| Asosiasi | `Peminjaman.objects.filter(user=user, status="Dibaca")` (`:379`). Bacaan terakhir lewat `.select_related("buku_digital")`. `Bookmark.objects.filter(user=user).count()` (`:439`) |
| Class dan object | `BukuDigital.objects.order_by("id_item")` (`:449`) untuk rekomendasi |
| Method warisan | `p.save(update_fields=["tanggal_kembali"])` (`:412`) |

**Kalimat presentasi:** "Dashboard tidak menghitung streak sendiri. Ia memanggil `user.reset_streak_if_expired()`, karena aturannya dienkapsulasi di class `CustomUser`."

**Jangan klaim:** `bacaan_terakhir` sebagai object (itu dict yang membungkus object). Peringkat (`:383`) dihitung di view, bukan method class.

### U4. Katalog Buku

**Di website:** daftar buku, filter kategori (program studi), pencarian, dan tombol bookmark. Sebagian buku dari database lokal, sebagian dari Open Library lewat JavaScript.

**Bukti** (`katalog_view`, `views.py:462`):

```python
buku_queryset = BukuDigital.objects.all().distinct()           # :466
...
'kategori': buku.kategori_label,                                # :515
'subjek': [buku.kategori_label] if buku.kategori else [],       # :516
```

| Konsep | Bukti |
|---|---|
| Class dan object | `BukuDigital.objects.all()` (`:466`). List `buku_lokal` dibentuk dari object itu |
| Abstraksi | ORM: `filter(Q(judul__icontains=...) | Q(penulis__icontains=...) ...)` tanpa satu baris SQL |
| Pewarisan | `id_item`, `judul`, `penulis`, `tahun_terbit`, `cover` diwarisi dari `ItemKatalog`. `deskripsi` dan `kategori` milik `BukuDigital` |
| Enkapsulasi | `buku.kategori_label` memanggil property (`models.py:57`) |
| Asosiasi | `Bookmark.objects.filter(user=request.user, buku_digital__isnull=False)` (`:499`) |
| Pembuatan object | Buku Open Library menjadi `BukuDigital.objects.create(...)` di `_pastikan_buku` (`:547`) |

**Kalimat presentasi:** "Setiap buku lokal di katalog adalah object `BukuDigital` yang mewarisi `ItemKatalog`. Label kategorinya lewat property `kategori_label`, jadi aturan konversinya tidak ditulis ulang di view."

**Jangan klaim:** hasil Open Library yang tampil lewat JavaScript (dict tanpa class).

### U5. Detail Buku dan Membaca

**Di website:** halaman detail (info buku, tombol pinjam, bookmark), lalu halaman baca layar penuh dengan PDF.js. Posisi terakhir dibaca tersimpan.

**Bukti:**

| Konsep | Bukti |
|---|---|
| Class dan object | `BukuDigital.objects.filter(pk=olid).first()` (`buku_detail`, `:559`). `get_object_or_404(BukuDigital, id_item=olid)` (`baca_buku`, `:599`) |
| Asosiasi | `Peminjaman.objects.get_or_create(user=..., buku_digital=buku, status="Dibaca", ...)` (`:602`) menghubungkan user dengan buku |
| Method warisan | `peminjaman.save(update_fields=["tanggal_pinjam"])`; `simpan_progress` menyimpan `progress_baca` |
| Atribut tertentu | `user.max_loans` dipakai untuk menentukan slot penuh (`:591-592`) |

**Kalimat presentasi:** "Membaca buku berarti ada object `Peminjaman` yang menghubungkan `CustomUser` dengan `BukuDigital`. Progres bacanya disimpan sebagai atribut object itu."

**Jangan klaim:** PDF.js, `buku.js`, dan `baca-ring.js` sebagai OOP. Itu JavaScript tanpa class. Validasi `ITEM_ID_RE` (`views.py:38`, pola `^[\w-]+$`) adalah regex biasa.

### U6. Peminjaman (pinjam, perpanjang, kembalikan)

**Di website:** daftar buku yang sedang dipinjam, jatuh tempo, sisa slot, tombol perpanjang dan kembalikan.

**Bukti:**

```python
Peminjaman.objects.create(user=user, buku_digital=buku)             # pinjam_buku, :661

pinjaman = get_object_or_404(Peminjaman, id=pinjaman_id, user=request.user)  # kembalikan_buku, :821
pinjaman.status = "Selesai"
pinjaman.progress_baca = 100
pinjaman.tanggal_kembali = timezone.now()
pinjaman.save()
```

| Konsep | Bukti |
|---|---|
| Pembuatan object | `Peminjaman.objects.create(...)` (`:661`) |
| Asosiasi | satu `Peminjaman` menunjuk ke `CustomUser` dan ke `BukuDigital` atau `Jurnal` |
| Pola object | diambil, diubah atributnya, lalu `save()` (`kembalikan_buku`, `perpanjang_buku`) |
| Batas pinjam | memakai atribut `user.max_loans` (`:652`) dan konstanta `MAKS_PERPANJANG = 1`, `MASA_PERPANJANG_HARI = 7`, `MASA_PINJAM_HARI = 14` (`:31-33`) |
| Keamanan kepemilikan | `get_object_or_404(..., user=request.user)` hanya menemukan pinjaman milik user yang login |

**Kalimat presentasi:** "Siklus pinjam adalah siklus hidup object `Peminjaman`: dibuat saat meminjam, diubah statusnya jadi `Selesai` saat dikembalikan, dan disimpan lewat `save()` warisan dari `Model`."

**Jangan klaim:** `pinjaman.jatuh_tempo` sebagai field class. Atribut itu **ditempel saat runtime** di view (`peminjaman_view`, `:794`). `_jatuh_tempo` (`:202`) adalah fungsi biasa. `kembalikan_buku` tidak memakai `@require_POST` dan tidak menambah EXP.

### U7. E-Journal

**Di website:** daftar jurnal (cover, judul, penulis, tahun), pencarian, dan tombol baca. Sebagian jurnal dari database, sebagian dari berkas `jurnal_teknik.json`.

**Bukti** (`ejournal_view`, `views.py:684`):

| Konsep | Bukti |
|---|---|
| Class dan object | `Jurnal.objects.order_by("judul", "id_item")` (`:687`). `get_object_or_404(Jurnal, pk=jurnal_id)` di `baca_jurnal` |
| Pewarisan | `judul`, `penulis`, `cover`, `tahun_terbit` diwarisi dari `ItemKatalog`. `issn`, `volume`, `bidang_ilmu`, `link_akses` milik `Jurnal` (`models.py:69`) |
| Polimorfisme | `Jurnal.__str__` menghasilkan `[Jurnal] ...`, `BukuDigital.__str__` menghasilkan `[Digital] ...`. **Tidak terlihat di halaman**; demokan lewat `python manage.py shell` |

**Kalimat presentasi:** "Jurnal adalah subclass `ItemKatalog` seperti buku. Atribut dasarnya sama, tetapi `Jurnal` menambah ISSN, volume, dan bidang ilmu."

**Jangan klaim:** daftar yang tampil berbentuk dict (`baca_url`, `:695`). Entri `JSON-{index}` (`baca_jurnal`) juga bukan object model.

### U8. Bookmarks

**Di website:** koleksi buku dan jurnal tersimpan, hapus bookmark, paginasi, tombol hati di katalog.

**Bukti** (`bookmarks_view`, `views.py:877`):

| Konsep | Bukti |
|---|---|
| Asosiasi | `Bookmark` punya `ForeignKey` ke user, `buku_digital`, dan `jurnal` (`models.py:111-121`). `Bookmark.objects.filter(user=request.user)` (`:879`). Template: `mark.buku_digital.judul`, `mark.jurnal.judul` |
| Enkapsulasi | `mark.buku_digital.kategori_label` di `bookmarks.html` |
| Method warisan | `get_or_create` (`toggle_bookmark`, `:679`), `bookmark.delete()` (`hapus_bookmark_api`, `:906`) |
| Pewarisan | buku dan jurnal tampil dengan atribut warisan yang sama (`judul`, `penulis`, `cover`) |

**Kalimat presentasi:** "Bookmark adalah object yang menghubungkan user dengan buku atau jurnal lewat `ForeignKey`. Datanya tidak disalin, melainkan diakses lewat relasi."

**Jangan klaim:** percabangan `{% if mark.buku_digital %}` di template sebagai polimorfisme. Itu hanya kondisi.

### U9. Riwayat Peminjaman

**Di website:** semua pinjaman milik user (aktif dan selesai), dengan tanggal pinjam dan jatuh tempo.

**Bukti** (`riwayat_view`, `views.py:916`):

| Konsep | Bukti |
|---|---|
| Asosiasi | `Peminjaman.objects.filter(user=request.user).order_by("-tanggal_pinjam")` (`:917`) |
| Class dan object | object `Peminjaman` dibaca atributnya: `status`, `tanggal_pinjam`, `tanggal_kembali` |

**Kalimat presentasi:** "Riwayat adalah kumpulan object `Peminjaman` milik satu user. Karena `Peminjaman.buku_digital` memakai `SET_NULL`, riwayat tetap ada walau bukunya dihapus admin."

**Jangan klaim:** komentar di `:919` menyebut "classmethod Pengaturan", padahal kodenya **tidak memanggil** `Pengaturan.muat()`. Jangan sebut fitur ini memakai `@classmethod`.

### U10. Top List Reader (Leaderboard)

**Di website:** podium 3 teratas dan tabel peringkat 4 sampai 10 (Rank, Pengguna, Streak, Total EXP).

**Bukti** (`leaderboard_view`, `views.py:930`):

| Konsep | Bukti |
|---|---|
| Class dan object | `CustomUser.objects.filter(is_member=True, is_admin=False, ...).order_by("-exp", "pk")` (`:931`) |
| Enkapsulasi | `podium.rank1.nama_asli` di template |
| Pewarisan | `exp`, `streak_days`, `foto_profil` adalah atribut tambahan `CustomUser` di atas `AbstractUser` |

**Kalimat presentasi:** "Papan peringkat memakai object `CustomUser`. Nama yang tampil lewat property `nama_asli`, jadi aturan pemilihan nama tidak ditulis ulang di template."

**Jangan klaim:** persen bar (`:940`) dan peringkat sebagai method class. Itu logika di view.

### U11. Profil

**Di website:** informasi pribadi, foto profil, background cover, ganti password, level, EXP, streak, achievement yang disematkan, dan profil publik user lain.

**Bukti** (`profil_view`, `views.py:957`):

| Konsep | Bukti |
|---|---|
| Pewarisan | `user.check_password(...)` (`:1022`) dan `user.set_password(...)` (`:1023`) dari `AbstractUser`. `user.save()` dari `Model` |
| Enkapsulasi | `user.nama_asli`; `user.update_streak()` (`waktu_baca`, `:1311`) |
| Object dari field | `user.foto_profil.delete(save=False)` dan `.url`. `ImageField` menghasilkan object file dengan method sendiri |
| Class dan object | `get_object_or_404(CustomUser, username=username)` (`profil_user_detail_view`, `:1190`) |

**Kalimat presentasi:** "Ganti password memakai method `set_password()` warisan dari `AbstractUser`. Foto profil adalah field `ImageField` yang menghasilkan object file."

**Jangan klaim:** level (`user.exp // LEVEL_EXP + 1`, `:1035`, `LEVEL_EXP = 100` di `:1073`) sebagai property. Itu dihitung di view. `_achievements` (`:1075`) adalah fungsi.

### U12. Waktu Baca, EXP, dan Streak

**Di website:** cincin waktu baca yang terisi saat membaca, EXP bertambah, streak harian (target 30 menit), notifikasi level naik dan peringkat naik.

**Ini fitur dengan enkapsulasi paling jelas di sisi user.** `waktu_baca` (`views.py:1262`) memanggil method milik object:

```python
user.reset_streak_if_expired()         # :1287
...
user.exp += exp_gain                   # :1299
...
user.update_streak()                   # :1311
```

| Konsep | Bukti |
|---|---|
| Enkapsulasi | `reset_streak_if_expired()` dan `update_streak()` (`models.py:246, 226`). Aturan streak tidak ditulis ulang di view |
| Konstanta | `TARGET_HARIAN_DETIK = 30 * 60` (`models.py`), `RING_DETIK_PENUH = 120`, `RING_EXP = 10` (`views.py:34-35`); cocok dengan `data-total="120"` di `baca.html` |
| Asosiasi | `Peminjaman` ikut diperbarui (`waktu_baca_detik`) dan `Notifikasi` dibuat lewat `_notifikasi` |
| Dependensi | view memakai `CustomUser` dan `_notifikasi` sementara selama fungsi berjalan |

**Kalimat presentasi:** "Streak dihitung oleh method `update_streak()` di class `CustomUser`. View `waktu_baca` hanya memanggilnya, jadi kalau aturan streak berubah, cukup diubah di satu tempat."

**Jangan klaim:** `add_exp()` dipakai di fitur ini. EXP ditambah langsung (`user.exp += exp_gain`, `:1299`), bukan lewat `add_exp()`. Anti-curang `session["ring_terakhir"]` adalah logika fungsi.

### U13. Achievement

**Di website:** daftar achievement terkunci dan terbuka (gambar dari admin), dan user bisa menyematkan sampai 5 achievement yang sudah terbuka di profil.

**Bukti** (`achievement_view`, `views.py:1327`):

| Konsep | Bukti |
|---|---|
| Class dan object | object `AchievementGambar` dibaca atributnya (`nama`, `kategori`, `tipe_syarat`, `target`) di `_achievements` (`:1075`) |
| Enkapsulasi | property `ikon_tampil` (`models.py:360`): field `ikon` dulu, lalu kamus nama, lalu kosong |
| Method warisan | `request.user.save(update_fields=["pinned_achievements"])` (`:1359`) |

**Kalimat presentasi:** "Ikon achievement ditentukan oleh property `ikon_tampil` di class `AchievementGambar`, jadi aturan pemilihan ikon ada di model, bukan di template."

**Jangan klaim:** OOP yang kaya. Logika syarat ada di fungsi `_achievements`, bukan di method class. Model `Achievement` dan `UserAchievement` **tidak dipakai** fitur ini (lihat bagian 10).

### U14. Notifikasi

**Di website:** lonceng notifikasi di pojok atas, halaman semua notifikasi, tandai dibaca, hapus semua. Pengingat jatuh tempo dan target harian dibuat otomatis.

**Bukti** (`notifikasi_view`, `views.py:216`):

| Konsep | Bukti |
|---|---|
| Asosiasi | `Notifikasi.user = ForeignKey(CustomUser, related_name="notifikasi")` (`models.py:161`) |
| Class `Meta` bersarang | `ordering = ["-dibuat", "-pk"]` (`models.py:170`) |
| Method warisan | `item.save(update_fields=["dibaca"])` (`buka_notifikasi`, `:258`), `Notifikasi.objects.filter(...).delete()` |
| Object | `_notifikasi(user, jenis, judul, isi, url)` (`:211`) membuat object `Notifikasi` dari banyak fitur |

**Kalimat presentasi:** "Tiap notifikasi adalah object yang terhubung ke satu `CustomUser`. Kalau user dihapus, notifikasinya ikut terhapus (`CASCADE`)."

**Jangan klaim:** `_notifikasi` sebagai method class. Itu fungsi pembantu.

### Logout

Tidak ada OOP khusus.

### Ringkasan OOP per halaman user

| Halaman | Konsep OOP yang sah |
|---|---|
| Login, Register | Pewarisan (`CustomUser` ← `AbstractUser`), pembuatan object |
| Dashboard | Enkapsulasi (`reset_streak_if_expired`, `kategori_label`), asosiasi |
| Katalog | Pewarisan, abstraksi (ORM), enkapsulasi, asosiasi |
| Detail dan Baca | Asosiasi (`Peminjaman`), method warisan |
| Peminjaman | Siklus hidup object `Peminjaman`, asosiasi |
| E-Journal | Pewarisan (`Jurnal` ← `ItemKatalog`) |
| Bookmarks | Asosiasi, enkapsulasi |
| Riwayat | Asosiasi, `SET_NULL` (agregasi) |
| Top Reader | Object `CustomUser`, enkapsulasi (`nama_asli`) |
| Profil | Pewarisan (`set_password`), enkapsulasi |
| Waktu baca dan streak | **Enkapsulasi paling jelas** (`update_streak`), dependensi |
| Achievement | Enkapsulasi (`ikon_tampil`), OOP minim |
| Notifikasi | Asosiasi/komposisi, class `Meta` bersarang |

---

## 6. OOP per Fitur: Admin

*Semua halaman admin dilindungi `@user_passes_test(_boleh_admin, login_url="login")` (`views_admin.py:38`), yaitu decorator fungsi, bukan OOP. Tampilan memakai `{% extends 'perpustakaan/base_admin.html' %}`, yaitu fitur template Django; sebut hanya sebagai analogi pewarisan.*

### A1. Dashboard Admin

**Di website:** total anggota, total buku, sedang dipinjam, dikembalikan minggu ini, grafik 7 hari, aktivitas terbaru, buku terpopuler.

**Bukti** (`dashboard_admin`, `views_admin.py:158`):

| Konsep | Bukti |
|---|---|
| Class dan object | `BukuDigital.objects.count()` dan `Jurnal.objects.count()` |
| Asosiasi | `BukuDigital.objects.annotate(jumlah=Count("peminjaman"))` (`:170`) memakai relasi balik ke `Peminjaman` |
| Pewarisan | `_judul_item` (`:58`) memakai `item.judul`, atribut yang diwarisi dari `ItemKatalog` |
| Enkapsulasi | `_nama_pengguna` mengembalikan `user.nama_asli` (`:63-64`) |

**Kalimat presentasi:** "Dashboard admin mengambil statistik dari object model dan memanfaatkan relasi `Peminjaman` ke `BukuDigital` untuk menghitung buku terpopuler."

**Jangan klaim:** `_aktivitas_terbaru` (`:67`), `_data_grafik_7_hari` (`:118`), `_waktu_lalu` (`:43`) sebagai OOP. Itu fungsi pembantu.

### A2. Kelola Buku

**Di website:** form tambah buku (judul, penulis, kategori, tahun, ISBN, cover, deskripsi, PDF), tabel buku, edit, hapus, dan hapus massal lewat checkbox.

**Bukti** (`kelola_buku`, `views_admin.py:209`):

| Konsep | Bukti |
|---|---|
| Pembuatan object | `BukuDigital.objects.create(...)` (`:232`) dengan atribut warisan dan atribut khusus (`isbn`, `link_unduh`, `ukuran_file`, `format_file`, `deskripsi`) |
| Pewarisan | bandingkan dengan `Jurnal.objects.create` (`:353`): atribut dasar sama, tetapi jurnal punya `issn`, `volume`, `bidang_ilmu`, `link_akses`. Bisa didemokan dari dua form di UI |
| Enkapsulasi | `buku.kategori_label` di teks notifikasi (`:248`) dan di tabel |
| Method warisan | `buku.save()` (`edit_buku`, `:306`), `buku.delete()` (`hapus_buku`, `:274`), `qs.delete()` untuk hapus massal (`:316`) |
| Asosiasi | satu `Notifikasi` dibuat per anggota saat buku baru masuk (`:247`) |

**Kalimat presentasi:** "Admin membuat object `BukuDigital`, subclass `ItemKatalog`. Field umum diwarisi, ISBN khusus buku. Edit dan hapus memakai `save()` dan `delete()` warisan dari `Model`."

**Jangan klaim:** `b.total_pembaca` (`:261`) sebagai atribut class. Itu atribut yang **ditempel saat runtime**.

### A3. Kelola Jurnal

**Di website:** form tambah jurnal (judul, penulis, bidang ilmu, volume, tahun, cover, link akses, ISSN, PDF), tabel, edit, hapus, hapus massal.

**Bukti** (`kelola_jurnal`, `views_admin.py:326`):

| Konsep | Bukti |
|---|---|
| Pembuatan object | `Jurnal.objects.create(...)` (`:353`) |
| Pewarisan | field dasar sama dengan buku; field khusus `issn`, `volume`, `bidang_ilmu`, `link_akses` |
| Method warisan | `jurnal.save()` (`:448`), `jurnal.delete()` (`:414`), `qs.delete()` (`:458`) |
| Asosiasi | `Notifikasi.objects.create(...)` per anggota (`:365`) |

**Kalimat presentasi:** "Jurnal dan buku berbagi induk `ItemKatalog`, jadi form admin keduanya punya field dasar yang sama, dan hanya field khususnya yang berbeda."

**Jangan klaim:** `j.total_pembaca` (`:379`) sebagai atribut class.

### A4. Kelola Achievement

**Di website:** form tambah achievement (nama, kategori Umum/Langka/Epik, deskripsi, jenis syarat, target, gambar terkunci dan terbuka), daftar, edit, hapus, hapus massal.

**Ini fitur dengan OOP paling kaya di sisi admin.** Class `AchievementForm` (`views_admin.py:471`):

```python
class AchievementForm(forms.ModelForm):
    class Meta:
        model = AchievementGambar
        fields = ["nama", "kategori", "deskripsi", "tipe_syarat", "target",
                  "gambar_terkunci", "gambar_terbuka"]

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields["nama"].error_messages["required"] = "wajib diisi."
        ...

    def clean_nama(self):
        nama = self.cleaned_data["nama"].strip()
        ada = AchievementGambar.objects.filter(nama__iexact=nama)
        ...

    def _cek_ukuran(self, field):
        berkas = self.cleaned_data.get(field)
        ...

    def clean_gambar_terkunci(self):
        return self._cek_ukuran("gambar_terkunci")
```

| Konsep | Bukti |
|---|---|
| Pewarisan | `AchievementForm(forms.ModelForm)` (`:471`) |
| Class bersarang | `class Meta` menentukan `model = AchievementGambar` dan daftar `fields` |
| Overriding (sah) | `__init__` ditimpa dan memanggil `super().__init__(...)` untuk mengubah pesan error |
| Enkapsulasi | validasi berada di dalam class: `clean_nama`, `clean_target`, `clean_gambar_terkunci`, `clean_gambar_terbuka`; helper `_cek_ukuran` berawalan `_` (internal menurut konvensi) |
| Method warisan | `form.is_valid()` (`:543`), `form.save()` (`:544, 583`), `ach.delete()` (`:600`) |
| Property | `a.ikon_tampil` (`models.py:360`) di `kelola_achievement.html` |
| Keterkaitan class | `AchievementForm(request.POST, request.FILES, instance=ach)` (`edit_achievement`, `:581`) mengikat form ke object `AchievementGambar` |

**Kalimat presentasi:** "Form achievement mewarisi `ModelForm`, meng-override `__init__` dengan `super()`, dan menyimpan aturan validasinya di dalam class itu sendiri."

**Jangan klaim:** `clean_*` sebagai overriding. Itu *hook* yang dipanggil Django berdasarkan nama field (`ModelForm` tidak punya method `clean_nama`). Overriding yang sah hanya `__init__`. `_galat_form` (`:525`) dan `_hapus_berkas` (`:533`) adalah fungsi biasa.

### A5. Kelola Anggota

**Di website:** daftar anggota (aktif dan nonaktif), persetujuan atau penolakan akun baru, setujui semua, tambah anggota, edit, hapus, aktifkan kembali.

**Bukti** (`kelola_anggota`, `views_admin.py:674`):

| Konsep | Bukti |
|---|---|
| Instansiasi | `CustomUser(...)` dibuat langsung (`:689`) |
| Pewarisan | `user.set_password(password)` (`:696`); `is_active` diwarisi; `menunggu_persetujuan` milik `CustomUser` |
| Method warisan | `user.save(update_fields=["is_active", "menunggu_persetujuan"])` (`setujui_anggota`, `:723`), `user.delete()` (`hapus_anggota`, `:744`) |
| Enkapsulasi | `user.nama_asli` (`:733, 743`) |
| Asosiasi | `Notifikasi.objects.create(...)` ke user saat disetujui (`:724`) |

**Kalimat presentasi:** "Persetujuan anggota hanya mengubah *state* object `CustomUser` lewat `save()`, lalu membuat object `Notifikasi` yang terhubung ke user itu."

**Jangan klaim:** `a.jumlah_pinjaman` (`:705`) sebagai atribut class. Itu ditempel saat runtime.

### A6. Peminjaman (admin)

**Di website:** tabel semua pinjaman (anggota, buku, tanggal pinjam, jatuh tempo, status, sudah dikembalikan atau telat) dengan filter.

**Bukti** (`peminjaman_admin`, `views_admin.py:896`):

| Konsep | Bukti |
|---|---|
| Asosiasi | `Peminjaman.objects.select_related(_USER_FIELD, "buku_digital", "jurnal")` (`:899`). Template: `p.pengguna.nama_asli`, `p.item.judul` |
| Pewarisan | `p.item = p.buku_digital or p.jurnal` (`:905`), lalu `p.item.judul` dan `p.item.penulis` memakai atribut warisan `ItemKatalog` |
| Enkapsulasi | `nama_asli` pada anggota |

**Kalimat presentasi:** "Satu object `Peminjaman` terhubung ke user dan ke buku atau jurnal, lalu kami membaca judulnya lewat atribut warisan `ItemKatalog`."

**Jangan klaim:** `p.is_kembali` (`:906`) dan `p.is_telat` (`:912`) sebagai atribut class; keduanya ditempel saat runtime. `_hitung_jatuh_tempo` (`:845`) adalah fungsi. `buku_digital or jurnal` bukan polimorfisme murni; ia bekerja karena keduanya mewarisi `ItemKatalog` (kesamaan atribut), jadi sebut sebagai manfaat pewarisan.

### A7. Laporan

**Di website:** kartu statistik (anggota aktif, buku dan jurnal tersedia, peminjaman bulan ini), "Buku per kategori", dan unduh CSV.

**OOP di halaman ini sedikit, jangan dipaksakan:**

| Konsep | Bukti |
|---|---|
| Class dan object | `CustomUser.objects.filter(...)`, `Peminjaman.objects.filter(...)`, `Jurnal.objects.count()`, `BukuDigital.objects.count()` (`laporan`, `:923`) |
| Object library | `HttpResponse(content_type="text/csv")` dan `csv.writer(response)` (`unduh_laporan`, `:986-992`) |

**Kalimat presentasi:** "Laporan mengagregasi data dari object model dan menghasilkan CSV memakai object `HttpResponse` dan `csv.writer`."

**Jangan klaim:** pewarisan, enkapsulasi, atau polimorfisme di Laporan.

### A8. Pengumuman

**Di website:** admin mengisi jenis (pengumuman atau pilihan gamifikasi), judul, isi, dan tautan opsional. Notifikasi terkirim ke semua anggota.

**Bukti** (`buat_pengumuman`, `views_admin.py:389`):

| Konsep | Bukti |
|---|---|
| Class dan object | `CustomUser.objects.filter(is_member=True, is_admin=False)` untuk menentukan penerima |
| Asosiasi | `Notifikasi.objects.bulk_create([Notifikasi(user=user, ...) for user in anggota])`: satu object `Notifikasi` per anggota, tiap object terhubung ke satu `CustomUser` |

**Kalimat presentasi:** "Pengumuman admin membuat banyak object `Notifikasi`, satu untuk tiap anggota, dan masing-masing terhubung ke pemiliknya lewat `ForeignKey`."

**Jangan klaim:** polimorfisme atau enkapsulasi. Ini pembuatan object massal.

### Ringkasan OOP per halaman admin

| Halaman | Konsep OOP yang sah |
|---|---|
| Dashboard Admin | Asosiasi (relasi balik), pewarisan atribut |
| Kelola Buku, Kelola Jurnal | Pewarisan, pembuatan object, method warisan, asosiasi |
| **Kelola Achievement** | **Pewarisan, overriding `__init__`, class bersarang, enkapsulasi validasi** |
| Kelola Anggota | Instansiasi, method warisan, asosiasi |
| Peminjaman Admin | Asosiasi, pewarisan atribut |
| Laporan | Object (OOP minim) |
| Pengumuman | Asosiasi, pembuatan object massal |

### Pilar paling kuat untuk presentasi

1. **Pewarisan:** `ItemKatalog` → `BukuDigital` / `Jurnal`; `CustomUser` ← `AbstractUser`; `AchievementForm` ← `ModelForm`.
2. **Abstraksi:** `ItemKatalog` dengan `Meta.abstract = True`, dan ORM.
3. **Enkapsulasi:** `update_streak()` dan `reset_streak_if_expired()`, `nama_asli`, `kategori_label`, `ikon_tampil`, dan validasi di `AchievementForm`.
4. **Polimorfisme:** `__str__` yang di-override di banyak model, `Peminjaman.__str__`, `Pengaturan.save()`, `AchievementForm.__init__`.
5. **Asosiasi:** `Peminjaman`, `Bookmark`, `Notifikasi` yang terhubung ke `CustomUser`.

### Peta cepat: pilar → fitur

| Pilar | Fitur tempat terlihat |
|---|---|
| Pewarisan | Login (U1), Katalog (U4), E-Journal (U7), Kelola Buku/Jurnal (A2, A3), Kelola Achievement (A4) |
| Abstraksi | Katalog (U4, ORM dan `ItemKatalog`) |
| Enkapsulasi | Dashboard (U3), Waktu baca dan streak (U12), Top Reader (U10), Achievement (U13), Kelola Achievement (A4) |
| Polimorfisme | `__str__` di semua model (demo lewat `manage.py shell` atau panel `/admin`), Kelola Achievement (A4) |
| Asosiasi | Bookmarks (U8), Peminjaman (U6), Notifikasi (U14), Peminjaman admin (A6), Pengumuman (A8) |
| Agregasi | Riwayat (U9): `Peminjaman.buku_digital` `SET_NULL` |
| Komposisi | Notifikasi (U14): `Notifikasi.user` `CASCADE` |
| Dependensi | Waktu baca (U12): view memakai `CustomUser` dan `_notifikasi` sementara |

---

## 7. Relasi Antarobjek di Proyek

Django mewujudkan relasi lewat `ForeignKey` dan `on_delete`. `SET_NULL` (dengan `null=True`) mengarah ke **agregasi**, `CASCADE` mengarah ke **komposisi**.

> Pemetaan ini adalah **tafsiran konseptual**. Django tidak punya kata kunci "agregasi" atau "komposisi", dan komentar di `models.py` hanya menyebut "asosiasi antarobjek". Jelaskan sebagai **perilaku** hasil `on_delete`.

### Asosiasi: `Peminjaman` dan `Bookmark`

```python
class Peminjaman(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    buku_digital = models.ForeignKey(BukuDigital, null=True, blank=True, on_delete=models.SET_NULL)
    jurnal = models.ForeignKey(Jurnal, null=True, blank=True, on_delete=models.SET_NULL)

class Bookmark(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    buku_digital = models.ForeignKey(BukuDigital, null=True, blank=True, on_delete=models.CASCADE)
    jurnal = models.ForeignKey(Jurnal, null=True, blank=True, on_delete=models.CASCADE)
```

Keduanya menghubungkan `CustomUser` dengan item katalog lewat **referensi**, tanpa menyalin data. Keduanya membawa atribut sendiri (`status`, `progress_baca`, `tanggal_pinjam`), jadi dalam UML disebut **kelas asosiasi**. Terlihat di fitur U6, U8, A6.

### Agregasi: `Peminjaman` → `BukuDigital` / `Jurnal`

```python
buku.delete()
# Peminjaman TETAP ada; kolom buku_digital menjadi NULL
```

Buku hidup sendiri di katalog. Buku dihapus admin, riwayat peminjaman tetap tersimpan. Terlihat di fitur U9 (Riwayat) dan A2 (hapus buku).

### Komposisi: `Notifikasi` → `CustomUser`

```python
class Notifikasi(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE,
                             related_name="notifikasi")

user.delete()          # semua notifikasi milik user ikut terhapus
user.notifikasi.all()  # jalan balik lewat related_name
```

Notifikasi hanya bermakna bagi satu user dan siklus hidupnya mengikuti user itu. `Peminjaman.user` dan `Bookmark.user` juga `CASCADE`. Terlihat di fitur U14 dan A5 (hapus anggota).

**Batasan jujur:** komposisi murni berarti bagian tidak punya identitas sendiri, sedangkan di Django tiap model tetap punya id dan tabel sendiri. Sebut sebagai "mendekati komposisi".

`UserAchievement.user` (`models.py:147`) juga `CASCADE`, tetapi model ini tidak dipakai view mana pun (hanya terdaftar di `admin.py`). Sah sebagai contoh di tingkat model, jangan dikatakan "dipakai di fitur".

### Dependensi: view memakai model

`waktu_baca` (U12) hanya **memakai** `CustomUser`, `update_streak()`, dan `_notifikasi()` selama fungsinya berjalan, tanpa menyimpannya sebagai atribut. View bergantung pada model tetapi tidak memilikinya.

### Pemetaan cepat

| Relasi di kode | `on_delete` | Jenis |
|---|---|---|
| `Peminjaman` → `BukuDigital` / `Jurnal` | `SET_NULL` | Agregasi |
| `Peminjaman` → `CustomUser` | `CASCADE` | Asosiasi, mendekati komposisi |
| `Notifikasi` → `CustomUser` | `CASCADE` | Komposisi (mendekati) |
| `UserAchievement` → `CustomUser` | `CASCADE` | Komposisi (tingkat model saja) |
| `Bookmark` → user dan item | `CASCADE` | Asosiasi |
| `waktu_baca` memakai `CustomUser` | tidak ada | Dependensi |

- `CASCADE`: data induk dihapus → data terkait ikut dihapus.
- `SET_NULL`: data induk dihapus → kolom relasinya dikosongkan.
- Saat buku dihapus: `Peminjaman` tetap ada (kolom buku kosong), `Bookmark` buku itu ikut terhapus.

---

## 8. Alur Program (Ringkas)

### 8.1 Siklus satu request (MTV)

```
Browser  →  config/urls.py (admin/, simpan_progress, include perpustakaan.urls)
         →  perpustakaan/urls.py (cocokkan alamat ke view)
         →  views.py / views_admin.py (cek login, baca parameter, ambil data lewat ORM)
         →  models.py → db.sqlite3
         →  render(request, "...html", context)  →  Template (DTL)  →  Browser
```

| Lapisan | Tugas | File |
|---|---|---|
| **M**odel | struktur data dan aturan objek | `models.py` |
| **T**emplate | tampilan | `templates/` |
| **V**iew | menerima request, mengolah data, memilih template | `views.py`, `views_admin.py` |

Pemisahan tanggung jawab: aturan streak ada di model, bukan di view atau template, sehingga `update_streak()` bisa dites tanpa membuka halaman.

### 8.2 Alur meminjam dan mengembalikan

1. `pinjam_buku` (POST, `buku/<olid>/pinjam/`): validasi id dengan `ITEM_ID_RE`; tolak jika sudah dipinjam atau sudah mencapai `user.max_loans` (default 5); `_pastikan_buku`; `Peminjaman.objects.create(...)` berstatus `Dibaca`; notifikasi; redirect ke `/peminjaman/`.
2. `perpanjang_buku` (POST): tolak jika `perpanjangan >= MAKS_PERPANJANG` (1); jika belum, tambah `MASA_PERPANJANG_HARI` (7 hari).
3. `kembalikan_buku`: status `Selesai`, `progress_baca = 100`, `tanggal_kembali` diisi, notifikasi; pencapaian 5/10/25/50 buku selesai memicu notifikasi khusus.
4. Catatan: `baca_buku` juga membuat `Peminjaman` berstatus `Dibaca` (lewat `get_or_create`) bila belum ada, sehingga membuka halaman baca sama dengan memulai pinjaman.

### 8.3 Alur membaca, EXP, dan streak

1. `baca/<olid>/` merender `baca.html`; `buku.js` memuat PDF dengan PDF.js dari `data-link`; `baca-ring.js` mengirim detik baca berkala ke `api/waktu-baca/`.
2. `waktu_baca` membatasi klaim detik dengan selisih waktu nyata (`session["ring_terakhir"]`), menambah ke `Peminjaman.waktu_baca_detik`, mengelola "kolam" detik (setiap 120 detik beri 10 EXP), menambah detik harian (target 1800 detik), lalu memanggil `reset_streak_if_expired()` dan `update_streak()`.
3. Level = `exp // 100 + 1`. Notifikasi dikirim saat streak 3/7/14/30, EXP bertambah, level naik, atau peringkat naik.

### 8.4 Pencarian katalog

`katalog_view` membaca `kategori` dan `q`. Filter kategori memakai `Q(...) | Q(...)` dengan `icontains`, sehingga nilai baru (`computer_science`) dan label lama (`Computer Science`) sama-sama cocok. Pencarian `q` mencari di judul, penulis, kategori, dan deskripsi.

---

## 9. Tanya Jawab

### A. Konsep OOP

**1. Apa itu OOP?**
Paradigma pemrograman yang memodelkan program sebagai kumpulan objek berisi data (atribut) dan perilaku (method). Empat pilarnya: encapsulation, abstraction, inheritance, polymorphism.

**2. Apa beda class dan object?**
Class adalah cetak biru, object adalah wujud nyatanya. `BukuDigital` adalah class; satu buku tertentu di database adalah object.

**3. Jelaskan encapsulation dan contohnya di proyekmu.**
Membungkus data dan method pengolahnya dalam satu class. `CustomUser` menyimpan `exp` dan `streak_days` bersama `update_streak()` dan `reset_streak_if_expired()`. View `waktu_baca` (U12) cukup memanggil method itu tanpa menulis ulang aturannya.

**4. Jelaskan abstraction dan contohnya.**
Menyembunyikan detail dan menampilkan yang perlu. `ItemKatalog` (abstract base model berisi kerangka atribut umum) dan ORM yang menyembunyikan SQL (terlihat di Katalog, U4).

**5. Jelaskan inheritance dan contohnya.**
Subclass mewarisi atribut dan method induk. `BukuDigital` dan `Jurnal` mewarisi `ItemKatalog`; `CustomUser` mewarisi `AbstractUser`; `AchievementForm` mewarisi `ModelForm`.

**6. Jelaskan polymorphism dan contohnya.**
Nama method sama berperilaku berbeda tergantung objeknya. `__str__` ditimpa di banyak model: `str(buku)` menghasilkan `[Digital] ...`, `str(jurnal)` menghasilkan `[Jurnal] ...`.

**7. Mengapa `__str__` termasuk polimorfisme?**
Ada hierarki (semua mewarisi `models.Model`), nama method sama, dipanggil dengan cara sama (`str(obj)`), hasilnya berbeda per class. Ini method overriding.

**8. Ada polimorfisme yang terjadi saat program berjalan?**
Ya, `Peminjaman.__str__`: `item = self.buku_digital or self.jurnal` bisa berisi buku atau jurnal, dan `f"{item}"` memanggil `__str__` milik class yang sedang dipegang.

**9. Apa beda overriding dan overloading?**
Overriding: subclass menulis ulang method induk dengan nama dan parameter sama. Overloading: nama sama dengan parameter berbeda dalam satu class. Python tidak mendukung overloading secara langsung; proyek ini memakai overriding.

**10. Apa itu `super()`?**
Memanggil method class induk, misalnya `super().__init__(*args, **kwargs)` di `AchievementForm` atau `super().save(...)` di `Pengaturan`.

**11. Apa arti `abstract = True`?**
Class hanya cetak biru: tidak membuat tabel sendiri dan tidak bisa dibuat objeknya langsung. Field-nya disalin ke tabel subclass.

**12. Beda abstraction dan encapsulation?**
Abstraction: apa yang disembunyikan dari pemakai (`ItemKatalog`, ORM). Encapsulation: membungkus data dengan method yang mengaturnya (`CustomUser` dengan `update_streak()`).

**13. Apa fungsi `@property`?**
Membuat method dibaca seperti atribut tanpa kurung, misalnya `buku.kategori_label`, `user.nama_asli`, `achievement.ikon_tampil`. Nilainya dihitung, tidak disimpan di database.

**14. Apa fungsi `@classmethod` pada `Pengaturan.muat()`?**
Dipanggil dari class tanpa membuat object dulu, lalu mengambil atau membuat satu-satunya baris pengaturan. (Belum dipanggil di view mana pun; lihat bagian 10.)

**15. Apa itu `related_name`?**
Nama jalan balik dari induk ke objek terkait, misalnya `user.notifikasi.all()`.

### B. Per fitur

**16. OOP apa di halaman Login?**
Pewarisan. `authenticate()` mengembalikan object `CustomUser`; `is_active` dan `check_password()` diwarisi dari `AbstractUser`; `menunggu_persetujuan` dan field OTP ditambahkan `CustomUser`. `login_view` sendiri fungsi biasa.

**17. OOP apa di Katalog Buku?**
Tiap buku lokal adalah object `BukuDigital` yang mewarisi `ItemKatalog`. Label kategori lewat property `kategori_label`, status bookmark lewat asosiasi `Bookmark`, dan pencarian lewat ORM.

**18. OOP apa di Kelola Buku admin?**
Pembuatan object `BukuDigital` dengan atribut warisan dan khusus (`isbn`), `save()` dan `delete()` warisan dari `Model`, `kategori_label` untuk label, dan asosiasi saat membuat `Notifikasi` untuk tiap anggota.

**19. Fitur mana paling kaya OOP di sisi admin?**
Kelola Achievement: `AchievementForm` mewarisi `forms.ModelForm`, memakai class `Meta` bersarang, meng-override `__init__` dengan `super()`, dan menyimpan aturan validasi di dalam class.

**20. Apakah `clean_nama()` termasuk overriding?**
Tidak. `ModelForm` tidak punya method itu. Django memanggil method berawalan `clean_` berdasarkan nama field, jadi itu hook. Overriding yang sah adalah `__init__`.

**21. Di fitur mana enkapsulasi paling jelas?**
Waktu baca dan streak (U12), Dashboard (U3, `reset_streak_if_expired`), serta `nama_asli` (Dashboard, Top Reader, Profil, Admin) dan `kategori_label` (Katalog, Bookmark, Kelola Buku).

**22. Di fitur mana asosiasi paling jelas?**
Bookmarks (`mark.buku_digital.judul`), Peminjaman (`p.item.judul`), dan Notifikasi (satu object per `CustomUser`).

**23. Halaman mana OOP-nya paling sedikit?**
Laporan, Login, dan Register. Jawab jujur: halaman itu terutama memakai object model lewat ORM; OOP-nya ada di class yang dipakainya (`CustomUser`, `Peminjaman`), bukan di view-nya.

**24. Mengapa `Peminjaman` punya dua `ForeignKey` yang boleh kosong?**
Satu peminjaman berisi buku atau jurnal. Karena itu `__str__` memakai `self.buku_digital or self.jurnal`.

### C. Relasi antarobjek

**25. Apa itu asosiasi? Contohnya?**
Hubungan antarobjek lewat referensi tanpa menyalin data. `Peminjaman` dan `Bookmark` menghubungkan `CustomUser` dengan `BukuDigital` atau `Jurnal` lewat `ForeignKey`.

**26. Apa itu agregasi? Contohnya?**
Has-a lemah: bagian bisa hidup tanpa induk. `Peminjaman.buku_digital` memakai `SET_NULL`, jadi buku bisa dihapus tanpa menghapus riwayat.

**27. Apa itu komposisi? Contohnya?**
Has-a kuat: bagian ikut hilang bersama induk. `Notifikasi.user` memakai `CASCADE`, jadi user dihapus, notifikasinya ikut hilang.

**28. Beda agregasi dan komposisi?**
Tanyakan: "kalau induk dihapus, apakah bagian masih bermakna?" Masih → agregasi. Tidak → komposisi.

**29. Apakah Django menandai relasi itu secara eksplisit?**
Tidak. Django hanya punya `ForeignKey` dan `on_delete`. Agregasi dan komposisi adalah tafsiran dari perilaku `SET_NULL` dan `CASCADE`.

**30. Apa itu dependensi? Beda dengan asosiasi?**
Dependensi: satu bagian kode memakai object lain sementara (parameter atau variabel lokal), seperti `waktu_baca` memakai `user.update_streak()`. Asosiasi disimpan sebagai atribut (relasi tetap), seperti `Peminjaman.user`.

**31. Apa beda `CASCADE` dan `SET_NULL`?**
`CASCADE` menghapus data terkait saat induk dihapus. `SET_NULL` hanya mengosongkan kolom relasinya.

### D. Alur, keamanan, dan Django

**32. Jelaskan alur dari awal sampai halaman tampil.**
Browser meminta alamat → `config/urls.py` → `perpustakaan/urls.py` → fungsi di view → model mengambil data → `render()` ke template → HTML dikirim ke browser.

**33. Apa itu MTV?**
Model (data), Template (tampilan), View (logika). View Django ≈ Controller di MVC, Template Django ≈ View di MVC.

**34. Kenapa user tidak bisa mengembalikan atau memperpanjang pinjaman orang lain?**
`get_object_or_404(Peminjaman, id=..., user=request.user)` hanya menemukan pinjaman milik user yang login; selain itu 404.

**35. Apa fungsi `{% csrf_token %}`, `@login_required`, dan `@require_POST`?**
`csrf_token`: perlindungan CSRF di setiap form POST. `@login_required`: mengalihkan user yang belum login. `@require_POST`: menolak selain POST untuk aksi yang mengubah data.

**36. Kenapa pakai ORM, bukan SQL langsung?**
Lebih ringkas, aman dari SQL injection, dan database mudah diganti tanpa mengubah kode.

**37. Apa itu migrasi?**
Catatan perubahan struktur model ke database. `makemigrations` membuat berkasnya, `migrate` menerapkannya.

**38. Apa fungsi `format_html` di `admin.py`?**
Membuat HTML yang aman dengan meng-escape variabel, sehingga terhindar dari XSS.

**39. Bagaimana sistem mencegah waktu baca dipalsukan?**
`waktu_baca` membatasi detik yang diklaim browser sebesar selisih waktu nyata sejak permintaan sebelumnya (`session["ring_terakhir"]`).

**40. Bagaimana peringkat dan level dihitung?**
Level = `exp // 100 + 1`. Peringkat di `waktu_baca` = 1 + jumlah member non-admin dengan EXP lebih tinggi. (Dashboard memakai hitungan yang sedikit berbeda; lihat bagian 10.)

### E. Tes

**41. Apa fungsi `tests.py` dan bagaimana menjalankannya?**
Menguji otomatis bahwa fitur sesuai aturan. Dijalankan dengan `python manage.py test`.

**42. Apa fungsi `setUp()`, `refresh_from_db()`, dan `force_login`?**
`setUp()` menyiapkan data awal (ditimpa dari `TestCase`, jadi contoh overriding). `refresh_from_db()` memuat ulang object dari database karena view mengubah data di database sementara object di tes masih menyimpan nilai lama. `force_login` membuat user langsung login tanpa mengisi form.

**43. Apa hasil `python manage.py test` saat ini?**
13 tes dijalankan, 3 gagal (rincian di bagian 10). Jawab jujur dan tunjukkan tes yang gagal, bukan disembunyikan.

### F. Pertanyaan jebakan

**44. Apakah enkapsulasi di Python benar-benar menyembunyikan data?**
Tidak seketat Java. Python memakai konvensi (awalan `_`), `@property`, dan pengelompokan data dengan method dalam class.

**45. Apakah `add_exp()` dipakai?**
Tidak. Sudah dicari di seluruh file `.py`: tidak ada pemanggilnya. EXP ditambah langsung di `waktu_baca` (`user.exp += exp_gain`, `views.py:1299`). Jawab: method itu disiapkan sebagai antarmuka resmi, belum dipakai.

**46. Kenapa `kembalikan_buku` bisa lewat GET?**
Tombolnya berupa tautan dan view tanpa `@require_POST`. Praktik yang lebih baik adalah POST dengan CSRF; itu bahan perbaikan.

**47. Kenapa `tanggal_kembali` dipakai untuk jatuh tempo sekaligus tanggal pengembalian?**
Selama pinjaman aktif ia menyimpan jatuh tempo, dan saat dikembalikan diisi waktu pengembalian. Itu pilihan desain; desain yang lebih bersih memakai field `jatuh_tempo` terpisah.

**48. Apa yang kamu kembangkan jika ada waktu?**
Memindahkan `kembalikan_buku` ke POST, memisahkan field `jatuh_tempo`, membersihkan kode lama (`_ikon` di `admin.py`, view ganda di `views.py`), memperbaiki 3 tes yang gagal, dan menambah tes untuk pinjam ganda dan batas pinjam.

### G. Cara menjawab

**Rumus 3 langkah:** (1) pengertian singkat, (2) tunjuk contoh nyata dengan nama file dan class atau method, (3) alasan mengapa itu termasuk pilar tersebut.

**Contoh jawaban polimorfisme:** "Polimorfisme adalah method bernama sama yang berperilaku berbeda tergantung objeknya. Di `models.py`, `__str__` ditimpa di `BukuDigital`, `Jurnal`, dan model lain. `str(buku)` menghasilkan `[Digital] ...` dan `str(jurnal)` menghasilkan `[Jurnal] ...`. Nama method sama, cara memanggil sama, hasil berbeda per class."

**Tips menunjuk kode:** komentar di `models.py` memakai penanda yang bisa dicari dengan `Ctrl+F`: `[ABSTRAKSI]`, `[PEWARISAN]`, `[POLIMORFISME - overriding]`, dan `ASOSIASI ANTAROBJEK`.

---

## 10. Catatan Jujur (Cek Sebelum Presentasi)

**Hasil `python manage.py test` (dijalankan pada kode di ZIP, Django 6.1.1): 13 tes, 3 gagal.**

| Tes | Gejala | Penyebab yang terlihat |
|---|---|---|
| `LocalCatalogLoanFlowTests.test_saved_book_can_be_borrowed_and_returned_without_open_library` (`tests.py:43`) | `self.user.exp` bernilai `0`, tes mengharapkan `50` | `kembalikan_buku` tidak menambah EXP; EXP sekarang bertambah di `waktu_baca`. Tes belum disesuaikan |
| `CollectionPaginationTests.test_bookmarks_are_paginated_and_show_no_fake_statistics` (`tests.py:136`) | teks `Filter kategori buku` tidak ada di halaman bookmarks | tes mengharapkan elemen filter kategori yang tidak ada di `bookmarks.html` saat ini |
| `CollectionPaginationTests.test_history_is_paginated` (`tests.py:184`) | `KeyError: 'page_obj'` | `riwayat_view` tidak mengirim `page_obj` ke template (tidak ada paginasi), padahal tes mengharapkannya |

Perbaiki tes atau kodenya sebelum pengumpulan, atau siapkan penjelasan bahwa tes tertinggal dari perubahan fitur.

**Temuan lain di kode**

1. **`add_exp()` dan `Pengaturan.muat()` tidak dipanggil di mana pun** (sudah dicari di seluruh `.py` selain migrasi). Komentar di `views.py:919` menyebut "classmethod Pengaturan", padahal kodenya tidak memanggilnya.
2. **`CustomUser.save()` hanya memanggil `super().save()`.** Overriding secara teknis, tetapi bukan contoh polimorfisme yang kuat.
3. **`Achievement` dan `UserAchievement` hanya terdaftar di `admin.py`;** tidak dipakai view. Yang aktif di fitur achievement adalah `AchievementGambar` dan fungsi `_achievements` (`views.py:1075`).
4. **View ganda yang tidak dirutekan.** `dashboard_admin` dan `kelola_buku_admin` di `views.py:1364-1380` tidak dipakai karena `urls.py` mengarah ke `views_admin.py`. Aman dihapus sebelum pengumpulan.
5. **`kembalikan_buku` tanpa `@require_POST`** (`views.py:820`), berbeda dengan `perpanjang_buku`.
6. **Kode tidak terjangkau di `admin.py`.** Di `_ikon`, semua kode setelah `return` pertama tidak pernah dijalankan dan memakai `IKON_ACHIEVEMENT_BERSIH` yang tidak di-import di file itu. Logikanya sudah dipindah ke property `ikon_tampil`.
7. **Peringkat tidak seragam.** `dashboard` menghitung `User.objects.filter(exp__gt=user.exp).count() + 1` (semua user, `views.py:383`), sedangkan `waktu_baca` dan `leaderboard_view` hanya menghitung member non-admin.
8. **`baca_buku` membuat `Peminjaman` otomatis** lewat `get_or_create` (`views.py:602`), jadi membuka halaman baca sama dengan memulai pinjaman.
9. **Atribut sementara bukan bagian class.** `total_pembaca`, `jumlah_pinjaman`, `is_kembali`, `is_telat`, `p.item`, `p.pengguna`, dan `jatuh_tempo` ditempel ke object saat runtime. Jangan dihitung sebagai enkapsulasi.
10. **Atribut private hanya konvensi.** Tidak ada atribut `_x` atau `__x` di `models.py`. Method berawalan `_` yang benar-benar ada di class adalah `_cek_ukuran` (`AchievementForm`) serta `_preview` dan `_ikon` (`admin.py`).
11. **`clean_*` di `AchievementForm` bukan overriding.** Itu hook yang dipanggil Django berdasarkan nama. Overriding yang sah hanya `__init__`.
12. **`id_item` memakai `unique=True` dan `primary_key=True` sekaligus.** `unique` redundan, tidak merusak.
13. **`simpan_progress` didaftarkan di `config/urls.py`,** bukan di `perpustakaan/urls.py`. Berfungsi normal; bisa dipindahkan.
14. **Tipe `pk` anggota tidak seragam** di `urls.py`: `<str:pk>` untuk edit/hapus/aktifkan, `<int:pk>` untuk setujui/tolak.
15. **`baca.html` masih punya CSS `iframe`** dengan komentar Internet Archive, kemungkinan sisa versi lama karena buku sekarang dibaca lewat PDF.js.
16. **Agregasi dan komposisi adalah tafsiran** dari `on_delete` (`SET_NULL` dan `CASCADE`), bukan sesuatu yang tertulis eksplisit di kode.
17. **Belum dijelaskan baris per baris:** isi `katalog.html` dan `peminjaman.html`, serta `_pastikan_buku` dan helper lain di `views.py`. Alur utamanya sudah dicek, tetapi detail template tidak dibahas.
