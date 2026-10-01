
# API Development Guide

Dokumen ini menjelaskan cara membangun, menguji, dan menjalankan NestJS API di dalam pnpm workspace.

## Posisi API dalam project

```text
event-ticketing/
├── apps/
│   └── api/        # NestJS REST API
├── packages/
├── docs/
├── package.json
└── pnpm-workspace.yaml
```

`apps/api` adalah package tersendiri dengan `package.json`, source code, dependency, dan script-nya sendiri. Root workspace dapat menjalankan script package tersebut melalui pnpm.

## Cara membaca script

Script API berada di `apps/api/package.json`:

```json
{
  "scripts": {
    "build": "nest build",
    "start:dev": "nest start --watch",
    "test": "vitest run"
  }
}
```

Perintah ini:

```powershell
pnpm --dir apps/api run test
```

berarti:

1. `pnpm` menjalankan package manager.
2. `--dir apps/api` membuat pnpm bekerja dengan konteks folder API.
3. `run test` menjalankan script bernama `test` dari `apps/api/package.json`.

Alternatif dari root workspace:

```powershell
pnpm --filter api test
```

`--filter api` memilih package yang memiliki nama `api`.

## `test`: memeriksa perilaku kode

```powershell
pnpm --dir apps/api run test
```

### Tujuan

Test memeriksa apakah kode berperilaku sesuai harapan. Test tidak menjalankan HTTP server untuk dipakai terus-menerus dan tidak membuat build produksi.

Contoh konsep test:

```typescript
it('mengembalikan Hello World', () => {
  expect(appService.getHello()).toBe('Hello World!');
});
```

Test tersebut mempunyai tiga bagian:

1. Menjalankan `appService.getHello()`.
2. Mengambil hasilnya.
3. Membandingkannya dengan hasil yang diharapkan.

Jika sesuai, test lulus. Jika berbeda, test gagal dan process mengembalikan exit code non-zero.

### Mengapa dijalankan?

* Menemukan kesalahan secara otomatis.
* Mencegah perubahan baru merusak perilaku lama.
* Memberikan bukti bahwa suatu aturan bisnis bekerja.
* Nantinya digunakan oleh CI di GitHub.

### Kapan dijalankan?

* Setelah mengubah business logic.
* Sebelum membuat commit.
* Sebelum membuka pull request.
* Secara otomatis di CI.

### Hasil yang diharapkan

```text
Test Files  1 passed
Tests       1 passed
```

Lulus berarti test yang tersedia berhasil. Itu belum membuktikan bahwa seluruh aplikasi bebas bug; kualitas pemeriksaan bergantung pada cakupan dan kualitas test.

## `build`: mengompilasi aplikasi

```powershell
pnpm --dir apps/api run build
```

### Tujuan

Build mengubah source code TypeScript menjadi JavaScript yang dapat dijalankan Node.js.

Secara konsep:

```text
apps/api/src/main.ts
        ↓ TypeScript compiler
apps/api/dist/main.js
```

### Mengapa TypeScript perlu dibangun?

Node.js menjalankan JavaScript. TypeScript menambahkan type annotation dan pemeriksaan compile-time yang tidak digunakan langsung sebagai format deployment utama project ini.

Contoh TypeScript:

```typescript
function add(a: number, b: number): number {
  return a + b;
}
```

Hasil JavaScript secara konsep:

```javascript
function add(a, b) {
  return a + b;
}
```

Build juga memeriksa banyak kesalahan TypeScript. Contoh:

```typescript
const quantity: number = 'sepuluh';
```

Build akan gagal karena string tidak dapat diberikan kepada variable bertipe number.

### Kapan dijalankan?

* Sebelum commit penting.
* Sebelum deployment.
* Di CI.
* Setelah mengubah konfigurasi TypeScript atau module system.

### Hasil yang diharapkan

Perintah selesai dengan exit code `0` dan folder `apps/api/dist` dibuat atau diperbarui.

`dist/` adalah hasil generate dan tidak perlu dimasukkan ke Git karena dapat dibuat ulang dari source code.

## `start:dev`: menjalankan development server

```powershell
pnpm --filter api start:dev
```

### Tujuan

Menjalankan HTTP server dalam development mode agar endpoint dapat diakses selama kita mengembangkan aplikasi.

Bagian `--watch` pada script membuat NestJS memantau perubahan file. Ketika file disimpan, aplikasi dikompilasi dan dimulai ulang secara otomatis.

