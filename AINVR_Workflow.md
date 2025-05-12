# AINVR Development Workflow

> This workflow outlines the steps for Mr. Don and Bejo to collaborate on developing the AINVR project efficiently.

## Overview

- **Purpose**:
  - Menyusun rencana kerja yang terstruktur untuk membangun dan mengembangkan AINVR.
  - Memastikan kolaborasi yang lancar antara Mr. Don dan Bejo.

- **Features**:
  + Pembagian tugas berdasarkan keahlian (Mr. Don: visi dan kebutuhan, Bejo: coding dan implementasi).
  + Iterasi pengembangan dengan review rutin.
  + Dokumentasi dan testing untuk stabilitas proyek.

## Project Structure
```
AINVR/
├── docs/                     # Dokumentasi rencana dan progress
├── scripts/                  # Kode utama proyek
├── tests/                    # Unit test
├── data/                     # Data pendukung (recordings, logs)
└── README.md                 # Panduan utama
```

## Prerequisites

- **Tools**
  - Git: Untuk version control.
  - Python 3.8+: Untuk development.
  - IDE: Pilihannya (VSCode, PyCharm, dll.).
  - Docker: Untuk testing lingkungan.

- **Communication**
  - Channel: Diskusi via pesan ini atau tool seperti Discord/Telegram.
  - Jadwal: Review mingguan (setiap Senin, 10:00 WIB).

- **Rekomendasi**
  > Gunakan virtual environment (`venv` atau `miniconda`) untuk konsistensi.

## Setup

1. **Initialize Repository**
   ```bash
   git clone <repository-url>
   cd AINVR
   git checkout -b main
   ```

2. **Install Development Tools**
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   pip install pytest flake8 black  # Untuk testing dan formatting
   ```

3. **Configure Environment**
   - Buat file `.env` untuk variabel sensitif (e.g., API keys, database URL).
   - Example:
     ```bash
     DATABASE_URL=sqlite:///detections.db
     SECRET_KEY=your_secret_key
     ```

4. **Set Communication Channel**
   - Tentukan alat komunikasi (e.g., Discord).
   - Set jadwal review dengan Mr. Don.

## Building the Project

**Note**: Ikuti langkah-langkah ini secara berurutan.

1. **Planning Phase**
   - Diskusi visi dan fitur dengan Mr. Don.
   - Buat daftar tugas di `docs/tasks.md`.
   - > Contoh: Tambah FAISS untuk face search, optimasi YOLOv8.

2. **Development Phase**
   - Bejo koding fitur berdasarkan tugas.
   - Mr. Don review kode dan beri feedback.
   - Commit secara rutin:
     ```bash
     git add .
     git commit -m "Add feature X"
     git push origin feature-branch
     ```

3. **Testing Phase**
   - Jalankan unit test:
     ```bash
     pytest tests/
     ```
   - Perbaiki bug berdasarkan hasil test.
   - > Pastikan coverage minimal 80% (pakai `pytest --cov`).

4. **Review and Merge**
   - Mr. Don dan Bejo review perubahan di branch.
   - Merge ke `main` setelah approved:
     ```bash
     git checkout main
     git merge feature-branch
     git push origin main
     ```

## Using the Workflow

1. **Assign Tasks**
   - Mr. Don tentukan prioritas (e.g., dashboard thumbnail, AI analysis).
   - Bejo breakdown ke sub-tugas di `docs/tasks.md`.

2. **Track Progress**
   - Update status di `docs/progress.md` setiap hari.
   - > Contoh: "Bejo: Selesai kode Flask dashboard, 80% tested."

3. **Iterate**
   - Ulangi Planning, Development, Testing, Review setiap minggu.
   - Tambah fitur baru berdasarkan feedback Mr. Don.

4. **Deploy Test**
   - Test di lingkungan lokal atau Docker:
     ```bash
     docker-compose up --build
     ```
   - Laporkan hasil ke Mr. Don.

## Maintenance

1. **Update Documentation**
   - Perbarui README.md dan docs/ setelah setiap iterasi.
   - > Gunakan `git commit -m "Update docs"`

2. **Clean Up Code**
   - Jalankan formatter:
     ```bash
     black scripts/
     flake8 scripts/
     ```

3. **Backup Repository**
   - Push ke remote secara rutin:
     ```bash
     git push origin main
     ```

4. **Monitor Issues**
   - Cek log di `data/logs/` untuk error.
   - > Laporkan ke Mr. Don jika ada masalah kritis.

## Troubleshooting

- **Git Conflict**
  - Resolve dengan `git mergetool` atau manual.
  - > Push ulang setelah selesai.

- **Test Failure**
  - Cek output `pytest` dan perbaiki kode.
  - > Pastikan dependensi terbaru.

- **Docker Error**
  - Verifikasi `Dockerfile` dan restart Docker:
    ```bash
    sudo systemctl restart docker
    ```

## Contributing

1. Fork the repository (jika eksternal).
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

- **Collaboration**: Mr. Don fokus visi, Bejo handle coding.
- **Timeline**: Target iterasi 1 minggu per fitur.
- **Tools**: Gunakan GitHub Issues untuk tracking opsional.

## License
> TBD (Planned: MIT License)