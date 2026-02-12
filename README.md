# go-ubot (Docker Builder)

Repository ini hanya untuk **membuild Docker image** dari repo sumber dan **push ke GHCR** secara manual.

## Apa yang dilakukan
- Checkout repo sumber `lubluniky/ubot`
- Build Docker image dari `Dockerfile` di root repo sumber
- Push image ke `ghcr.io/banghasan/go-ubot`

## Cara pakai (manual)
1. Buka tab **Actions** di GitHub repo ini.
2. Jalankan workflow **Build & Push GHCR (manual)**.
3. Isi input:
   - `ref`: branch/tag/commit dari repo sumber (default `main`)
   - `tag`: tag image (default `latest`)

Contoh:
- `ref = main`
- `tag = latest`

## Output
Image akan tersedia di GHCR:
- `ghcr.io/banghasan/go-ubot:<tag>`

## Catatan
- Repo sumber publik, jadi tidak butuh token khusus untuk checkout.
- Jika nanti repo sumber jadi private, tambahkan `SOURCE_REPO_PAT` pada Secrets dan gunakan di langkah checkout.
