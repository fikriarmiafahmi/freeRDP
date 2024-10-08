# Tutorial Membuat RDP Gratis Menggunakan GitHub Actions

## Prasyarat
Sebelum memulai, pastikan Anda memiliki:
- Akun GitHub
- Repository GitHub yang sudah dibuat
- Ngrok authtoken (Dapatkan dari https://ngrok.com/)

## Langkah-langkah

### 1. Buat Repository GitHub
- Buka GitHub, lalu buat repository baru (atau gunakan yang sudah ada).
- Tambahkan file `.yml` untuk konfigurasi GitHub Actions.

### 2. Menyiapkan Secret Ngrok
- Masuk ke repository GitHub Anda.
- Klik **Settings** > **Secrets and variables** > **Actions**.
- Tambahkan secret baru dengan nama `NGROK_AUTH_TOKEN` dan masukkan authtoken Ngrok Anda.

### 3. Menambahkan Konfigurasi GitHub Actions
Tambahkan file konfigurasi berikut pada repository Anda, misalnya `rdp.yml`, dalam direktori `.github/workflows/`:

```yml
name: CI

on: [push, workflow_dispatch]

jobs:
  build:
    runs-on: windows-latest

    steps:
    - name: Download Ngrok
      run: Invoke-WebRequest https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-windows-amd64.zip -OutFile ngrok.zip
    - name: Extract Ngrok
      run: Expand-Archive ngrok.zip
    - name: Authenticate Ngrok
      run: .\ngrok\ngrok.exe authtoken $Env:NGROK_AUTH_TOKEN
      env:
        NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}
    - name: Enable Remote Desktop
      run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
    - name: Enable Firewall for RDP
      run: Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
    - name: Set RDP Authentication
      run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
    - name: Set Password for Admin
      run: Set-LocalUser -Name "runneradmin" -Password (ConvertTo-SecureString -AsPlainText "P@ssw0rd!" -Force)
    - name: Start Ngrok Tunnel
      run: .\ngrok\ngrok.exe tcp 3389
```
## 4. Menjalankan Workflow
- Setelah konfigurasi di atas selesai, commit dan push perubahan ke repository Anda.
- Buka tab Actions pada repository Anda, lalu jalankan workflow dengan memilih Run workflow (jika menggunakan workflow_dispatch).
## 5. Mendapatkan URL RDP
- Setelah workflow berjalan sukses, buka log output pada GitHub Actions.
- Cari URL dari Ngrok (misalnya: tcp://0.tcp.ngrok.io:xxxxx).
- Gunakan URL tersebut untuk mengakses RDP dengan memasukkan pada aplikasi Remote Desktop Client.
## 6. Login ke RDP
- Gunakan username runneradmin.
- Password default adalah P@ssw0rd!. Pastikan untuk mengganti password ini setelah berhasil login.

# Selesai!
## Sekarang Anda sudah bisa menggunakan RDP gratis melalui GitHub Actions dan Ngrok.
