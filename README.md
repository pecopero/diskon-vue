# Manajemen Diskon

Aplikasi CRUD untuk manajemen diskon, dibangun dengan Vue 3 + Vite + Material Design 3.

## 🔗 API

| Keterangan | URL |
|---|---|
| Base API | `https://crudcrud.com/api/ecda943a26c54cfea19dec66658b4b16` |
| Endpoint Diskon | `https://crudcrud.com/api/ecda943a26c54cfea19dec66658b4b16/DISKON` |

> ⚠️ API key crudcrud.com bersifat sementara (~24 jam). Jika expired, buat key baru di [crudcrud.com](https://crudcrud.com) dan update melalui tombol outlet di aplikasi.

## 🛠️ Teknologi

- [Vue 3](https://vuejs.org/) (Composition API)
- [Vite](https://vitejs.dev/)
- [Material Design 3](https://m3.material.io/)
- [crudcrud.com](https://crudcrud.com/) — REST API backend

## ✨ Fitur

- **Tambah diskon** — nama diskon + nilai (% atau Rp)
- **Ubah diskon** — edit data yang sudah ada
- **Hapus diskon** — single maupun bulk (multi-select)
- **Cari diskon** — filter real-time berdasarkan nama
- **Sortir** — berdasarkan nama atau nilai diskon
- **Badge "baru"** — muncul 10 detik setelah diskon ditambahkan
- **Validasi form** — nama tidak boleh kosong & tidak boleh duplikat
- **Toast notifikasi** — feedback setelah setiap aksi
- **Konfigurasi API URL** — bisa diganti langsung dari UI

## 🚀 Cara Menjalankan

### Development

```bash
# Install dependencies
npm install

# Jalankan dev server
npm run dev
```

Buka [http://localhost:5173](http://localhost:5173)

### Production Build

```bash
npm run build
```

Hasil build ada di folder `dist/`.

## 📁 Struktur Project

```
src/
├── App.vue                  # Komponen utama & logika CRUD
├── main.js                  # Entry point
├── assets/
│   └── main.css             # Global styles (Material Design 3)
└── components/
    ├── DiskonForm.vue        # Modal tambah / ubah diskon
    └── DeleteConfirm.vue     # Modal konfirmasi hapus
```

## 🌐 Cara Penggunaan

1. Buka aplikasi di URL : https://diskon-vue.vercel.app/
2. Klik chip **outlet** di toolbar
3. Masukkan API URL: `https://crudcrud.com/api/ecda943a26c54cfea19dec66658b4b16/DISKON`
4. Klik **Terapkan**
5. Mulai kelola diskon

---