### Cara kerja sederhana

```text
Source code berubah
        ↓
NestJS mendeteksi perubahan
        ↓
TypeScript dikompilasi
        ↓
Server dimulai ulang
        ↓
Endpoint menggunakan kode terbaru
```

### Mengapa terminal tetap aktif?

HTTP server adalah process yang terus menunggu request. Selama server berjalan, terminal digunakan untuk menampilkan log. Tekan `Ctrl+C` untuk menghentikannya.

### Kapan digunakan?

Gunakan selama coding dan pengujian manual lokal. Jangan gunakan development mode sebagai process produksi.

## `start:prod`: menjalankan hasil build

Setelah build:

```powershell
pnpm --dir apps/api run start:prod
```

Development mode menjalankan dan memantau source code. Production mode menjalankan JavaScript hasil build dari `dist/` tanpa file watcher.

```text
start:dev  → nyaman untuk coding, watch aktif
start:prod → menjalankan hasil build, untuk environment produksi
```

## Arti `Hello World!`

Ketika menjalankan:

```powershell
Invoke-RestMethod http://localhost:3000
```

PowerShell mengirim HTTP request:

```http
GET / HTTP/1.1
Host: localhost:3000
```

Alurnya:

```text
PowerShell/client
    ↓ GET /
NestJS HTTP server
    ↓
AppController
    ↓
AppService.getHello()
    ↓
"Hello World!"
    ↓ HTTP response
PowerShell/client
```

`Hello World!` bukan fitur produk. Itu adalah smoke test awal yang membuktikan bahwa:

* Process API dapat dimulai.
* Port 3000 dapat menerima koneksi.
* Routing NestJS bekerja.
* Controller dan service dapat dibuat oleh dependency injection container.
* Response dapat dikirim kembali ke client.

## Perbedaan test, build, dan start

| Perintah       | Pertanyaan yang dijawab                                            | Menjalankan server terus-menerus? | Membuat`dist/`?                          |
| -------------- | ------------------------------------------------------------------ | --------------------------------- | ------------------------------------------ |
| `test`       | Apakah perilaku kode sesuai dengan yang diharapkan?                | Tidak                             | Tidak                                      |
| `build`      | Apakah source code dapat dikompilasi menjadi JavaScript?           | Tidak                             | Ya                                         |
| `start:dev`  | Apakah API bisa dijalankan untuk development dan menerima request? | Ya                                | Dapat memakai proses kompilasi development |
| `start:prod` | Apakah hasil build dapat dijalankan seperti di produksi?           | Ya                                | Tidak; memakai`dist/` yang sudah ada     |

Satu perintah tidak menggantikan yang lain. Test dapat lulus tetapi build gagal karena konfigurasi. Build dapat berhasil tetapi business logic salah. Server dapat hidup tetapi endpoint tertentu masih bermasalah.

## Urutan verifikasi sebelum commit

```powershell
pnpm --dir apps/api run test
pnpm --dir apps/api run build
pnpm --filter api start:dev
```

Pada terminal kedua:

```powershell
Invoke-RestMethod http://localhost:3000
```

Kemudian hentikan server dengan `Ctrl+C`.

Urutan tersebut memeriksa:

1. Perilaku yang diuji masih benar.
2. TypeScript dapat dibangun.
3. Aplikasi dapat hidup dan menerima request nyata.

## Exit code

Setiap command-line program mengembalikan exit code:

* `0`: berhasil.
* Selain `0`: gagal.

pnpm menggunakan exit code untuk mengetahui apakah script berhasil. GitHub Actions nantinya juga menggunakan nilai ini untuk menentukan apakah CI lulus.

Pesan seperti `ELIFECYCLE` atau `RECURSIVE_RUN_FIRST_FAIL` biasanya merupakan laporan bahwa sebuah script mengembalikan exit code gagal. Penyebab utama biasanya berada beberapa baris sebelumnya.

## Checklist API awal

* [ ] `pnpm --dir apps/api run test` berhasil.
* [ ] `pnpm --dir apps/api run build` berhasil.
* [ ] `pnpm --filter api start:dev` berhasil.
* [ ] `GET http://localhost:3000` mengembalikan `Hello World!`.
* [ ] Perubahan diperiksa dengan `git status`.
* [ ] Tidak ada `.env`, `node_modules`, atau `dist` yang masuk staging.
