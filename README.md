# Pertemuan13_OOP

## Profil
| Variable | Isi |
| -------- | --- |
| **Nama** | Intan Virginia Aulia Putri |
| **NIM** | 312310657 |
| **Kelas** | TI.23.A.6 |
| **Mata Kuliah** | Pemrograman Orientasi Objek |
| **Link vidio Penjelasan** | [https://youtu.be/pOxdVMJguDI?si=iCp3rBvu8TWrRsuN] |

# MVC (Model-View-Controller) dengan Pendekatan OOP

## Apa itu MVC?

# MVC (Model-View-Controller)

MVC adalah sebuah pola desain arsitektur yang populer dalam pengembangan perangkat lunak. Singkatnya, MVC adalah cara untuk memisahkan logika aplikasi menjadi tiga komponen utama:

- **Model**: Mewakili data dan logika bisnis aplikasi. Ini adalah bagian "otak" dari aplikasi, yang mengelola data dan melakukan perhitungan.
- **View**: Menampilkan data kepada pengguna dalam bentuk antarmuka pengguna (user interface). View tidak memiliki logika, hanya menampilkan data yang diberikan oleh model.
- **Controller**: Menjembatani antara model dan view. Controller menerima input dari pengguna, memproses input tersebut, dan kemudian memperbarui model atau view sesuai kebutuhan.

# Aplikasi Pengelolaan Data Mahasiswa
Aplikasi ini adalah aplikasi sederhana yang saya buat untuk mengelola data mahasiswa dan nilai mereka menggunakan **database MySQL**. Aplikasi ini dirancang untuk mempermudah pencatatan data mahasiswa seperti nama, NIM, program studi, serta pengelolaan nilai mereka dalam berbagai mata kuliah.  
Fokus utama dari aplikasi ini adalah memberikan solusi sederhana namun efektif untuk manajemen data, dengan fitur-fitur dasar seperti menambah, mengedit, menghapus, dan menampilkan data secara langsung.

# BaseModel.java
``` Java
package classes;

import java.sql.*;
import java.util.List;

public abstract class BaseModel<T> {
    protected Connection connection;
    
    public BaseModel(Connection connection) {
        this.connection = connection;
    }
    
    public abstract List<T> findAll() throws SQLException;
    public abstract T findById(int id) throws SQLException;
    public abstract boolean insert(T object) throws SQLException;
    public abstract boolean update(T object) throws SQLException;
    public abstract boolean delete(int id) throws SQLException;
}
```
### **Penjelasan BaseModel.java**  

`BaseModel` adalah class abstrak yang menjadi kerangka dasar untuk mengelola operasi database menggunakan Java.  

#### **Elemen Penting**  
1. **`protected Connection connection`**  
   - Menyimpan koneksi ke database, diakses oleh class turunan.  

2. **Konstruktor**  
   - **`BaseModel(Connection connection)`**: Menginisialisasi koneksi database.  

3. **Method Abstrak**  
   - **`findAll()`**: Mengambil semua data.  
   - **`findById(int id)`**: Mengambil data berdasarkan ID.  
   - **`insert(T object)`**: Menambahkan data baru.  
   - **`update(T object)`**: Memperbarui data.  
   - **`delete(int id)`**: Menghapus data berdasarkan ID.  

#### **Fungsi Utama**  
- Digunakan sebagai kerangka CRUD untuk tabel database, memungkinkan reusabilitas dan modularitas kode.

# Database.java
``` Java
package classes;

import java.sql.*;

public class Database {
    private static final String DRIVER = "com.mysql.cj.jdbc.Driver";
    private static final String URL = "jdbc:mysql://localhost:3306/akademik";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "";
    
    public static Connection getConnection() throws SQLException {
        try {
            Class.forName(DRIVER);
            return DriverManager.getConnection(URL, USERNAME, PASSWORD);
        } catch (ClassNotFoundException e) {
            throw new SQLException("Database driver not found", e);
        }
    }
}
```
### **Penjelasan Database.java**  

`Database` adalah class utilitas yang bertanggung jawab untuk mengelola koneksi ke database MySQL.  

#### **Elemen Utama**  
1. **Konstanta**  
   - **`DRIVER`**: Nama driver JDBC untuk MySQL.  
   - **`URL`**: Alamat database (dalam contoh ini, database bernama `akademik` di localhost).  
   - **`USERNAME`** dan **`PASSWORD`**: Kredensial untuk mengakses database.  

2. **Method `getConnection()`**  
   - Menginisialisasi koneksi ke database.  
   - **`Class.forName(DRIVER)`**: Memuat driver JDBC.  
   - **`DriverManager.getConnection(URL, USERNAME, PASSWORD)`**: Membuat dan mengembalikan koneksi ke database.  
   - **`ClassNotFoundException`**: Ditangani jika driver JDBC tidak ditemukan.  

#### **Fungsi Utama**  
- Memberikan koneksi yang siap digunakan untuk operasi database.  
- Menyederhanakan pengelolaan koneksi dalam aplikasi.

RowMapper.java
``` Java
package classes;

import java.sql.ResultSet;
import java.sql.SQLException;

public interface RowMapper<T> {
    T mapRow(ResultSet rs) throws SQLException;
}
```
### **Penjelasan RowMapper.java**  

`RowMapper` adalah sebuah interface yang digunakan untuk memetakan baris data dari hasil query SQL (`ResultSet`) ke dalam objek Java.  

#### **Elemen Utama**  
1. **Method `mapRow(ResultSet rs)`**  
   - **Parameter**:  
     - `ResultSet rs`: Objek yang berisi hasil query SQL.  
   - **Output**:  
     - Mengembalikan objek generik (`<T>`) yang merepresentasikan satu baris data dari database.  
   - **Throws**:  
     - `SQLException` jika terjadi kesalahan saat membaca data dari `ResultSet`.  

#### **Fungsi Utama**  
- Digunakan untuk memisahkan logika pemetaan data dari hasil query ke objek Java, membuat kode lebih modular dan bersih.  

# MahasiswaController.java
``` Java
package controller;

import java.sql.Connection;
import java.sql.SQLException;
import java.util.List;
import javax.swing.JOptionPane;
import model.*;
import view.FormMahasiswa;
import view.FormNilai;

public class MahasiswaController {
    private MahasiswaModel model;
    private FormMahasiswa view;
    private Connection connection;
    
    public MahasiswaController(MahasiswaModel model, FormMahasiswa view, Connection connection) {
        this.model = model;
        this.view = view;
        this.connection = connection;
        
        // Initialize view event handlers
        this.view.addSaveListener(e -> saveMahasiswa());
        this.view.addDeleteListener(e -> deleteMahasiswa());
        this.view.addClearListener(e -> clearForm());
        this.view.addNilaiListener(e -> showNilai());
        
        refreshTable();
    }
    
    private void saveMahasiswa() {
        try {
            Mahasiswa mahasiswa = view.getMahasiswa();
            if (mahasiswa.getId() == 0) {
                model.insert(mahasiswa);
            } else {
                model.update(mahasiswa);
            }
            clearForm();
            refreshTable();
            JOptionPane.showMessageDialog(view, "Data saved successfully!");
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(view, "Error saving data: " + ex.getMessage());
        }
    }
    
    private void deleteMahasiswa() {
        try {
            int id = view.getSelectedMahasiswaId();
            if (id > 0) {
                if (model.delete(id)) {
                    clearForm();
                    refreshTable();
                    JOptionPane.showMessageDialog(view, "Data deleted successfully!");
                }
            }
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(view, "Error deleting data: " + ex.getMessage());
        }
    }
    
    private void clearForm() {
        view.clearForm();
    }
    
    private void refreshTable() {
        try {
            List<Mahasiswa> mahasiswas = model.findAll();
            view.setMahasiswas(mahasiswas);
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(view, "Error loading data: " + ex.getMessage());
        }
    }

    private void showNilai() {
        try {
            int mahasiswaId = view.getSelectedMahasiswaId();
            if (mahasiswaId > 0) {
                // Get the selected Mahasiswa data
                Mahasiswa selectedMahasiswa = model.findById(mahasiswaId);
                if (selectedMahasiswa != null) {
                    // Create FormNilai with parent frame
                    FormNilai nilaiForm = new FormNilai(view);
                    // Set mahasiswa info after creation
                    nilaiForm.setMahasiswaInfo(selectedMahasiswa);
                    
                    NilaiModel nilaiModel = new NilaiModel(connection);
                    NilaiController nilaiController = new NilaiController(nilaiModel, nilaiForm, mahasiswaId);
                    nilaiForm.setVisible(true);
                } else {
                    JOptionPane.showMessageDialog(view, 
                        "Error: Selected student data not found!", 
                        "Error", 
                        JOptionPane.ERROR_MESSAGE);
                }
            } else {
                JOptionPane.showMessageDialog(view, 
                    "Please select a student first!", 
                    "Warning", 
                    JOptionPane.WARNING_MESSAGE);
            }
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(view, 
                "Database error: " + ex.getMessage(), 
                "Error", 
                JOptionPane.ERROR_MESSAGE);
        }
    }
}
```
### **Penjelasan MahasiswaController.java**

`MahasiswaController` adalah class yang berperan sebagai **controller** dalam pola desain **MVC** (Model-View-Controller). Class ini menjembatani interaksi antara **view** (`FormMahasiswa`) dan **model** (`MahasiswaModel`) untuk mengelola data mahasiswa.

#### **Komponen Utama**

1. **Atribut**
   - **`MahasiswaModel model`**: Objek model yang digunakan untuk operasi database (CRUD) terkait mahasiswa.
   - **`FormMahasiswa view`**: Objek view (antarmuka pengguna) untuk menampilkan data mahasiswa.
   - **`Connection connection`**: Koneksi ke database untuk operasi model.

2. **Konstruktor**
   - **`MahasiswaController(MahasiswaModel model, FormMahasiswa view, Connection connection)`**:
     - Menginisialisasi **model**, **view**, dan **connection**.
     - Menyambungkan event handler (aksi tombol pada view) ke method controller, seperti `saveMahasiswa`, `deleteMahasiswa`, dll.
     - Memuat data awal ke tabel menggunakan `refreshTable()`.

3. **Method Utama**
   - **`saveMahasiswa()`**:
     - Menyimpan atau memperbarui data mahasiswa.
     - Mengambil data mahasiswa dari form (`view.getMahasiswa()`), lalu memanggil method **`insert`** atau **`update`** dari model.

   - **`deleteMahasiswa()`**:
     - Menghapus data mahasiswa berdasarkan ID yang dipilih (`view.getSelectedMahasiswaId()`).
     - Memanggil method **`delete`** dari model.

   - **`clearForm()`**:
     - Menghapus input pada form mahasiswa.

   - **`refreshTable()`**:
     - Mengambil semua data mahasiswa dari database (`model.findAll()`) dan memperbarui tampilan tabel di view (`view.setMahasiswas()`).

   - **`showNilai()`**:
     - Menampilkan form nilai (`FormNilai`) untuk mahasiswa yang dipilih.
     - Membuat objek **`NilaiModel`** dan **`NilaiController`** untuk mengelola data nilai terkait mahasiswa.

#### **Fungsi Utama**
- Mengelola interaksi antara pengguna (via **FormMahasiswa**) dan data mahasiswa (via **MahasiswaModel**).
- Memastikan data selalu sinkron antara database dan tampilan.

#### **Catatan Tambahan**
1. **Event Listener pada View**  
   Konstruktor menambahkan **event listener** untuk menangani aksi tombol pada form:
   - **`addSaveListener`** → Tombol Simpan.
   - **`addDeleteListener`** → Tombol Hapus.
   - **`addClearListener`** → Tombol Clear.
   - **`addNilaiListener`** → Tombol Lihat Nilai.

2. **Error Handling**  
   Menggunakan **try-catch** untuk menangani error yang mungkin terjadi, seperti kegagalan database atau data tidak valid.

3. **Integrasi dengan Form Nilai**
   - **`FormNilai`** digunakan untuk menampilkan data nilai mahasiswa yang dipilih.
   - Controller ini membuat dan mengelola instance **`NilaiController`** dan **`NilaiModel`** untuk mengelola nilai.

# Nilai Controller.java
``` Java
package controller;

import java.sql.SQLException;
import java.util.List;
import javax.swing.JOptionPane;
import model.*;
import view.FormNilai;

public class NilaiController {
    private NilaiModel model;
    private FormNilai view;
    private int mahasiswaId;
    
    public NilaiController(NilaiModel model, FormNilai view, int mahasiswaId) {
        this.model = model;
        this.view = view;
        this.mahasiswaId = mahasiswaId;
        
        // Initialize view event handlers
        this.view.addSaveListener(e -> saveNilai());
        this.view.addDeleteListener(e -> deleteNilai());
        this.view.addClearListener(e -> clearForm());
        
        refreshTable();
    }
    
    private void saveNilai() {
        try {
            // Get nilai from form and set mahasiswaId
            Nilai nilai = view.getNilai();
            nilai.setMahasiswaId(mahasiswaId);
            
            // Validate Mata Kuliah
            if (nilai.getMataKuliah() == null || nilai.getMataKuliah().trim().isEmpty()) {
                JOptionPane.showMessageDialog(view, 
                    "Mata Kuliah tidak boleh kosong!", 
                    "Validasi Error", 
                    JOptionPane.WARNING_MESSAGE);
                return;
            }
            
            // Validate Semester
            if (nilai.getSemester() == null || nilai.getSemester().trim().isEmpty()) {
                JOptionPane.showMessageDialog(view, 
                    "Semester tidak boleh kosong!", 
                    "Validasi Error", 
                    JOptionPane.WARNING_MESSAGE);
                return;
            }
            
            // Validate Nilai
            if (nilai.getNilai() < 0 || nilai.getNilai() > 100) {
                JOptionPane.showMessageDialog(view, 
                    "Nilai harus antara 0-100!", 
                    "Validasi Error", 
                    JOptionPane.WARNING_MESSAGE);
                return;
            }

            // Check if this is a new record or update
            if (nilai.getId() == 0) {
                // New record
                if (model.insert(nilai)) {
                    JOptionPane.showMessageDialog(view, 
                        "Nilai berhasil disimpan!", 
                        "Sukses", 
                        JOptionPane.INFORMATION_MESSAGE);
                    clearForm();
                    refreshTable();
                } else {
                    JOptionPane.showMessageDialog(view, 
                        "Gagal menyimpan nilai!", 
                        "Error", 
                        JOptionPane.ERROR_MESSAGE);
                }
            } else {
                // Update existing record
                int confirm = JOptionPane.showConfirmDialog(view,
                    "Apakah Anda yakin ingin mengupdate nilai ini?",
                    "Konfirmasi Update",
                    JOptionPane.YES_NO_OPTION);
                    
                if (confirm == JOptionPane.YES_OPTION) {
                    if (model.update(nilai)) {
                        JOptionPane.showMessageDialog(view, 
                            "Nilai berhasil diupdate!", 
                            "Sukses", 
                            JOptionPane.INFORMATION_MESSAGE);
                        clearForm();
                        refreshTable();
                    } else {
                        JOptionPane.showMessageDialog(view, 
                            "Gagal mengupdate nilai!", 
                            "Error", 
                            JOptionPane.ERROR_MESSAGE);
                    }
                }
            }
        } catch (Exception ex) {
            JOptionPane.showMessageDialog(view, 
                "Error: " + ex.getMessage(), 
                "Error", 
                JOptionPane.ERROR_MESSAGE);
        }
    }
    
    private void deleteNilai() {
        try {
            int id = view.getSelectedNilaiId();
            if (id > 0) {
                int confirm = JOptionPane.showConfirmDialog(view,
                    "Apakah Anda yakin ingin menghapus nilai ini?",
                    "Konfirmasi Hapus",
                    JOptionPane.YES_NO_OPTION);
                    
                if (confirm == JOptionPane.YES_OPTION) {
                    if (model.delete(id)) {
                        JOptionPane.showMessageDialog(view, 
                            "Nilai berhasil dihapus!", 
                            "Sukses", 
                            JOptionPane.INFORMATION_MESSAGE);
                        clearForm();
                        refreshTable();
                    } else {
                        JOptionPane.showMessageDialog(view, 
                            "Gagal menghapus nilai!", 
                            "Error", 
                            JOptionPane.ERROR_MESSAGE);
                    }
                }
            } else {
                JOptionPane.showMessageDialog(view, 
                    "Pilih nilai yang akan dihapus!", 
                    "Peringatan", 
                    JOptionPane.WARNING_MESSAGE);
            }
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(view, 
                "Error database: " + ex.getMessage(), 
                "Error", 
                JOptionPane.ERROR_MESSAGE);
        }
    }
    
    private void clearForm() {
        view.clearForm();
    }
    
    private void refreshTable() {
        try {
            List<Nilai> nilaiList = model.findByMahasiswaId(mahasiswaId);
            view.setNilaiList(nilaiList);
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(view, 
                "Error memuat data: " + ex.getMessage(), 
                "Error", 
                JOptionPane.ERROR_MESSAGE);
        }
    }
}
```
### **Penjelasan NilaiController.java**

`NilaiController` adalah class controller yang mengelola interaksi antara **view** (`FormNilai`) dan **model** (`NilaiModel`) dalam pengelolaan data nilai mahasiswa. Controller ini memastikan data nilai mahasiswa ditampilkan, disimpan, diperbarui, dan dihapus dengan benar.

#### **Komponen Utama**

1. **Atribut**
   - **`NilaiModel model`**: Objek model untuk melakukan operasi database terkait data nilai.
   - **`FormNilai view`**: Objek view untuk menampilkan dan mengelola data nilai melalui antarmuka pengguna.
   - **`int mahasiswaId`**: ID mahasiswa yang nilainya sedang dikelola.

2. **Konstruktor**
   - **`NilaiController(NilaiModel model, FormNilai view, int mahasiswaId)`**:
     - Menginisialisasi **model**, **view**, dan **mahasiswaId**.
     - Menambahkan event handler untuk tombol aksi di form (simpan, hapus, dan clear).
     - Memuat data nilai ke tabel melalui method `refreshTable()`.

3. **Method Utama**
   - **`saveNilai()`**:
     - Menyimpan data nilai baru atau memperbarui data nilai yang sudah ada.
     - Melakukan validasi data input, seperti:
       - **Mata Kuliah** tidak boleh kosong.
       - **Semester** tidak boleh kosong.
       - **Nilai** harus dalam rentang 0-100.
     - Menampilkan pesan sukses atau error menggunakan **`JOptionPane`**.

   - **`deleteNilai()`**:
     - Menghapus data nilai berdasarkan ID yang dipilih.
     - Meminta konfirmasi pengguna sebelum menghapus data.

   - **`clearForm()`**:
     - Membersihkan semua input pada form nilai.

   - **`refreshTable()`**:
     - Memuat data nilai mahasiswa berdasarkan `mahasiswaId`.
     - Mengambil data dari database (`model.findByMahasiswaId()`) dan memperbarui tabel di view (`view.setNilaiList()`).

#### **Fungsi Utama**
- Memastikan sinkronisasi antara data nilai di database dan tampilan pengguna.
- Memvalidasi input pengguna sebelum menyimpan atau memperbarui data.
- Menangani semua operasi pengelolaan nilai seperti tambah, edit, dan hapus dengan mudah.

#### **Catatan Tambahan**
1. **Validasi Data Input**
   - Validasi seperti **Mata Kuliah tidak boleh kosong** dan **Nilai harus dalam rentang 0-100** menjaga agar data yang masuk ke database valid.

2. **Konfirmasi Pengguna**
   - Sebelum melakukan penghapusan atau pembaruan, controller meminta konfirmasi dari pengguna untuk mencegah kesalahan.

3. **Error Handling**
   - Menggunakan **try-catch** untuk menangkap dan menampilkan error yang mungkin terjadi selama operasi database.

# Mahasiswa.java
``` Java
package model;

public class Mahasiswa {
    private int id;
    private String nim;
    private String nama;
    private String jurusan;
    private String angkatan;
    
    public Mahasiswa() {}
    
    public Mahasiswa(int id, String nim, String nama, String jurusan, String angkatan) {
        this.id = id;
        this.nim = nim;
        this.nama = nama;
        this.jurusan = jurusan;
        this.angkatan = angkatan;
    }
    
    // Getters and Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    
    public String getNim() { return nim; }
    public void setNim(String nim) { this.nim = nim; }
    
    public String getNama() { return nama; }
    public void setNama(String nama) { this.nama = nama; }
    
    public String getJurusan() { return jurusan; }
    public void setJurusan(String jurusan) { this.jurusan = jurusan; }
    
    public String getAngkatan() { return angkatan; }
    public void setAngkatan(String angkatan) { this.angkatan = angkatan; }
}
```

# Nilai.java
``` Java
package model;

public class Nilai {
    private int id;
    private int mahasiswaId;
    private String mataKuliah;
    private String semester;
    private int nilai;
    
    public Nilai() {}
    
    public Nilai(int id, int mahasiswaId, String mataKuliah, String semester, int nilai) {
        this.id = id;
        this.mahasiswaId = mahasiswaId;
        this.mataKuliah = mataKuliah;
        this.semester = semester;
        this.nilai = nilai;
    }
    
    // Getters and Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    
    public int getMahasiswaId() { return mahasiswaId; }
    public void setMahasiswaId(int mahasiswaId) { this.mahasiswaId = mahasiswaId; }
    
    public String getMataKuliah() { return mataKuliah; }
    public void setMataKuliah(String mataKuliah) { this.mataKuliah = mataKuliah; }
    
    public String getSemester() { return semester; }
    public void setSemester(String semester) { this.semester = semester; }
    
    public int getNilai() { return nilai; }
    public void setNilai(int nilai) { this.nilai = nilai; }
}
```
### **Penjelasan Kode: Mahasiswa.java dan Nilai.java**

#### **Mahasiswa.java**

`Mahasiswa` adalah model yang merepresentasikan data mahasiswa dalam aplikasi, seperti ID, NIM, nama, jurusan, dan angkatan. Class ini bertugas menyimpan informasi terkait mahasiswa yang nantinya akan dikelola dalam aplikasi.

- **Atribut**:
  - **`id`**: ID unik mahasiswa.
  - **`nim`**: Nomor Induk Mahasiswa.
  - **`nama`**: Nama mahasiswa.
  - **`jurusan`**: Jurusan mahasiswa.
  - **`angkatan`**: Tahun angkatan mahasiswa.

- **Konstruktor**:
  - Terdapat konstruktor default tanpa parameter dan konstruktor dengan parameter untuk inisialisasi data mahasiswa.

- **Getter dan Setter**:
  - Setiap atribut memiliki metode **getter** (untuk mengambil nilai) dan **setter** (untuk mengubah nilai) agar data dapat diakses dan dimodifikasi.

#### **Nilai.java**

`Nilai` adalah model yang merepresentasikan data nilai mahasiswa, termasuk ID, ID mahasiswa, mata kuliah, semester, dan nilai. Class ini bertanggung jawab untuk menyimpan informasi terkait nilai yang diperoleh oleh mahasiswa dalam setiap mata kuliah dan semester tertentu.

- **Atribut**:
  - **`id`**: ID unik nilai (untuk setiap record).
  - **`mahasiswaId`**: ID mahasiswa yang nilai-nya disimpan.
  - **`mataKuliah`**: Nama mata kuliah yang diambil.
  - **`semester`**: Semester mata kuliah tersebut.
  - **`nilai`**: Nilai yang diperoleh mahasiswa dalam mata kuliah tersebut.

- **Konstruktor**:
  - Konstruktor default tanpa parameter dan konstruktor dengan parameter untuk inisialisasi nilai mahasiswa.

- **Getter dan Setter**:
  - Sama seperti `Mahasiswa`, setiap atribut pada `Nilai` memiliki **getter** dan **setter** untuk memudahkan pengambilan dan perubahan data.

#### **Tujuan**
- **Mahasiswa** dan **Nilai** adalah **model** yang menyimpan data yang akan dikelola dalam aplikasi (misalnya, menambah, memperbarui, atau menghapus data).
- Data ini digunakan oleh **controller** untuk memanipulasi data mahasiswa dan nilai, serta menampilkannya melalui **view**.

# MahasiswaModel.java
``` Java
package model;

import classes.BaseModel;
import classes.RowMapper;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class MahasiswaModel extends BaseModel<Mahasiswa> {
    public MahasiswaModel(Connection connection) {
        super(connection);
    }
    
    private RowMapper<Mahasiswa> mapper = rs -> new Mahasiswa(
        rs.getInt("id"),
        rs.getString("nim"),
        rs.getString("nama"),
        rs.getString("jurusan"),
        rs.getString("angkatan")
    );
    
    @Override
    public List<Mahasiswa> findAll() throws SQLException {
        List<Mahasiswa> result = new ArrayList<>();
        try (Statement stmt = connection.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM mahasiswa")) {
            while (rs.next()) {
                result.add(mapper.mapRow(rs));
            }
        }
        return result;
    }
    
    @Override
    public Mahasiswa findById(int id) throws SQLException {
        String sql = "SELECT * FROM mahasiswa WHERE id = ?";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setInt(1, id);
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    return mapper.mapRow(rs);
                }
            }
        }
        return null;
    }
    
    @Override
    public boolean insert(Mahasiswa mahasiswa) throws SQLException {
        String sql = "INSERT INTO mahasiswa (nim, nama, jurusan, angkatan) VALUES (?, ?, ?, ?)";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setString(1, mahasiswa.getNim());
            stmt.setString(2, mahasiswa.getNama());
            stmt.setString(3, mahasiswa.getJurusan());
            stmt.setString(4, mahasiswa.getAngkatan());
            return stmt.executeUpdate() > 0;
        }
    }
    
    @Override
    public boolean update(Mahasiswa mahasiswa) throws SQLException {
        String sql = "UPDATE mahasiswa SET nim=?, nama=?, jurusan=?, angkatan=? WHERE id=?";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setString(1, mahasiswa.getNim());
            stmt.setString(2, mahasiswa.getNama());
            stmt.setString(3, mahasiswa.getJurusan());
            stmt.setString(4, mahasiswa.getAngkatan());
            stmt.setInt(5, mahasiswa.getId());
            return stmt.executeUpdate() > 0;
        }
    }
    
    @Override
    public boolean delete(int id) throws SQLException {
        String sql = "DELETE FROM mahasiswa WHERE id=?";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setInt(1, id);
            return stmt.executeUpdate() > 0;
        }
    }
}
```

# NilaiModel.java
``` Java
package model;

import classes.BaseModel;
import classes.RowMapper;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class NilaiModel extends BaseModel<Nilai> {
    public NilaiModel(Connection connection) {
        super(connection);
    }
    
    private RowMapper<Nilai> mapper = rs -> new Nilai(
        rs.getInt("id"),
        rs.getInt("mahasiswa_id"),
        rs.getString("mata_kuliah"),
        rs.getString("semester"),
        rs.getInt("nilai")
    );
    
    @Override
    public List<Nilai> findAll() throws SQLException {
        List<Nilai> result = new ArrayList<>();
        try (Statement stmt = connection.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM nilai")) {
            while (rs.next()) {
                result.add(mapper.mapRow(rs));
            }
        }
        return result;
    }
    
    public List<Nilai> findByMahasiswaId(int mahasiswaId) throws SQLException {
        List<Nilai> result = new ArrayList<>();
        String sql = "SELECT * FROM nilai WHERE mahasiswa_id = ?";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setInt(1, mahasiswaId);
            try (ResultSet rs = stmt.executeQuery()) {
                while (rs.next()) {
                    result.add(mapper.mapRow(rs));
                }
            }
        }
        return result;
    }
    
    @Override
    public Nilai findById(int id) throws SQLException {
        String sql = "SELECT * FROM nilai WHERE id = ?";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setInt(1, id);
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    return mapper.mapRow(rs);
                }
            }
        }
        return null;
    }
    
    @Override
    public boolean insert(Nilai nilai) throws SQLException {
        String sql = "INSERT INTO nilai (mahasiswa_id, mata_kuliah, semester, nilai) VALUES (?, ?, ?, ?)";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setInt(1, nilai.getMahasiswaId());
            stmt.setString(2, nilai.getMataKuliah());
            stmt.setString(3, nilai.getSemester());
            stmt.setInt(4, nilai.getNilai());
            return stmt.executeUpdate() > 0;
        }
    }
    
    @Override
    public boolean update(Nilai nilai) throws SQLException {
        String sql = "UPDATE nilai SET mahasiswa_id=?, mata_kuliah=?, semester=?, nilai=? WHERE id=?";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setInt(1, nilai.getMahasiswaId());
            stmt.setString(2, nilai.getMataKuliah());
            stmt.setString(3, nilai.getSemester());
            stmt.setInt(4, nilai.getNilai());
            stmt.setInt(5, nilai.getId());
            return stmt.executeUpdate() > 0;
        }
    }
    
    @Override
    public boolean delete(int id) throws SQLException {
        String sql = "DELETE FROM nilai WHERE id=?";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setInt(1, id);
            return stmt.executeUpdate() > 0;
        }
    }
}
```
### **Penjelasan Kode: MahasiswaModel.java dan NilaiModel.java**

#### **MahasiswaModel.java**

`MahasiswaModel` adalah **model** yang mengelola operasi database untuk entitas **Mahasiswa**. Model ini berfungsi untuk melakukan operasi CRUD (Create, Read, Update, Delete) pada tabel `mahasiswa` dalam database.

- **Konstruktor**:
  - Menerima objek `Connection` yang digunakan untuk berinteraksi dengan database.
  
- **RowMapper**:
  - **Mapper** digunakan untuk mengubah hasil dari `ResultSet` menjadi objek `Mahasiswa`. Setiap baris yang diambil dari database akan dipetakan ke objek `Mahasiswa`.

- **Metode**:
  - **`findAll()`**: Mengambil seluruh data mahasiswa dari tabel `mahasiswa` dan mengembalikan daftar mahasiswa.
  - **`findById(int id)`**: Mengambil data mahasiswa berdasarkan ID.
  - **`insert(Mahasiswa mahasiswa)`**: Menambahkan data mahasiswa baru ke dalam tabel.
  - **`update(Mahasiswa mahasiswa)`**: Memperbarui data mahasiswa berdasarkan ID.
  - **`delete(int id)`**: Menghapus data mahasiswa berdasarkan ID.

#### **NilaiModel.java**

`NilaiModel` adalah **model** yang mengelola operasi database untuk entitas **Nilai**. Model ini digunakan untuk mengelola nilai yang diberikan kepada mahasiswa, termasuk operasi CRUD pada tabel `nilai` dalam database.

- **Konstruktor**:
  - Menerima objek `Connection` yang digunakan untuk berinteraksi dengan database.

- **RowMapper**:
  - **Mapper** digunakan untuk mengubah hasil dari `ResultSet` menjadi objek `Nilai`.

- **Metode**:
  - **`findAll()`**: Mengambil seluruh data nilai dari tabel `nilai` dan mengembalikan daftar nilai.
  - **`findByMahasiswaId(int mahasiswaId)`**: Mengambil seluruh nilai mahasiswa berdasarkan `mahasiswa_id`. Ini memungkinkan pengambilan nilai berdasarkan mahasiswa tertentu.
  - **`findById(int id)`**: Mengambil data nilai berdasarkan ID.
  - **`insert(Nilai nilai)`**: Menambahkan data nilai baru untuk mahasiswa.
  - **`update(Nilai nilai)`**: Memperbarui data nilai berdasarkan ID.
  - **`delete(int id)`**: Menghapus data nilai berdasarkan ID.

### **Perbedaan dan Tujuan**
- **MahasiswaModel** mengelola data mahasiswa, sementara **NilaiModel** mengelola data nilai mahasiswa.
- Kedua kelas ini mewarisi `BaseModel`, yang kemungkinan adalah kelas dasar dengan implementasi umum untuk akses database, seperti pembuatan koneksi atau eksekusi query.

### **Fitur Umum**:
- Keduanya memiliki **CRUD operations** (Create, Read, Update, Delete) untuk memanipulasi data mahasiswa dan nilai dalam database.
- **RowMapper** digunakan untuk memetakan hasil query SQL menjadi objek Java, sehingga memudahkan pengolahan data.

# FormMahasiswa.java
``` Java
package view;

import java.awt.*;
import java.awt.event.ActionListener;
import java.util.List;
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import model.Mahasiswa;

public class FormMahasiswa extends JFrame {
    private JTextField txtId;
    private JTextField txtNim;
    private JTextField txtNama;
    private JTextField txtJurusan;
    private JTextField txtAngkatan;
    private JButton btnSave;
    private JButton btnDelete;
    private JButton btnClear;
    private JButton btnNilai;
    private JTable tableMahasiswa;
    private DefaultTableModel tableModel;
    
    public FormMahasiswa() {
        initComponents();
    }
    
    private void initComponents() {
        setTitle("Form Mahasiswa");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout());
        
        // Form Panel
        JPanel formPanel = new JPanel(new GridLayout(5, 2, 5, 5));
        formPanel.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));
        
        txtId = new JTextField();
        txtId.setEditable(false);
        txtNim = new JTextField();
        txtNama = new JTextField();
        txtJurusan = new JTextField();
        txtAngkatan = new JTextField();
        
        formPanel.add(new JLabel("ID:"));
        formPanel.add(txtId);
        formPanel.add(new JLabel("NIM:"));
        formPanel.add(txtNim);
        formPanel.add(new JLabel("Nama:"));
        formPanel.add(txtNama);
        formPanel.add(new JLabel("Jurusan:"));
        formPanel.add(txtJurusan);
        formPanel.add(new JLabel("Angkatan:"));
        formPanel.add(txtAngkatan);
        
        // Button Panel
        JPanel buttonPanel = new JPanel();
        btnSave = new JButton("Save");
        btnDelete = new JButton("Delete");
        btnClear = new JButton("Clear");
        btnNilai = new JButton("Lihat Nilai");
        
        buttonPanel.add(btnSave);
        buttonPanel.add(btnDelete);
        buttonPanel.add(btnClear);
        buttonPanel.add(btnNilai);
        
        // Table
        String[] columns = {"ID", "NIM", "Nama", "Jurusan", "Angkatan"};
        tableModel = new DefaultTableModel(columns, 0);
        tableMahasiswa = new JTable(tableModel);
        JScrollPane scrollPane = new JScrollPane(tableMahasiswa);
        
        // Add components to frame
        add(formPanel, BorderLayout.NORTH);
        add(buttonPanel, BorderLayout.CENTER);
        add(scrollPane, BorderLayout.SOUTH);
        
        pack();
        setLocationRelativeTo(null);
    }
    
    public void addSaveListener(ActionListener listener) {
        btnSave.addActionListener(listener);
    }
    
    public void addDeleteListener(ActionListener listener) {
        btnDelete.addActionListener(listener);
    }
    
    public void addClearListener(ActionListener listener) {
        btnClear.addActionListener(listener);
    }

    public void addNilaiListener(ActionListener listener) {
        btnNilai.addActionListener(listener);
    }
    
    public Mahasiswa getMahasiswa() {
        Mahasiswa m = new Mahasiswa();
        m.setId(txtId.getText().isEmpty() ? 0 : Integer.parseInt(txtId.getText()));
        m.setNim(txtNim.getText());
        m.setNama(txtNama.getText());
        m.setJurusan(txtJurusan.getText());
        m.setAngkatan(txtAngkatan.getText());
        return m;
    }
    
    public void setMahasiswas(List<Mahasiswa> mahasiswas) {
        tableModel.setRowCount(0);
        for (Mahasiswa m : mahasiswas) {
            tableModel.addRow(new Object[]{
                m.getId(),
                m.getNim(),
                m.getNama(),
                m.getJurusan(),
                m.getAngkatan()
            });
        }
    }
    
    public void clearForm() {
        txtId.setText("");
        txtNim.setText("");
        txtNama.setText("");
        txtJurusan.setText("");
        txtAngkatan.setText("");
    }
    
    public int getSelectedMahasiswaId() {
        int row = tableMahasiswa.getSelectedRow();
        return row >= 0 ? (Integer) tableModel.getValueAt(row, 0) : 0;
    }
}
```

# FormNilai.java
``` Java
package view;

import java.awt.*;
import java.awt.event.ActionListener;
import java.util.List;
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import model.Mahasiswa;
import model.Nilai;

public class FormNilai extends JDialog {
    private JTextField txtId;
    private JTextField txtMataKuliah;
    private JTextField txtSemester;
    private JTextField txtNilai;
    private JButton btnSave;
    private JButton btnDelete;
    private JButton btnClear;
    private JTable tableNilai;
    private DefaultTableModel tableModel;
    private int mahasiswaId;
    private JLabel lblMahasiswa;
    
    public FormNilai(Frame parent) {
        super(parent, "Data Nilai Mahasiswa", true);
        initComponents();
    }
    
    private void initComponents() {
        setLayout(new BorderLayout());
        
        // Form Panel
        JPanel formPanel = new JPanel(new GridLayout(4, 2, 5, 5));
        formPanel.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));
        
        txtId = new JTextField();
        txtId.setEditable(false);
        txtMataKuliah = new JTextField();
        txtSemester = new JTextField();
        txtNilai = new JTextField();
        
        formPanel.add(new JLabel("ID:"));
        formPanel.add(txtId);
        formPanel.add(new JLabel("Mata Kuliah:"));
        formPanel.add(txtMataKuliah);
        formPanel.add(new JLabel("Semester:"));
        formPanel.add(txtSemester);
        formPanel.add(new JLabel("Nilai:"));
        formPanel.add(txtNilai);
        
        // Button Panel
        JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.CENTER));
        btnSave = new JButton("Save");
        btnDelete = new JButton("Delete");
        btnClear = new JButton("Clear");
        
        buttonPanel.add(btnSave);
        buttonPanel.add(btnDelete);
        buttonPanel.add(btnClear);
        
        // Table
        String[] columns = {"ID", "Mata Kuliah", "Semester", "Nilai"};
        tableModel = new DefaultTableModel(columns, 0) {
            @Override
            public boolean isCellEditable(int row, int column) {
                return false;
            }
        };
        tableNilai = new JTable(tableModel);
        JScrollPane scrollPane = new JScrollPane(tableNilai);
        
        // Add table selection listener
        tableNilai.getSelectionModel().addListSelectionListener(e -> {
            if (!e.getValueIsAdjusting()) {
                int selectedRow = tableNilai.getSelectedRow();
                if (selectedRow >= 0) {
                    txtId.setText(tableModel.getValueAt(selectedRow, 0).toString());
                    txtMataKuliah.setText(tableModel.getValueAt(selectedRow, 1).toString());
                    txtSemester.setText(tableModel.getValueAt(selectedRow, 2).toString());
                    txtNilai.setText(tableModel.getValueAt(selectedRow, 3).toString());
                }
            }
        });
        
        // Main content panel to hold form and buttons
        JPanel contentPanel = new JPanel(new BorderLayout());
        contentPanel.add(formPanel, BorderLayout.CENTER);
        contentPanel.add(buttonPanel, BorderLayout.SOUTH);
        
        // Add all components to dialog
        add(contentPanel, BorderLayout.NORTH);
        add(scrollPane, BorderLayout.CENTER);
        
        setSize(500, 400);
        setLocationRelativeTo(null);
    }
    
    public void setMahasiswaInfo(Mahasiswa mahasiswa) {
        this.mahasiswaId = mahasiswa.getId();
        setTitle("Data Nilai Mahasiswa: " + mahasiswa.getNama() + " (" + mahasiswa.getNim() + ")");
    }
    
    public Nilai getNilai() throws Exception {
        // Validasi input
        if (txtMataKuliah.getText().trim().isEmpty()) {
            throw new Exception("Mata kuliah tidak boleh kosong");
        }
        
        if (txtSemester.getText().trim().isEmpty()) {
            throw new Exception("Semester tidak boleh kosong");
        }
        
        String nilaiStr = txtNilai.getText().trim();
        if (nilaiStr.isEmpty()) {
            throw new Exception("Nilai tidak boleh kosong");
        }
        
        int nilaiInt;
        try {
            nilaiInt = Integer.parseInt(nilaiStr);
            if (nilaiInt < 0 || nilaiInt > 100) {
                throw new Exception("Nilai harus antara 0-100");
            }
        } catch (NumberFormatException e) {
            throw new Exception("Nilai harus berupa angka");
        }
        
        Nilai n = new Nilai();
        n.setId(txtId.getText().isEmpty() ? 0 : Integer.parseInt(txtId.getText()));
        n.setMahasiswaId(mahasiswaId);
        n.setMataKuliah(txtMataKuliah.getText().trim());
        n.setSemester(txtSemester.getText().trim());
        n.setNilai(nilaiInt);
        
        return n;
    }
    
    public void setNilaiList(List<Nilai> nilaiList) {
        tableModel.setRowCount(0);
        for (Nilai n : nilaiList) {
            tableModel.addRow(new Object[]{
                n.getId(),
                n.getMataKuliah(),
                n.getSemester(),
                n.getNilai()
            });
        }
    }
    
    public void clearForm() {
        txtId.setText("");
        txtMataKuliah.setText("");
        txtSemester.setText("");
        txtNilai.setText("");
        tableNilai.clearSelection();
    }
    
    public int getSelectedNilaiId() {
        int row = tableNilai.getSelectedRow();
        return row >= 0 ? (Integer) tableModel.getValueAt(row, 0) : 0;
    }
    
    public void addSaveListener(ActionListener listener) {
        btnSave.addActionListener(listener);
    }
    
    public void addDeleteListener(ActionListener listener) {
        btnDelete.addActionListener(listener);
    }
    
    public void addClearListener(ActionListener listener) {
        btnClear.addActionListener(listener);
    }
}
```
### **Penjelasan Kode: FormMahasiswa.java dan FormNilai.java**

### **FormMahasiswa.java**
Form ini digunakan untuk mengelola data mahasiswa. Formulir ini memungkinkan pengguna untuk melakukan operasi CRUD (Create, Read, Update, Delete) pada data mahasiswa.

#### Komponen Utama:
- **JTextField** (`txtId`, `txtNim`, `txtNama`, `txtJurusan`, `txtAngkatan`): Digunakan untuk menginput detail mahasiswa seperti ID, NIM (nomor induk mahasiswa), nama, jurusan, dan angkatan.
- **JButton** (`btnSave`, `btnDelete`, `btnClear`, `btnNilai`): Tombol untuk melakukan berbagai aksi, seperti menyimpan data mahasiswa, menghapus mahasiswa, membersihkan form, atau melihat nilai mahasiswa.
- **JTable** (`tableMahasiswa`): Digunakan untuk menampilkan daftar mahasiswa.
- **DefaultTableModel**: Digunakan untuk mengelola data dan struktur tabel.

#### Fungsi Utama:
- **addSaveListener**, **addDeleteListener**, **addClearListener**, **addNilaiListener**: Metode-metode ini digunakan untuk menambahkan listener yang menangani aksi pengguna seperti menyimpan mahasiswa baru, menghapus mahasiswa yang ada, membersihkan form, atau melihat nilai mahasiswa.
- **getMahasiswa()**: Mengambil data mahasiswa dari field teks dan mengembalikannya dalam bentuk objek `Mahasiswa`. Jika ID kosong, akan diset ke nilai 0.
- **setMahasiswas()**: Mengisi tabel mahasiswa dengan data mahasiswa yang diberikan. Ini akan mengupdate tabel dengan daftar mahasiswa yang diberikan.

### **FormNilai.java**
Form ini digunakan untuk mengelola nilai mahasiswa. Formulir ini memungkinkan pengguna untuk menambah, mengubah, dan menghapus nilai mahasiswa.

#### Komponen Utama:
- **JTextField** (`txtId`, `txtMataKuliah`, `txtSemester`, `txtNilai`): Digunakan untuk memasukkan data terkait nilai mahasiswa, seperti ID nilai, mata kuliah, semester, dan nilai yang diperoleh.
- **JButton** (`btnSave`, `btnDelete`, `btnClear`): Tombol untuk menyimpan, menghapus, atau membersihkan form.
- **JTable** (`tableNilai`): Tabel untuk menampilkan daftar nilai mahasiswa.
- **DefaultTableModel**: Digunakan untuk mengelola data dan struktur tabel nilai mahasiswa.

#### Fungsi Utama:
- **setMahasiswaInfo()**: Menampilkan informasi mahasiswa di header dialog berdasarkan ID mahasiswa yang dipilih. Ini mengubah judul dialog dengan nama mahasiswa yang bersangkutan.
- **getNilai()**: Mengambil data nilai dari field teks dan mengembalikannya dalam bentuk objek `Nilai`. Sebelum itu, dilakukan validasi input, seperti memastikan nilai di antara 0 hingga 100.
- **setNilaiList()**: Mengisi tabel dengan daftar nilai mahasiswa.
- **clearForm()**: Membersihkan semua field input dan menghapus seleksi pada tabel.
- **addSaveListener**, **addDeleteListener**, **addClearListener**: Metode-metode ini menambahkan listener untuk menangani aksi pengguna seperti menyimpan nilai, menghapus nilai, atau membersihkan form.

# Main.java
``` Java
import classes.Database;
import controller.MahasiswaController;
import java.sql.Connection;
import model.MahasiswaModel;
import view.FormMahasiswa;

public class Main {
    public static void main(String[] args) {
        try {
            Connection connection = Database.getConnection();
            MahasiswaModel model = new MahasiswaModel(connection);
            FormMahasiswa view = new FormMahasiswa();
            MahasiswaController controller = new MahasiswaController(model, view, connection);
            
            view.setVisible(true);
        } catch (Exception e) {
            e.printStackTrace();
            System.exit(1);
        }
    }
}
```

# TestConnection.java
``` Java
// TestConnection.java
import classes.Database;
import java.sql.Connection;

public class TestConnection {
    public static void main(String[] args) {
        try {
            Connection conn = Database.getConnection();
            System.out.println("Koneksi berhasil!");
            conn.close();
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```
