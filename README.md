# Don_Virtus_NVR

AINVR
AINVR adalah sistem Network Video Recorder (NVR) yang sederhana namun powerful, dirancang untuk mengelola stream video dari kamera IP (RTSP), melakukan object detection secara real-time dan offline menggunakan AI (YOLOv8), serta menyediakan dashboard yang user-friendly untuk monitoring dan analisis.
Tujuan Proyek

Menyediakan solusi NVR yang efisien untuk kamera IP dengan fitur AI object detection.
Mendukung monitoring real-time dan analisis video tersimpan berdasarkan rentang waktu.
Memberikan antarmuka dashboard yang mudah digunakan oleh berbagai jenis pengguna (administrator dan operator).
Memungkinkan pengaturan fleksibel untuk kamera, AI, rekaman, dan notifikasi.

Struktur Dashboard
Dashboard AINVR dirancang dengan antarmuka yang intuitif dan terdiri dari menu berikut:

Home:

Menampilkan live view hingga 8 stream video dari kamera (dengan duplikasi stream untuk 2 kamera).
Overlay hasil AI (bounding box, label) untuk object detection real-time.
Contoh penggunaan: User melihat stream langsung dengan deteksi orang atau kendaraan.


Playback:

Memungkinkan user memilih video tersimpan dari folder recordings untuk ditonton.
Menampilkan metadata deteksi AI (misal: waktu dan jenis objek).
Contoh penggunaan: User memutar rekaman dari Cam1 pada 2025-05-12 10:00:00.


AI Analysis:

Form untuk memilih rentang waktu (start_time, end_time) dan kamera untuk analisis AI offline.
Hasil analisis ditampilkan sebagai laporan deteksi dan video dengan overlay.
Contoh penggunaan: User menganalisis video Cam1 dari 2025-05-12 10:00:00 sampai 11:00:00 untuk mendeteksi "person".


Settings (hanya admin untuk pengaturan sistem, operator bisa lihat status):

Camera:
Tambah/edit/hapus kamera (nama, RTSP URL, kredensial).
Aktifkan/nonaktifkan stream.
Contoh: Tambah Cam1 dengan URL rtsp://admin:password@camera1_ip:554/stream.


Dashboard:
Atur layout grid (4x2, 2x4, dll.) dan assign kamera ke slot.
Aktifkan/nonaktifkan overlay AI.
Contoh: Set layout 4x2, slot 1-4 untuk Cam1, slot 5-8 untuk Cam2.


AI:
Aktifkan/nonaktifkan AI per kamera.
Pilih kelas objek (e.g., person, car) dan confidence threshold.
Contoh: Aktifkan AI untuk Cam1, deteksi hanya "person" dengan confidence 0.6.


Recording:
Aktifkan/nonaktifkan rekaman per kamera.
Atur durasi file rekaman dan masa retensi.
Contoh: Rekam Cam1 dengan file 1 jam, hapus otomatis setelah 30 hari.


Notifications:
Aktifkan notifikasi untuk deteksi tertentu (e.g., "person").
Pilih metode (email, push notification).
Contoh: Kirim email jika "person" terdeteksi di Cam1.





Prerequisites
Sebelum memulai, pastikan Anda memiliki:

Hardware:
Server dengan CPU minimal 4 core, RAM 8GB (disarankan GPU untuk performa AI optimal).
Kamera IP yang mendukung RTSP (minimal 2 kamera).
Ruang penyimpanan cukup untuk rekaman (SSD direkomendasikan).


Software:
OS: Ubuntu 20.04+ atau distribusi Linux berbasis Debian.
Python 3.8+.
Docker (opsional, untuk deploy MediaMTX dan komponen lain).
Browser modern (Chrome, Firefox, dll.).


Dependencies:
MediaMTX (streaming server).
FFmpeg (pemrosesan video).
Python packages: opencv-python, ultralytics, flask, flask-login, celery, redis, psycopg2 (untuk PostgreSQL), pyyaml.
Database: SQLite (default) atau PostgreSQL (disarankan untuk multi-user).


