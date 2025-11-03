Praktikkum 1
# 3toolsdatabesar

🎯 Ringkasan Praktikum
Praktikum ini mencakup 3 tools penyimpanan data besar:

HDFS - untuk distributed file system
MongoDB - untuk NoSQL document database
Cassandra - untuk NoSQL wide-column database

📋 Persiapan Awal
Pastikan sistem Anda sudah memiliki:

Docker installed (cara termudah)
RAM minimal 8GB
Koneksi internet untuk pull images


🗂️ Bagian 1: Praktikum HDFS
Langkah 1: Setup HDFS dengan Docker
bash# Pull image Hadoop
docker pull sequenceiq/hadoop-docker:2.7.0

# Jalankan container
docker run -it sequenceiq/hadoop-docker:2.7.0 /etc/bootstrap.sh -bash
Langkah 2: Eksperimen Dasar HDFS
Setelah masuk ke container, jalankan:
bash# Membuat folder di HDFS
hdfs dfs -mkdir /praktikum

# Buat file dummy untuk testing
echo "NIM,Nama,Jurusan" > dataset.csv
echo "12345,Andi,Informatika" >> dataset.csv
echo "12346,Budi,Sistem Informasi" >> dataset.csv

# Upload file ke HDFS
hdfs dfs -put dataset.csv /praktikum/

# Lihat isi folder
hdfs dfs -ls /praktikum/

# Baca isi file
hdfs dfs -cat /praktikum/dataset.csv
📝 Latihan HDFS: File Besar
Untuk testing file besar (>100MB):
bash# Buat file dummy 150MB
dd if=/dev/zero of=large_file.txt bs=1M count=150

# Upload ke HDFS
hdfs dfs -put large_file.txt /praktikum/

# Cek block size dan replikasi
hdfs fsck /praktikum/large_file.txt -files -blocks -locations

🍃 Bagian 2: Praktikum MongoDB
Langkah 1: Setup MongoDB dengan Docker
bash# Jalankan MongoDB container
docker run -d -p 27017:27017 --name mongo mongo:latest

# Masuk ke MongoDB shell
docker exec -it mongo mongosh
Langkah 2: Eksperimen Dasar MongoDB
javascript// Pilih/buat database
use praktikum

// Insert satu data
db.mahasiswa.insertOne({ 
    nim: "12345", 
    nama: "Andi", 
    jurusan: "Informatika" 
})

// Insert banyak data
db.mahasiswa.insertMany([
    { nim: "12346", nama: "Budi", jurusan: "Sistem Informasi" },
    { nim: "12347", nama: "Citra", jurusan: "Teknik Komputer" }
])

// Lihat semua data
db.mahasiswa.find()

// Query dengan filter
db.mahasiswa.find({ jurusan: "Informatika" })

// Buat index untuk performa
db.mahasiswa.createIndex({ nim: 1 })

// Query dengan sorting
db.mahasiswa.find().sort({ nama: 1 })
📝 Latihan MongoDB: Nested JSON
javascript// Insert data dengan struktur nested
db.mahasiswa.insertOne({
    nim: "12348",
    nama: "Deni",
    jurusan: "Informatika",
    biodata: {
        alamat: {
            jalan: "Jl. Sudirman No. 123",
            kota: "Jakarta",
            provinsi: "DKI Jakarta"
        },
        kontak: {
            email: "deni@email.com",
            telepon: "081234567890"
        }
    },
    nilai: [
        { matkul: "Database", skor: 85 },
        { matkul: "Algoritma", skor: 90 }
    ]
})

// Query nested field
db.mahasiswa.find({ "biodata.alamat.kota": "Jakarta" })

// Query array
db.mahasiswa.find({ "nilai.matkul": "Database" })

🔷 Bagian 3: Praktikum Cassandra
Langkah 1: Setup Cassandra dengan Docker
bash# Jalankan Cassandra container
docker run --name cassandra -d -p 9042:9042 cassandra:latest

# Tunggu beberapa saat (Cassandra butuh waktu untuk start)
# Cek status: docker logs cassandra

# Masuk ke CQL shell
docker exec -it cassandra cqlsh
Langkah 2: Eksperimen Dasar Cassandra
sql-- Buat keyspace (seperti database)
CREATE KEYSPACE praktikum
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

-- Gunakan keyspace
USE praktikum;

-- Buat tabel
CREATE TABLE mahasiswa (
    nim text PRIMARY KEY,
    nama text,
    jurusan text
);

-- Insert data
INSERT INTO mahasiswa (nim, nama, jurusan)
VALUES ('12345', 'Budi', 'Informatika');

INSERT INTO mahasiswa (nim, nama, jurusan) 
VALUES ('12346', 'Citra', 'Sistem Informasi');

INSERT INTO mahasiswa (nim, nama, jurusan) 
VALUES ('12347', 'Dewi', 'Teknik Komputer');

-- Query semua data
SELECT * FROM mahasiswa;

-- Query dengan filter (butuh ALLOW FILTERING karena bukan partition key)
SELECT * FROM mahasiswa WHERE jurusan='Informatika' ALLOW FILTERING;
📝 Latihan Cassandra: 2 Node Cluster
Buat file docker-compose.yml:
yamlversion: '3'
services:
  cassandra-1:
    image: cassandra:latest
    container_name: cassandra-node1
    ports:
      - "9042:9042"
    environment:
      - CASSANDRA_CLUSTER_NAME=PraktikumCluster
      - CASSANDRA_SEEDS=cassandra-1
    
  cassandra-2:
    image: cassandra:latest
    container_name: cassandra-node2
    ports:
      - "9043:9042"
    environment:
      - CASSANDRA_CLUSTER_NAME=PraktikumCluster
      - CASSANDRA_SEEDS=cassandra-1
    depends_on:
      - cassandra-1
Jalankan:
bashdocker-compose up -d

# Cek status cluster
docker exec -it cassandra-node1 nodetool status

📊 Perbandingan Ketiga Tools
AspekHDFSMongoDBCassandraTipeDistributed File SystemDocument DatabaseWide-Column DatabaseBest ForBig data analytics, batch processingFlexible schema, rapid developmentHigh write throughput, time-seriesKonsistensiStrong consistencyTunable consistencyEventual consistencySkalabilitasHorizontal (add nodes)Horizontal & VerticalHorizontal (linear scale)

❓ Troubleshooting
HDFS tidak bisa start:

Pastikan port tidak bentrok
Cek logs: docker logs <container_id>

MongoDB connection refused:

Tunggu beberapa detik setelah container start
Cek: docker ps apakah container running

Cassandra lambat:

Cassandra butuh RAM besar, pastikan minimal 2GB free
Tunggu 1-2 menit setelah start container
