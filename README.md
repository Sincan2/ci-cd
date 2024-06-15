Untuk membuat script otomatis CI/CD dari GitHub ke DigitalOcean, kita bisa menggunakan GitHub Actions bersama dengan DigitalOcean. Berikut adalah langkah-langkah yang perlu diikuti:

1. **Persiapkan Droplet di DigitalOcean**: Pastikan Anda memiliki droplet yang sudah disiapkan di DigitalOcean.

2. **Install Nginx dan Node.js di Droplet**: Jika Anda menggunakan aplikasi Node.js, install Nginx dan Node.js di droplet Anda. Gunakan perintah berikut:

   ```sh
   sudo apt update
   sudo apt install nginx
   sudo apt install nodejs
   sudo apt install npm
   ```

3. **Buat Kunci SSH**: Buat kunci SSH untuk akses dari GitHub ke droplet Anda.

   ```sh
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

4. **Tambahkan Kunci Publik ke DigitalOcean**: Salin isi file `~/.ssh/id_rsa.pub` dan tambahkan ke bagian "SSH Keys" di DigitalOcean.

5. **Tambahkan Kunci Privat ke GitHub**: Tambahkan isi file `~/.ssh/id_rsa` ke Secrets GitHub (Settings -> Secrets) dengan nama `DO_SSH_KEY`.

6. **Buat GitHub Action Workflow**: Buat file `deploy.yml` di folder `.github/workflows` di repository Anda.

Berikut contoh isi file `deploy.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Setup SSH
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.DO_SSH_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan -H 123.45.67.89 >> ~/.ssh/known_hosts

    - name: Copy files via SCP
      run: scp -r -o StrictHostKeyChecking=no ./* user@123.45.67.89:/path/to/your/project

    - name: SSH and deploy
      run: ssh user@123.45.67.89 << 'EOF'
        cd /path/to/your/project
        npm install
        pm2 restart all
        exit
        EOF
```

Gantilah `123.45.67.89` dengan alamat IP droplet Anda dan `/path/to/your/project` dengan path yang sesuai di droplet Anda.

7. **Instal dan Konfigurasi PM2 di Droplet**: Jika belum diinstal, instal PM2 untuk manajemen proses Node.js:

   ```sh
   sudo npm install -g pm2
   pm2 startup
   ```

   Kemudian, jalankan aplikasi Anda menggunakan PM2:

   ```sh
   pm2 start app.js --name "myapp"
   pm2 save
   ```

Dengan langkah-langkah ini, setiap kali Anda mendorong perubahan ke cabang `main` di GitHub, workflow GitHub Actions akan secara otomatis mengirimkan kode Anda ke droplet di DigitalOcean dan memulai ulang aplikasi Node.js menggunakan PM2.
