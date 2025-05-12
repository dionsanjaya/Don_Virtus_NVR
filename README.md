# AINVR

> AINVR adalah sistem Network Video Recorder (NVR) sederhana namun powerful yang mengelola stream video dari kamera IP (RTSP), melakukan object detection real-time dan offline menggunakan AI (YOLOv8), serta menyediakan dashboard user-friendly untuk monitoring dan analisis.

## Overview

- **Purpose**:
  - Menyediakan solusi NVR efisien dengan AI object detection.
  - Mendukung monitoring real-time dan analisis video berdasarkan rentang waktu.
  - Menawarkan dashboard untuk administrator dan operator dengan pengaturan fleksibel.

- **Features**:
  + Data streaming dari kamera IP via MediaMTX.
  + Object detection real-time dan offline menggunakan YOLOv8.
    - Sub-pilihan: Car detection, Motorcycle detection (dapat ditambah: truck, bicycle, dll.).
  + Face recognition dan face search menggunakan `dlib`.
  + Object classification menggunakan ResNet.
  + People detection (prioritas deteksi "person").
  + Dashboard dengan live view, playback, dan analisis AI.
  + Pengaturan kamera, AI, rekaman, notifikasi, dan manajemen user.

## Project Structure
```
AINVR/
├── config/
│   └── config.py              # Pengaturan aplikasi (database, SMTP, dll.)
├── data/
│   ├── detections.db          # Database SQLite (opsional PostgreSQL)
│   ├── recordings/            # Folder untuk video tersimpan
│   ├── faces/                 # Database wajah untuk face search
│   ├── logs/                  # File log aplikasi
│   └── plots/                 # Gambar hasil overlay atau visualisasi
├── models/
│   ├── yolo/                 # Model YOLOv8 weights
│   └── resnet/               # Model ResNet untuk klasifikasi
├── scripts/
│   ├── app.py                # Flask dashboard
│   ├── tasks.py              # Celery tasks untuk AI processing
│   ├── init_db.py            # Inisialisasi database
│   ├── backup.py             # Script backup data
│   ├── face_utils.py         # Utilitas untuk face recognition/search
│   ├── utils.py              # Utilitas umum
│   └── __init__.py
├── tests/
│   ├── test_app.py           # Unit test untuk Flask
│   ├── test_ai.py            # Unit test untuk AI
│   └── __init__.py
├── README.md
├── requirements.txt
└── Dockerfile
```

## Prerequisites

- **Hardware**
  - Server: CPU 4 core, RAM 8GB (GPU disarankan untuk AI).
  - Kamera IP: Minimal 2 dengan RTSP support.
  - Penyimpanan: SSD untuk rekaman.

- **Software**
  - OS: Ubuntu 20.04+ atau Linux berbasis Debian.
  - Python: 3.8+.
  - Docker: Opsional untuk deploy MediaMTX.

- **Dependencies**
  + MediaMTX: Streaming server.
  + FFmpeg: Pemrosesan video.
  + Python Packages: `opencv-python`, `ultralytics`, `dlib`, `face_recognition`, `torch`, `torchvision`, `flask`, `flask-login`, `celery`, `redis`, `psycopg2`.
  + Database: SQLite (default) atau PostgreSQL.

- **Rekomendasi**
  > Gunakan `venv` untuk Python dependencies.
  > Siapkan SMTP server (e.g., Gmail) untuk notifikasi.

## Setup

1. **Clone the Repository**
   ```bash
   git clone <repository-url>
   cd AINVR
   ```