Rekomendasi:
Gunakan virtual environment (venv) untuk Python dependencies.
Siapkan SMTP server (e.g., Gmail) untuk notifikasi email.



Instalasi
Ikuti langkah-langkah berikut untuk mengatur AINVR:

Clone Repository:
git clone https://github.com/username/AINVR.git
cd AINVR


Setup Virtual Environment:
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt


Install MediaMTX:

Manual:wget https://github.com/bluenviron/mediamtx/releases/download/v1.12.2/mediamtx_v1.12.2_linux_amd64.tar.gz
tar -xvzf mediamtx_v1.12.2_linux_amd64.tar.gz
sudo mv mediamtx /usr/local/bin/
sudo mv mediamtx.yml /usr/local/etc/


Docker (opsional):docker run --rm -it -e MTX_PROTOCOLS=tcp -p 8554:8554 -p 8889:8889 -v $(pwd)/mediamtx.yml:/mediamtx.yml bluenviron/mediamtx




Setup Database:

SQLite (default, cocok untuk setup sederhana):
Tidak perlu konfigurasi tambahan.
Database otomatis dibuat di detections.db saat aplikasi dijalankan.


PostgreSQL (direkomendasikan untuk multi-user dan skalabilitas):sudo apt install postgresql
sudo -u postgres psql
CREATE DATABASE ainvr;
CREATE USER ainvr_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE ainvr TO ainvr_user;
\q


Update config.py:DATABASE_URL = "postgresql://ainvr_user:your_password@localhost/ainvr"






Konfigurasi Awal:

Edit config.py:SECRET_KEY = "your_secret_key"  # Ganti dengan random string
DATABASE_URL = "sqlite:///detections.db"  # Atau PostgreSQL URL
MEDIAMTX_API = "http://localhost:9997/v3/"  # API MediaMTX
SMTP_SERVER = "smtp.gmail.com"
SMTP_PORT = 587
SMTP_USER = "your_email@gmail.com"
SMTP_PASSWORD = "your_app_password"


Edit /usr/local/etc/mediamtx.yml untuk kamera:paths:
  cam1:
    source: rtsp://admin:password@camera1_ip:554/stream
    record: yes
    recordPath: ./recordings/cam1/%Y-%m-%d_%H-%M-%S.mp4
  cam2:
    source: rtsp://admin:password@camera2_ip:554/stream
    record: yes
    recordPath: ./recordings/cam2/%Y-%m-%d_%H-%M-%S.mp4




Inisialisasi Database:
python init_db.py


Script ini membuat tabel detections dan users (untuk multi-user).



Penggunaan
Setelah instalasi selesai, jalankan sistem dengan langkah berikut:

Jalankan MediaMTX:
mediamtx /usr/local/etc/mediamtx.yml


Jalankan Redis (untuk Celery):
docker run -d -p 6379:6379 redis


Jalankan Flask App:
python app.py


Jalankan Worker AI (Celery):
celery -A tasks worker --loglevel=info


Akses Dashboard:

Buka http://server_ip:5000 di browser.
Login dengan kredensial default:
Admin: admin / admin
Operator: operator / operator


Catatan: Ubah password default di menu Settings > User Management setelah login pertama.



Cara Setting di Dashboard
Berikut panduan langkah demi langkah untuk pengaturan di dashboard:
1. Settings > Camera (Admin Only)

Akses: Menu Settings > Camera.
Fitur:
Tambah kamera baru: Input nama, RTSP URL, username, password.
Edit/hapus kamera existing.
Aktifkan/nonaktifkan stream.


Contoh:
Klik "Add Camera".
Input: Name=Cam1, RTSP URL=rtsp://admin:password@camera1_ip:554/stream, Enabled=Yes.
Klik "Save" untuk update mediamtx.yml.


