USE sdg_transport;

-- TAHAP 1: EKSPLORASI JOIN (KONDISI NORMALISASI)
SELECT 
    p.id_pengguna,
    p.nama_lengkap,
    COUNT(j.id_perjalanan) AS frekuensi_trip,
    SUM(j.jarak_km) AS akumulasi_jarak_km,
    SUM(j.biaya) AS total_pengeluaran
FROM pengguna p
INNER JOIN perjalanan j ON p.id_pengguna = j.id_pengguna
GROUP BY p.id_pengguna, p.nama_lengkap
ORDER BY total_pengeluaran DESC;

-- TAHAP 2: PENERAPAN DENORMALISASI (TABEL REKAPITULASI)
ALTER TABLE pengguna 
ADD COLUMN total_trip INT DEFAULT 0,
ADD COLUMN total_biaya DECIMAL(15,2) DEFAULT 0.00;

UPDATE pengguna p
SET 
    p.total_trip = (
        SELECT COUNT(j.id_perjalanan) 
        FROM perjalanan j 
        WHERE j.id_pengguna = p.id_pengguna
    ),
    p.total_biaya = (
        SELECT COALESCE(SUM(j.biaya), 0) 
        FROM perjalanan j 
        WHERE j.id_pengguna = p.id_pengguna
    );

CREATE TABLE IF NOT EXISTS rekap_pengguna (
    id_pengguna INT PRIMARY KEY,
    nama_lengkap VARCHAR(100) NOT NULL,
    jumlah_perjalanan INT NOT NULL,
    total_biaya DECIMAL(15,2) NOT NULL,
    FOREIGN KEY (id_pengguna) REFERENCES pengguna(id_pengguna) ON DELETE CASCADE
);

INSERT INTO rekap_pengguna (id_pengguna, nama_lengkap, jumlah_perjalanan, total_biaya)
SELECT 
    p.id_pengguna,
    p.nama_lengkap,
    COUNT(j.id_perjalanan),
    COALESCE(SUM(j.biaya), 0)
FROM pengguna p
INNER JOIN perjalanan j ON p.id_pengguna = j.id_pengguna
GROUP BY p.id_pengguna, p.nama_lengkap;

-- TAHAP 3: PEMBUKTIAN (MEMBACA DATA TANPA JOIN & AGREGASI)
SELECT 
    id_pengguna,
    nama_lengkap,
    jumlah_perjalanan,
    total_biaya
FROM rekap_pengguna
ORDER BY total_biaya DESC;