2. **Install Dependencies**
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```
   + For GPU support (YOLOv8, ResNet, dlib):
     ```bash
     pip install torch torchvision --extra-index-url https://download.pytorch.org/whl/cu118
     pip install dlib --verbose  # Pastikan CUDA dan cuDNN terinstall
     ```

3. **Configure Settings**
   Edit `config.py` untuk menyesuaikan:
   - Database URL (SQLite atau PostgreSQL).
   - MediaMTX API endpoint.
   - SMTP untuk notifikasi.
   Example:
   ```python
   DATABASE_URL = "sqlite:///detections.db"  # Atau "postgresql://user:password@localhost/ainvr"
   MEDIAMTX_API = "http://localhost:9997/v3/"
   SMTP_SERVER = "smtp.gmail.com"
   SMTP_PORT = 587
   SMTP_USER = "your_email@gmail.com"
   SMTP_PASSWORD = "your_app_password"
   SECRET_KEY = "your_secret_key"
   FACE_DATABASE_PATH = "data/faces/"  # Untuk face recognition/search
   ```

4. **Install MediaMTX**
   - Manual
     ```bash
     wget https://github.com/bluenviron/mediamtx/releases/download/v1.12.2/mediamtx_v1.12.2_linux_amd64.tar.gz
     tar -xvzf mediamtx_v1.12.2_linux_amd64.tar.gz
     sudo mv mediamtx /usr/local/bin/
     sudo mv mediamtx.yml /usr/local/etc/
     ```
   - Docker
     ```bash
     docker run --rm -it -e MTX_PROTOCOLS=tcp -p 8554:8554 -p 8889:8889 -v $(pwd)/mediamtx.yml:/mediamtx.yml bluenviron/mediamtx
     ```

5. **Setup Database**
   - **SQLite** (Default)
     + Tidak perlu konfigurasi, otomatis dibuat sebagai `detections.db`.
   - **PostgreSQL** (Recommended)
     ```bash
     sudo apt install postgresql
     sudo -u postgres psql
     CREATE DATABASE ainvr;
     CREATE USER ainvr_user WITH PASSWORD 'your_password';
     GRANT ALL PRIVILEGES ON DATABASE ainvr TO ainvr_user;
     \q
     ```
     + Update `config.py` dengan `DATABASE_URL = "postgresql://ainvr_user:your_password@localhost/ainvr"`.

6. **Initialize Database**
   ```bash
   python init_db.py
   ```
   > Membuat tabel `detections` (untuk hasil AI), `users` (untuk multi-user), dan `faces` (untuk face recognition).

## Building the Project

**Note**: Run all scripts using `python -m scripts.<script>` for proper module resolution.

1. **Run MediaMTX**
   ```bash
   mediamtx /usr/local/etc/mediamtx.yml
   ```
   > Pastikan kamera RTSP sudah terkonfigurasi di `mediamtx.yml`.

2. **Run Flask Dashboard**
   ```bash
   python -m scripts.app
   ```
   > Akses di `http://localhost:5000`, login dengan `admin`/`admin` atau `operator`/`operator` (ubah setelah login).

3. **Run Celery Worker (for AI Processing)**
   ```bash
   celery -A scripts.tasks worker --loglevel=info
   ```
   + Pastikan Redis berjalan:
     ```bash
     docker run -d -p 6379:6379 redis
     ```

4. **Configure Cameras**
   - Edit `/usr/local/etc/mediamtx.yml`:
     ```yaml
     paths:
       cam1:
         source: rtsp://admin:password@camera1_ip:554/stream
         record: yes
         recordPath: ./recordings/cam1/%Y-%m-%d_%H-%M-%S.mp4
       cam2:
         source: rtsp://admin:password@camera2_ip:554/stream
         record: yes
         recordPath: ./recordings/cam2/%Y-%m-%d_%H-%M-%S.mp4
     ```

## Using the Program

1. **Access Dashboard**
   - Buka `http://localhost:5000`.
   - Login sebagai Admin atau Operator.
   - > Admin bisa akses semua menu, Operator terbatas ke Home, Playback, dan AI Analysis.

2. **Set Camera Settings (Admin)**
   - Menu: `Settings > Camera`.
   - Tambah kamera: Input nama, RTSP URL, aktifkan stream.
   - > Contoh: Tambah `Cam1` dengan `rtsp://admin:password@camera1_ip:554/stream`, klik "Save".

3. **Configure Dashboard Layout (Admin)**
   - Menu: `Settings > Dashboard`.
   - Pilih layout (e.g., 4x2), assign kamera ke slot, aktifkan overlay AI.
   - > Contoh: Set 4x2, slot 1-4 untuk `Cam1`, aktifkan overlay.

4. **Configure AI Settings (Admin)**
   - Menu: `Settings > AI`.
   - Pilih jenis AI: Face Recognition, Face Search, Object Detection, Object Classification, People Detection.
   - Untuk Object Detection, pilih sub-kelas: Car, Motorcycle, dll.
   - > Contoh: Aktifkan Face Recognition untuk `Cam1`, pilih Object Detection dengan sub-kelas `Car`.