Catatan: Restart MediaMTX mungkin diperlukan setelah perubahan (sudo systemctl restart mediamtx).

2. Settings > Dashboard (Admin Only)

Akses: Menu Settings > Dashboard.
Fitur:
Pilih layout grid (4x2, 2x4, dll.).
Assign kamera ke slot (e.g., slot 1-4 untuk Cam1).
Aktifkan/nonaktifkan overlay AI.


Contoh:
Pilih layout 4x2.
Set slot 1-4 ke Cam1, slot 5-8 ke Cam2.
Centang "Show AI Overlay".
Klik "Save" untuk update tampilan live view.



3. Settings > AI (Admin Only)

Akses: Menu Settings > AI.
Fitur:
Aktifkan/nonaktifkan AI per kamera.
Pilih kelas objek (e.g., person, car).
Atur confidence threshold (0.0-1.0).


Contoh:
Centang "Enable AI for Cam1".
Pilih kelas person dan car.
Set confidence ke 0.6.
Klik "Save" untuk update script AI.



4. Settings > Recording (Admin Only)

Akses: Menu Settings > Recording.
Fitur:
Aktifkan/nonaktifkan rekaman per kamera.
Atur durasi file (e.g., 3600 detik = 1 jam).
Atur masa retensi (e.g., 30 hari).


Contoh:
Centang "Record Cam1".
Set durasi 3600 detik.
Set retensi 30 hari.
Klik "Save" untuk update mediamtx.yml dan jadwalkan cleanup.



5. Settings > Notifications (Admin Only)

Akses: Menu Settings > Notifications.
Fitur:
Aktifkan notifikasi untuk kelas objek tertentu.
Pilih metode (email, push notification).


Contoh:
Centang "Notify for person".
Input email user@example.com.
Klik "Save" untuk update pengaturan notifikasi.



6. Settings > User Management (Admin Only)

Akses: Menu Settings > User Management.
Fitur:
Tambah/edit/hapus user (admin atau operator).
Ubah password.


Contoh:
Klik "Add User".
Input: Username=new_operator, Password=secure123, Role=Operator.
Klik "Save" untuk update database users.



7. AI Analysis (Admin dan Operator)

Akses: Menu AI Analysis.
Fitur:
Pilih rentang waktu dan kamera untuk analisis offline.
Lihat hasil sebagai laporan dan video dengan overlay.


Contoh:
Input: Start Time=2025-05-12 10:00:00, End Time=2025-05-12 11:00:00, Camera=Cam1.
Klik "Process with AI".
Lihat laporan deteksi di halaman hasil.



Pilihan Database
AINVR mendukung dua opsi database:

SQLite:
Kelebihan: Default, tidak perlu setup tambahan, cocok untuk sistem kecil atau pengembangan.
Kekurangan: Kurang optimal untuk multi-user atau data besar karena keterbatasan konkurensi.
Penggunaan: Otomatis digunakan jika DATABASE_URL di config.py diset ke sqlite:///detections.db.


PostgreSQL:
Kelebihan: Direkomendasikan untuk multi-user, mendukung konkurensi tinggi, skalabilitas baik.
Kekurangan: Perlu setup server PostgreSQL.
Penggunaan: Set DATABASE_URL ke postgresql://user:password@localhost/ainvr di config.py.


Fleksibilitas: Sistem dirancang untuk mendukung keduanya tanpa perubahan kode besar. Gunakan SQLite untuk prototype atau sistem kecil, dan PostgreSQL untuk produksi atau multi-user.

Rekomendasi: Gunakan PostgreSQL jika ada banyak user atau data rekaman besar. Untuk setup awal atau testing, SQLite sudah cukup.
Jenis User
AINVR mendukung multi-user dengan dua peran:

Administrator:
Akses penuh ke semua menu dan pengaturan (Camera, Dashboard, AI, Recording, Notifications, User Management).
Dapat menambah/mengedit operator.
Contoh: Mengatur kamera baru, mengubah layout dashboard, atau menambah user.


