# Dokumentasi Pipeline DevSecOps – Vuln Bank

Hi, jadi saya mau jelasin soal pipeline yang saya bikin buat project DevSecOps ini.  
Jadi ceritanya si Vuln Bank ini kan punya aplikasi web yang masih banyak celah,  
nah tugasnya bikin pipeline otomatis di GitHub Actions yang bisa nge-scan keamanan tiap kali ada perubahan code.

---

## Alur Kerja Pipeline

- Pertama, pipeline ini langsung jalan otomatis kalau ada push atau pull request ke branch `main`.
- Begitu jalan, dia checkout repo, terus siapin environment Python, install tool-tool yang diperlukan buat scanning.

### Step-step scanning:

1. **Secret Scanning (Gitleaks)** → buat nyari kalau ada API key, password, atau kredensial lain yang nggak sengaja ke-push.  
2. **SCA (pip-audit)** → ngecek library atau dependency Python yang dipakai, apakah ada yang rentan atau udah ketahuan ada CVE.  
3. **SAST (Bandit)** → cek langsung di source code Python, misalnya ada penggunaan fungsi berbahaya atau insecure coding.  
4. **Misconfiguration Scan (Trivy config)** → ngecek file konfigurasi kayak Dockerfile atau manifest biar nggak ada setting berbahaya (contoh: port kebuka, jalanin pakai root, dsb).  
5. **Image Scan (Trivy image)** → setelah build Docker image, dicek juga kalau ada package di dalam image yang rentan.  
6. **DAST (ZAP/Nikto)** → aplikasi dijalankan pakai docker-compose, lalu dites kayak diserang dari luar, buat nyari celah kayak XSS, SQL injection, dll.

Semua hasil scan ini disimpan dalam bentuk file (`json/html/txt`) dan di-upload jadi artifact di GitHub Actions, jadi bisa di-download buat dicek lagi.

---

## Cara Ngasih Tahu Kalau Ada Critical

Pipeline ini nggak cuma nge-scan aja. Saya bikin step buat ngumpulin semua hasil scan terus dihitung ada nggak temuan dengan level **HIGH** atau **CRITICAL**.  

- Kalau ada → pipeline bakal kirim notifikasi ke **Discord** lewat webhook yang udah diset di GitHub Secrets.  
- Isi notifnya itu ringkasan hasil scan plus link ke hasil run di GitHub. Jadi gampang buat langsung ngecek.  
- Ada juga opsi supaya pipeline langsung **gagal otomatis** kalau ada temuan high/critical → tinggal set `FAIL_ON_HIGH=true`.

---

## Tools yang Dipakai

- **Gitleaks** → secret scanning  
- **pip-audit** → cek dependency Python  
- **Bandit** → SAST Python  
- **Trivy** → config scan & image scan  
- **OWASP ZAP / Nikto** → DAST  
- **jq** → parsing hasil JSON biar gampang buat ngitung critical/high  

---

## Penutup

Intinya, pipeline ini udah lengkap buat ngejaga aplikasi biar lebih aman.  
Jadi setiap ada kode baru ke branch utama, pipeline otomatis ngecek dari sisi kode, dependency, konfigurasi, sampai aplikasi jalan.  
Kalau ada masalah critical, langsung kasih tahu di Discord.  