5. **Run AI Analysis (Admin/Operator)**
   - Menu: `AI Analysis`.
   - Input rentang waktu, kamera, dan jenis AI (e.g., Face Search, Object Detection).
   - > Contoh: Cari wajah tertentu di `Cam1` dari 10:00:00-11:00:00, atau deteksi `Car` di periode yang sama.

6. **Manage Recordings (Admin)**
   - Menu: `Settings > Recording`.
   - Aktifkan rekaman, set durasi (e.g., 3600 detik), retensi (e.g., 30 hari).
   - > Contoh: Rekam `Cam1` dengan durasi 1 jam, retensi 30 hari.

7. **Face Recognition/Search Example Output**
   ```
   === Face Recognition Result ===
   +----+-------------------+-----------------+
   | No | Timestamp         | Match           |
   +----+-------------------+-----------------+
   | 1  | 2025-05-12 10:05  | Person ID: 001  |
   | 2  | 2025-05-12 10:10  | Unknown         |
   +----+-------------------+-----------------+
   ```

8. **Object Detection Example Output**
   ```
   === Object Detection Result ===
   +----+-------------------+-----------------+
   | No | Timestamp         | Object          |
   +----+-------------------+-----------------+
   | 1  | 2025-05-12 10:05  | Car (0.92)      |
   | 2  | 2025-05-12 10:06  | Motorcycle (0.87)|
   +----+-------------------+-----------------+
   ```

## Maintenance

1. **Update Dependencies**
   ```bash
   pip install -r requirements.txt --upgrade
   ```

2. **Clean Up Old Recordings**
   - Manual cleanup:
     ```bash
     find ./recordings -type f -mtime +30 -delete
     ```
   - > Set retensi di `Settings > Recording` untuk otomatis.

3. **Backup Data**
   ```bash
   python -m scripts.backup --database detections.db --recordings ./recordings --output backup_$(date +%F).tar.gz
   ```

4. **Monitor Logs**
   - Cek log di `data/logs/`.
   - > Gunakan `tail -f data/logs/app.log` untuk real-time monitoring.

5. **Update MediaMTX Config**
   - Edit `/usr/local/etc/mediamtx.yml` untuk kamera baru.
   - Restart MediaMTX:
     ```bash
     sudo systemctl restart mediamtx
     ```

## Troubleshooting

- **MediaMTX Connection Error**
  - Periksa RTSP URL dan kredensial di `mediamtx.yml`.
  - > Jalankan `mediamtx` dengan `--log-level debug` untuk detail.

- **Dashboard Not Loading**
  - Pastikan Flask dan Celery berjalan.
  - > Cek port 5000 (`netstat -tuln | grep 5000`) dan restart `python -m scripts.app`.

- **AI Processing Slow**
  - Gunakan GPU dengan `pip install torch --extra-index-url https://download.pytorch.org/whl/cu118`.
  - > Verifikasi GPU: `python -c "import torch; print(torch.cuda.is_available())"`.

- **Face Recognition Not Detecting**
  - Pastikan `dlib` dan `face_recognition` terinstall.
  - > Periksa log: `data/logs/app.log` untuk error `dlib`.

- **Object Detection Missing Classes**
  - Pastikan sub-kelas (e.g., Car, Motorcycle) dipilih di `Settings > AI`.
  - > Update YOLOv8 weights di `models/yolo/` jika kelas tidak mendukung.

- **Database Error**
  - Pastikan `DATABASE_URL` di `config.py` sesuai (SQLite atau PostgreSQL).
  - > Untuk PostgreSQL, cek koneksi: `psql -h localhost -U ainvr_user -d ainvr`.

## Contributing

1. Fork the repository.
2. Create a feature branch:
   ```bash
   git checkout -b feature-name
   ```
3. Commit changes:
   ```bash
   git commit -m "Add feature"
   ```
4. Push to branch:
   ```bash
   git push origin feature-name
   ```
5. Open a pull request.

## Notes

- **Logs**: Disimpan di `data/logs/` dengan rotasi harian.
- **Recordings**: Disimpan di `./recordings/camX/`.
- **Models**: YOLOv8 di `models/yolo/`, ResNet di `models/resnet/`.
- **Security**: Gunakan HTTPS dengan Nginx reverse proxy.

## License
> TBD (Planned: MIT License)