Operator:
Akses terbatas: Hanya ke Home (live view), Playback, dan AI Analysis.
Tidak dapat mengubah pengaturan sistem atau mengelola user.
Contoh: Melihat stream langsung, memutar rekaman, atau menjalankan analisis AI offline.



Implementasi: Menggunakan Flask-Login dengan role-based access control (RBAC). Tabel users menyimpan username, password (hashed), dan role (admin atau operator).
Kemudahan Pengembangan ke Depan
Proyek ini dirancang agar mudah dikembangkan:

Modular Architecture: Komponen terpisah (MediaMTX untuk streaming, Flask untuk dashboard, Celery untuk AI) memudahkan upgrade.
Containerization: Docker Compose untuk deploy cepat dan skalabilitas.
API-First Design: Endpoint API (e.g., /api/detections, /api/cameras) untuk integrasi dengan frontend lain (misal: mobile app).
Extensible AI: Ganti model AI (e.g., dari YOLOv8 ke YOLOv9) atau tambah fitur seperti face recognition dengan library seperti dlib.
Database Flexibility: Mendukung SQLite dan PostgreSQL, dengan opsi migrasi ke database lain (e.g., MySQL) via SQLAlchemy.
Cloud-Ready: Siap deploy di cloud dengan Kubernetes atau platform seperti AWS/GCP.

Fitur Tambahan

Custom Notifications: Admin dapat mengatur notifikasi berbasis deteksi (e.g., email saat "person" terdeteksi).
Storage Management: Batas penyimpanan rekaman dan cleanup otomatis.
Backup: Script untuk backup database dan rekaman:python backup.py --database detections.db --recordings ./recordings --output backup_$(date +%F).tar.gz


Security: HTTPS via Nginx reverse proxy, autentikasi kuat, dan enkripsi password dengan bcrypt.
Monitoring: Logging dengan logging Python, opsional ELK stack atau Prometheus + Grafana.
Testing: Unit test untuk Flask dan AI script menggunakan pytest:pytest tests/



Rekomendasi Terbaik

Docker Compose: Untuk deploy semua komponen dengan satu perintah:version: '3'
services:
  mediamtx:
    image: bluenviron/mediamtx
    ports:
      - "8554:8554"
      - "8889:8889"
    volumes:
      - ./mediamtx.yml:/mediamtx.yml
      - ./recordings:/recordings
  flask:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - postgres
      - redis
    volumes:
      - ./recordings:/app/recordings
  celery:
    build: .
    command: celery -A tasks worker --loglevel=info
    depends_on:
      - redis
    volumes:
      - ./recordings:/app/recordings
  redis:
    image: redis
    ports:
      - "6379:6379"
  postgres:
    image: postgres
    environment:
      POSTGRES_DB: ainvr
      POSTGRES_USER: ainvr_user
      POSTGRES_PASSWORD: your_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
volumes:
  postgres_data:

Jalankan:docker-compose up -d


Database: Gunakan PostgreSQL untuk produksi. SQLite cukup untuk pengembangan atau sistem kecil.
UI/UX: Gunakan Bootstrap atau Tailwind CSS untuk tampilan responsif. Tambahkan kalender (flatpickr) untuk pilih waktu.
Security: Aktifkan HTTPS dan batasi akses API dengan token JWT.
Testing: Tambahkan coverage report:pytest --cov=app tests/



Kontribusi
Silakan buka issue atau pull request di GitHub repository untuk kontribusi atau laporan bug.
Lisensi
Dilisensikan di bawah MIT License.
Troubleshooting

MediaMTX gagal konek ke kamera: Periksa RTSP URL dan kredensial di mediamtx.yml.
Dashboard lambat: Pastikan Celery worker berjalan dan gunakan GPU untuk AI.
Database error: Verifikasi DATABASE_URL di config.py dan koneksi PostgreSQL.
Lihat Troubleshooting Guide untuk detail.

