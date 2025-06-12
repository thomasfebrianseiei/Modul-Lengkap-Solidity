# Bagian 1: Dasar-Dasar Solidity untuk Pemula
## Belajar Smart Contract dari Nol dengan Penjelasan Sederhana

---

## Apa itu Solidity?

**Solidity** adalah bahasa pemrograman khusus untuk membuat **Smart Contract** di blockchain Ethereum dan blockchain sejenis (seperti BSC, Polygon).

### Analogi Sederhana:
- **Smart Contract** = Kontrak pintar yang berjalan otomatis
- **Blockchain** = Buku catatan raksasa yang tidak bisa diubah
- **Solidity** = Bahasa untuk menulis aturan dalam kontrak pintar

### Contoh Kehidupan Nyata:
Bayangkan Anda punya **mesin penjual minuman otomatis**:
- Anda masukkan uang → mesin memberikan minuman
- Tidak ada yang bisa curang karena mesinnya otomatis
- Smart contract bekerja persis seperti ini, tapi di dunia digital!

---

## Setup Environment (Persiapan Alat)

### 1. Install Node.js
```bash
# Download dari https://nodejs.org
# Atau pakai installer Windows/Mac
```

### 2. Buat Project Baru
```bash
# Buat folder project
mkdir belajar-solidity
cd belajar-solidity

# Inisialisasi project
npm init -y

# Install Hardhat (alat untuk coding Solidity)
npm install --save-dev hardhat

# Setup Hardhat
npx hardhat
# Pilih "Create a JavaScript project"
```

### 3. Install Library Tambahan
```bash
# OpenZeppelin - library kontrak yang aman
npm install @openzeppelin/contracts

# Dotenv - untuk environment variables
npm install dotenv
```

---

## Struktur File Project

```
belajar-solidity/
├── contracts/          # Tempat file .sol (smart contract)
├── scripts/            # Script untuk deploy
├── test/              # File testing
├── hardhat.config.js  # Konfigurasi
└── .env              # Environment variables (rahasia)
```

---

## Hello World - Smart Contract Pertama

Mari buat smart contract pertama yang sederhana:

```solidity
// SPDX-License-Identifier: MIT
// Ini seperti "copyright" - wajib ada di setiap file
pragma solidity ^0.8.19;
// Ini versi Solidity yang kita pakai

// "contract" seperti "class" dalam bahasa pemrograman lain
contract HelloWorld {
    // Variabel untuk menyimpan pesan
    // "string" = tipe data teks
    // "public" = bisa dibaca siapa saja
    string public message;
    
    // Constructor = function yang jalan sekali saat contract dibuat
    constructor() {
        message = "Hello, Blockchain World!";
    }
    
    // Function untuk mengubah pesan
    // "public" = bisa dipanggil dari luar
    function setMessage(string memory newMessage) public {
        message = newMessage;
    }
    
    // Function untuk membaca pesan
    // "view" = hanya membaca, tidak mengubah data
    // "returns" = mengembalikan nilai
    function getMessage() public view returns (string memory) {
        return message;
    }
}
```

**Penjelasan Baris per Baris:**
1. **SPDX-License-Identifier**: Seperti label copyright
2. **pragma solidity**: Versi Solidity yang digunakan
3. **contract HelloWorld**: Nama kontrak (seperti nama class)
4. **string public message**: Variabel untuk menyimpan teks
5. **constructor()**: Function yang jalan saat kontrak dibuat
6. **setMessage()**: Function untuk mengubah pesan
7. **getMessage()**: Function untuk membaca pesan

---

## Tipe Data dalam Solidity

### 1. Tipe Data Dasar
```solidity
contract TipeData {
    // Boolean - true atau false
    bool public aktif = true;
    bool public selesai = false;
    
    // Integer - angka bulat
    uint public umur = 25;           // angka positif
    int public suhu = -10;           // bisa negatif
    uint256 public saldo = 1000;     // angka besar
    
    // Address - alamat wallet Ethereum
    address public owner;            // alamat pemilik
    address public user;             // alamat user
    
    // String - teks
    string public nama = "Budi";
    string public kota = "Jakarta";
    
    // Bytes - data mentah
    bytes32 public hash;             // untuk menyimpan hash
    
    constructor() {
        owner = msg.sender;  // msg.sender = yang deploy contract
    }
}
```

### 2. Array - Kumpulan Data
```solidity
contract ArrayContoh {
    // Array dinamis - ukuran bisa berubah
    uint[] public angka;
    string[] public namaSiswa;
    
    // Array tetap - ukuran tidak berubah
    uint[5] public nilai;  // array 5 elemen
    
    // Function untuk menambah data ke array
    function tambahAngka(uint angkaBaru) public {
        angka.push(angkaBaru);  // menambah di akhir array
    }
    
    // Function untuk membaca array
    function bacaAngka(uint index) public view returns (uint) {
        return angka[index];  // baca data di posisi 'index'
    }
    
    // Function untuk menghitung jumlah elemen
    function jumlahAngka() public view returns (uint) {
        return angka.length;
    }
}
```

### 3. Mapping - Seperti Kamus
```solidity
contract MappingContoh {
    // Mapping = seperti kamus atau dictionary
    // mapping(kunci => nilai)
    mapping(address => uint) public saldo;          // alamat => saldo
    mapping(string => bool) public ijin;            // nama => punya ijin
    mapping(uint => string) public idKeNama;        // id => nama
    
    // Nested mapping - mapping di dalam mapping
    mapping(address => mapping(address => uint)) public allowance;
    
    // Function untuk set saldo
    function setSaldo(address user, uint jumlah) public {
        saldo[user] = jumlah;
    }
    
    // Function untuk memberikan ijin
    function beriIjin(string memory nama) public {
        ijin[nama] = true;
    }
    
    // Function untuk cek ijin
    function cekIjin(string memory nama) public view returns (bool) {
        return ijin[nama];
    }
}
```

---

## Struct - Tipe Data Custom

**Struct** seperti template untuk membuat objek dengan beberapa properti:

```solidity
contract StructContoh {
    // Definisi struct Mahasiswa
    struct Mahasiswa {
        string nama;
        uint umur;
        string jurusan;
        bool aktif;
        uint ipk;  // IPK dikali 100 (contoh: 350 = IPK 3.5)
    }
    
    // Array untuk menyimpan banyak mahasiswa
    Mahasiswa[] public daftarMahasiswa;
    
    // Mapping dari alamat ke data mahasiswa
    mapping(address => Mahasiswa) public dataMahasiswa;
    
    // Function untuk menambah mahasiswa baru
    function tambahMahasiswa(
        string memory _nama,
        uint _umur,
        string memory _jurusan
    ) public {
        // Cara 1: Membuat struct langsung
        daftarMahasiswa.push(Mahasiswa({
            nama: _nama,
            umur: _umur,
            jurusan: _jurusan,
            aktif: true,
            ipk: 0
        }));
        
        // Cara 2: Simpan ke mapping
        dataMahasiswa[msg.sender] = Mahasiswa({
            nama: _nama,
            umur: _umur,
            jurusan: _jurusan,
            aktif: true,
            ipk: 0
        });
    }
    
    // Function untuk update IPK
    function updateIPK(uint _ipk) public {
        // Update IPK untuk mahasiswa yang memanggil function ini
        dataMahasiswa[msg.sender].ipk = _ipk;
    }
    
    // Function untuk mendapatkan info mahasiswa
    function getMahasiswa(address _alamat) public view returns (
        string memory nama,
        uint umur,
        string memory jurusan,
        bool aktif,
        uint ipk
    ) {
        Mahasiswa memory mhs = dataMahasiswa[_alamat];
        return (mhs.nama, mhs.umur, mhs.jurusan, mhs.aktif, mhs.ipk);
    }
    
    // Function untuk menghitung jumlah mahasiswa
    function jumlahMahasiswa() public view returns (uint) {
        return daftarMahasiswa.length;
    }
}
```

---

## Functions dalam Solidity

### 1. Jenis-jenis Function

```solidity
contract FunctionContoh {
    uint public angka = 10;
    string public teks = "Hello";
    address public owner;
    
    constructor() {
        owner = msg.sender;
    }
    
    // 1. Function biasa - bisa mengubah data
    function ubahAngka(uint angkaBaru) public {
        angka = angkaBaru;  // mengubah state
    }
    
    // 2. View function - hanya membaca, tidak mengubah
    function bacaAngka() public view returns (uint) {
        return angka;  // hanya membaca
    }
    
    // 3. Pure function - tidak baca dan tidak ubah state
    function hitungTambah(uint a, uint b) public pure returns (uint) {
        return a + b;  // hanya kalkulasi
    }
    
    // 4. Payable function - bisa menerima Ether
    function deposit() public payable {
        // msg.value = jumlah Ether yang dikirim
        // Function ini bisa menerima Ether
    }
    
    // 5. Function dengan multiple parameter
    function dataLengkap(
        string memory _nama,
        uint _umur,
        bool _aktif
    ) public pure returns (string memory, uint, bool) {
        return (_nama, _umur, _aktif);
    }
}
```

### 2. Visibility (Siapa yang Bisa Akses)

```solidity
contract VisibilityContoh {
    uint private rahasia = 100;        // hanya contract ini
    uint internal bersama = 200;       // contract ini + turunannya
    uint public terbuka = 300;         // semua orang
    
    // Private - hanya bisa dipanggil dari dalam contract ini
    function functionRahasia() private pure returns (string memory) {
        return "Ini rahasia";
    }
    
    // Internal - bisa dipanggil dari contract ini dan turunannya
    function functionBersama() internal pure returns (string memory) {
        return "Ini bersama";
    }
    
    // Public - bisa dipanggil dari mana saja
    function functionTerbuka() public pure returns (string memory) {
        return "Ini terbuka";
    }
    
    // External - hanya bisa dipanggil dari luar contract
    function functionEksternal() external pure returns (string memory) {
        return "Ini dari luar";
    }
    
    // Function yang memanggil function lain
    function panggilSemuaFunction() public view returns (string memory) {
        // Bisa panggil private dan internal dari dalam
        functionRahasia();
        functionBersama();
        
        // Untuk panggil external dari dalam, pakai this
        this.functionEksternal();
        
        return "Semua function dipanggil";
    }
}
```

---

## Control Flow (If, For, While)

### 1. If-Else Statement

```solidity
contract ControlFlowContoh {
    uint public saldo = 1000;
    
    // Function dengan if-else
    function cekSaldo(uint jumlahTarik) public view returns (string memory) {
        if (jumlahTarik > saldo) {
            return "Saldo tidak cukup";
        } else if (jumlahTarik == saldo) {
            return "Saldo akan habis";
        } else {
            return "Saldo cukup";
        }
    }
    
    // Function dengan multiple kondisi
    function cekUmur(uint umur) public pure returns (string memory) {
        if (umur >= 17 && umur < 60) {
            return "Boleh buat KTP";
        } else if (umur >= 60) {
            return "Senior citizen";
        } else {
            return "Masih anak-anak";
        }
    }
}
```

### 2. For Loop dan While Loop

```solidity
contract LoopContoh {
    uint[] public angka;
    
    // For loop - perulangan dengan jumlah tertentu
    function isiAngka() public {
        // Hapus array dulu
        delete angka;
        
        // For loop dari 1 sampai 10
        for (uint i = 1; i <= 10; i++) {
            angka.push(i);
        }
    }
    
    // Function untuk menjumlahkan semua angka dalam array
    function jumlahkanArray() public view returns (uint) {
        uint total = 0;
        
        // Loop melalui semua elemen array
        for (uint i = 0; i < angka.length; i++) {
            total += angka[i];
        }
        
        return total;
    }
    
    // While loop - perulangan dengan kondisi
    function hitungMundur(uint mulai) public pure returns (uint[] memory) {
        uint[] memory hasil = new uint[](mulai);
        uint index = 0;
        
        while (mulai > 0) {
            hasil[index] = mulai;
            mulai--;
            index++;
        }
        
        return hasil;
    }
    
    // Function untuk mencari angka dalam array
    function cariAngka(uint target) public view returns (bool, uint) {
        for (uint i = 0; i < angka.length; i++) {
            if (angka[i] == target) {
                return (true, i);  // found, index
            }
        }
        return (false, 0);  // not found
    }
}
```

---

## Events - Sistem Logging

**Events** seperti catatan log yang bisa dilihat di blockchain:

```solidity
contract EventContoh {
    // Deklarasi event
    event Transfer(
        address indexed dari,      // indexed bisa di-filter
        address indexed ke,
        uint jumlah,
        string keterangan
    );
    
    event StatusChanged(
        address indexed user,
        string status,
        uint timestamp
    );
    
    mapping(address => uint) public saldo;
    
    // Function yang emit event
    function transfer(address ke, uint jumlah, string memory keterangan) public {
        require(saldo[msg.sender] >= jumlah, "Saldo tidak cukup");
        
        // Update saldo
        saldo[msg.sender] -= jumlah;
        saldo[ke] += jumlah;
        
        // Emit event - seperti menulis log
        emit Transfer(msg.sender, ke, jumlah, keterangan);
    }
    
    function deposit() public payable {
        saldo[msg.sender] += msg.value;
        
        emit StatusChanged(
            msg.sender,
            "Deposit berhasil",
            block.timestamp  // waktu saat ini
        );
    }
    
    function ubahStatus(string memory statusBaru) public {
        emit StatusChanged(msg.sender, statusBaru, block.timestamp);
    }
}
```

**Kegunaan Events:**
- **Monitoring**: Lihat aktivitas di blockchain
- **Frontend**: Update UI ketika ada perubahan
- **Analytics**: Analisis penggunaan contract
- **Debugging**: Lacak masalah dalam contract

---

## Modifiers - Aturan Akses

**Modifiers** seperti security guard yang cek syarat sebelum function berjalan:

```solidity
contract ModifierContoh {
    address public owner;
    bool public paused = false;
    mapping(address => bool) public whitelist;
    
    constructor() {
        owner = msg.sender;  // yang deploy jadi owner
    }
    
    // Modifier hanya owner
    modifier onlyOwner() {
        require(msg.sender == owner, "Hanya owner yang bisa akses");
        _;  // titik ini = lanjutkan function
    }
    
    // Modifier cek pause
    modifier notPaused() {
        require(!paused, "Contract sedang di-pause");
        _;
    }
    
    // Modifier cek whitelist
    modifier onlyWhitelisted() {
        require(whitelist[msg.sender], "Anda tidak ada di whitelist");
        _;
    }
    
    // Modifier dengan parameter
    modifier minAmount(uint amount) {
        require(amount >= 1000, "Jumlah minimal 1000");
        _;
    }
    
    // Function dengan modifier
    function pauseContract() public onlyOwner {
        paused = true;
    }
    
    function unpauseContract() public onlyOwner {
        paused = false;
    }
    
    function addToWhitelist(address user) public onlyOwner {
        whitelist[user] = true;
    }
    
    // Function dengan multiple modifier
    function withdraw(uint amount) 
        public 
        notPaused 
        onlyWhitelisted 
        minAmount(amount) 
    {
        // Logic withdrawal di sini
        // Semua modifier di atas akan dicek dulu
    }
    
    // Function untuk ganti owner
    function transferOwnership(address newOwner) public onlyOwner {
        require(newOwner != address(0), "Owner baru tidak boleh address 0");
        owner = newOwner;
    }
}
```

---

## Error Handling

### 1. Require, Revert, Assert

```solidity
contract ErrorHandlingContoh {
    mapping(address => uint) public saldo;
    address public owner;
    
    constructor() {
        owner = msg.sender;
    }
    
    // REQUIRE - cek kondisi, kalau false akan revert
    function withdraw(uint jumlah) public {
        require(jumlah > 0, "Jumlah harus lebih dari 0");
        require(saldo[msg.sender] >= jumlah, "Saldo tidak cukup");
        
        saldo[msg.sender] -= jumlah;
        payable(msg.sender).transfer(jumlah);
    }
    
    // REVERT - langsung batalkan dengan pesan error
    function checkAge(uint age) public pure returns (string memory) {
        if (age < 18) {
            revert("Umur harus 18 tahun ke atas");
        }
        return "Umur valid";
    }
    
    // ASSERT - untuk kondisi yang seharusnya tidak pernah false
    function divide(uint a, uint b) public pure returns (uint) {
        assert(b != 0);  // ini seharusnya tidak pernah 0
        return a / b;
    }
    
    // Custom error (Solidity 0.8.4+)
    error InsufficientBalance(uint available, uint required);
    error Unauthorized(address caller);
    
    function transferWithCustomError(address to, uint amount) public {
        if (saldo[msg.sender] < amount) {
            revert InsufficientBalance({
                available: saldo[msg.sender],
                required: amount
            });
        }
        
        if (to == address(0)) {
            revert("Cannot transfer to zero address");
        }
        
        saldo[msg.sender] -= amount;
        saldo[to] += amount;
    }
}
```

**Perbedaan:**
- **require()**: Cek kondisi, gas dikembalikan jika gagal
- **revert()**: Langsung batalkan, gas dikembalikan
- **assert()**: Untuk kondisi yang tidak boleh false, gas tidak dikembalikan

---

## Contract Sederhana: Bank Digital

Mari buat contract bank digital sederhana yang menggabungkan semua konsep:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract BankDigital {
    // Data structures
    struct Nasabah {
        string nama;
        uint saldo;
        bool aktif;
        uint timestamp;
    }
    
    // State variables
    address public owner;
    mapping(address => Nasabah) public nasabah;
    mapping(address => bool) public terdaftar;
    address[] public daftarNasabah;
    uint public totalSaldo;
    bool public bankBuka = true;
    
    // Events
    event NasabahTerdaftar(address indexed alamat, string nama);
    event Deposit(address indexed nasabah, uint jumlah, uint saldoBaru);
    event Withdraw(address indexed nasabah, uint jumlah, uint saldoBaru);
    event Transfer(address indexed dari, address indexed ke, uint jumlah);
    
    // Modifiers
    modifier onlyOwner() {
        require(msg.sender == owner, "Hanya pemilik bank");
        _;
    }
    
    modifier bankTerbuka() {
        require(bankBuka, "Bank sedang tutup");
        _;
    }
    
    modifier nasabahTerdaftar() {
        require(terdaftar[msg.sender], "Anda belum terdaftar");
        _;
    }
    
    modifier nasabahAktif() {
        require(nasabah[msg.sender].aktif, "Akun Anda tidak aktif");
        _;
    }
    
    constructor() {
        owner = msg.sender;
    }
    
    // Function untuk daftar sebagai nasabah
    function daftar(string memory _nama) public bankTerbuka {
        require(!terdaftar[msg.sender], "Anda sudah terdaftar");
        require(bytes(_nama).length > 0, "Nama tidak boleh kosong");
        
        nasabah[msg.sender] = Nasabah({
            nama: _nama,
            saldo: 0,
            aktif: true,
            timestamp: block.timestamp
        });
        
        terdaftar[msg.sender] = true;
        daftarNasabah.push(msg.sender);
        
        emit NasabahTerdaftar(msg.sender, _nama);
    }
    
    // Function untuk deposit (setoran)
    function deposit() public payable bankTerbuka nasabahTerdaftar nasabahAktif {
        require(msg.value > 0, "Jumlah deposit harus lebih dari 0");
        
        nasabah[msg.sender].saldo += msg.value;
        totalSaldo += msg.value;
        
        emit Deposit(msg.sender, msg.value, nasabah[msg.sender].saldo);
    }
    
    // Function untuk withdraw (tarik tunai)
    function withdraw(uint _jumlah) public bankTerbuka nasabahTerdaftar nasabahAktif {
        require(_jumlah > 0, "Jumlah withdraw harus lebih dari 0");
        require(nasabah[msg.sender].saldo >= _jumlah, "Saldo tidak cukup");
        
        nasabah[msg.sender].saldo -= _jumlah;
        totalSaldo -= _jumlah;
        
        payable(msg.sender).transfer(_jumlah);
        
        emit Withdraw(msg.sender, _jumlah, nasabah[msg.sender].saldo);
    }
    
    // Function untuk transfer antar nasabah
    function transfer(address _ke, uint _jumlah) public bankTerbuka nasabahTerdaftar nasabahAktif {
        require(_ke != msg.sender, "Tidak bisa transfer ke diri sendiri");
        require(terdaftar[_ke], "Penerima belum terdaftar");
        require(nasabah[_ke].aktif, "Akun penerima tidak aktif");
        require(_jumlah > 0, "Jumlah transfer harus lebih dari 0");
        require(nasabah[msg.sender].saldo >= _jumlah, "Saldo tidak cukup");
        
        nasabah[msg.sender].saldo -= _jumlah;
        nasabah[_ke].saldo += _jumlah;
        
        emit Transfer(msg.sender, _ke, _jumlah);
    }
    
    // Function untuk cek saldo
    function cekSaldo() public view returns (uint) {
        return nasabah[msg.sender].saldo;
    }
    
    // Function untuk cek info nasabah
    function infoNasabah(address _alamat) public view returns (
        string memory nama,
        uint saldo,
        bool aktif,
        uint tanggalDaftar
    ) {
        require(terdaftar[_alamat], "Nasabah tidak terdaftar");
        Nasabah memory n = nasabah[_alamat];
        return (n.nama, n.saldo, n.aktif, n.timestamp);
    }
    
    // Function khusus owner
    function tutupBank() public onlyOwner {
        bankBuka = false;
    }
    
    function bukaBank() public onlyOwner {
        bankBuka = true;
    }
    
    function nonaktifkanNasabah(address _alamat) public onlyOwner {
        require(terdaftar[_alamat], "Nasabah tidak terdaftar");
        nasabah[_alamat].aktif = false;
    }
    
    function aktifkanNasabah(address _alamat) public onlyOwner {
        require(terdaftar[_alamat], "Nasabah tidak terdaftar");
        nasabah[_alamat].aktif = true;
    }
    
    // Function untuk mendapatkan statistik bank
    function statistikBank() public view returns (
        uint jumlahNasabah,
        uint totalSaldoBank,
        bool statusBank
    ) {
        return (daftarNasabah.length, totalSaldo, bankBuka);
    }
    
    // Emergency function untuk owner
    function emergencyWithdraw() public onlyOwner {
        payable(owner).transfer(address(this).balance);
    }
}
```

---

## Testing Contract

Buat file test untuk mengetes contract:

```javascript
// test/BankDigital.test.js
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("Bank Digital", function () {
    let BankDigital, bank, owner, nasabah1, nasabah2;
    
    beforeEach(async function () {
        BankDigital = await ethers.getContractFactory("BankDigital");
        [owner, nasabah1, nasabah2] = await ethers.getSigners();
        
        bank = await BankDigital.deploy();
        await bank.deployed();
    });
    
    describe("Pendaftaran Nasabah", function () {
        it("Nasabah bisa daftar dengan nama", async function () {
            await bank.connect(nasabah1).daftar("Budi");
            
            const info = await bank.infoNasabah(nasabah1.address);
            expect(info.nama).to.equal("Budi");
            expect(info.saldo).to.equal(0);
            expect(info.aktif).to.equal(true);
        });
        
        it("Tidak bisa daftar dengan nama kosong", async function () {
            await expect(
                bank.connect(nasabah1).daftar("")
            ).to.be.revertedWith("Nama tidak boleh kosong");
        });
    });
    
    describe("Deposit dan Withdraw", function () {
        beforeEach(async function () {
            await bank.connect(nasabah1).daftar("Budi");
        });
        
        it("Bisa deposit Ether", async function () {
            const depositAmount = ethers.utils.parseEther("1");
            
            await bank.connect(nasabah1).deposit({ value: depositAmount });
            
            const saldo = await bank.connect(nasabah1).cekSaldo();
            expect(saldo).to.equal(depositAmount);
        });
        
        it("Bisa withdraw sesuai saldo", async function () {
            const depositAmount = ethers.utils.parseEther("1");
            const withdrawAmount = ethers.utils.parseEther("0.5");
            
            await bank.connect(nasabah1).deposit({ value: depositAmount });
            await bank.connect(nasabah1).withdraw(withdrawAmount);
            
            const saldo = await bank.connect(nasabah1).cekSaldo();
            expect(saldo).to.equal(depositAmount.sub(withdrawAmount));
        });
        
        it("Tidak bisa withdraw lebih dari saldo", async function () {
            const withdrawAmount = ethers.utils.parseEther("1");
            
            await expect(
                bank.connect(nasabah1).withdraw(withdrawAmount)
            ).to.be.revertedWith("Saldo tidak cukup");
        });
    });
    
    describe("Transfer", function () {
        beforeEach(async function () {
            await bank.connect(nasabah1).daftar("Budi");
            await bank.connect(nasabah2).daftar("Siti");
            
            const depositAmount = ethers.utils.parseEther("2");
            await bank.connect(nasabah1).deposit({ value: depositAmount });
        });
        
        it("Bisa transfer ke nasabah lain", async function () {
            const transferAmount = ethers.utils.parseEther("1");
            
            await bank.connect(nasabah1).transfer(nasabah2.address, transferAmount);
            
            const saldoNasabah1 = await bank.connect(nasabah1).cekSaldo();
            const saldoNasabah2 = await bank.connect(nasabah2).cekSaldo();
            
            expect(saldoNasabah1).to.equal(ethers.utils.parseEther("1"));
            expect(saldoNasabah2).to.equal(ethers.utils.parseEther("1"));
        });
    });
});
```

---

## Deploy Contract

Buat script untuk deploy contract:

```javascript
// scripts/deploy.js
async function main() {
    console.log("🚀 Mulai deploy Bank Digital...");
    
    // Ambil akun yang akan deploy
    const [deployer] = await ethers.getSigners();
    console.log("📝 Deploy dengan akun:", deployer.address);
    
    // Cek saldo akun
    const balance = await deployer.getBalance();
    console.log("💰 Saldo akun:", ethers.utils.formatEther(balance), "ETH");
    
    // Deploy contract
    const BankDigital = await ethers.getContractFactory("BankDigital");
    console.log("🔨 Sedang deploy contract...");
    
    const bank = await BankDigital.deploy();
    await bank.deployed();
    
    console.log("✅ Bank Digital berhasil di-deploy!");
    console.log("📍 Alamat contract:", bank.address);
    console.log("👑 Owner contract:", await bank.owner());
    
    // Test fungsi dasar
    console.log("\n🧪 Testing fungsi dasar...");
    const stats = await bank.statistikBank();
    console.log("📊 Statistik bank:");
    console.log("   - Jumlah nasabah:", stats.jumlahNasabah.toString());
    console.log("   - Total saldo:", ethers.utils.formatEther(stats.totalSaldoBank), "ETH");
    console.log("   - Status bank:", stats.statusBank ? "Buka" : "Tutup");
    
    // Simpan info deployment
    const deploymentInfo = {
        network: network.name,
        contractAddress: bank.address,
        owner: deployer.address,
        deployTime: new Date().toISOString()
    };
    
    console.log("\n💾 Info deployment:", deploymentInfo);
}

main()
    .then(() => {
        console.log("🎉 Deployment selesai!");
        process.exit(0);
    })
    .catch((error) => {
        console.error("❌ Error saat deploy:", error);
        process.exit(1);
    });
```

## Cara Menjalankan

### 1. Compile Contract
```bash
npx hardhat compile
```

### 2. Run Test
```bash
npx hardhat test
```

### 3. Deploy ke Local Network
```bash
# Terminal 1 - Jalankan local blockchain
npx hardhat node

# Terminal 2 - Deploy
npx hardhat run scripts/deploy.js --network localhost
```

### 4. Deploy ke Testnet
```bash
# Deploy ke BSC Testnet
npx hardhat run scripts/deploy.js --network bscTestnet

# Deploy ke Sepolia Testnet
npx hardhat run scripts/deploy.js --network sepolia
```

---

## Interaksi dengan Contract

Setelah deploy, kita bisa berinteraksi dengan contract:

```javascript
// scripts/interact.js
async function main() {
    // Alamat contract yang sudah di-deploy
    const contractAddress = "0x..."; // ganti dengan alamat real
    
    // Ambil contract instance
    const BankDigital = await ethers.getContractFactory("BankDigital");
    const bank = BankDigital.attach(contractAddress);
    
    const [owner, nasabah1, nasabah2] = await ethers.getSigners();
    
    console.log("🏦 Demo Bank Digital");
    console.log("📍 Contract address:", contractAddress);
    
    try {
        // 1. Daftar nasabah
        console.log("\n👤 Mendaftarkan nasabah...");
        await bank.connect(nasabah1).daftar("Budi Santoso");
        console.log("✅ Budi berhasil terdaftar");
        
        await bank.connect(nasabah2).daftar("Siti Aminah");
        console.log("✅ Siti berhasil terdaftar");
        
        // 2. Deposit
        console.log("\n💰 Melakukan deposit...");
        const depositAmount = ethers.utils.parseEther("2");
        await bank.connect(nasabah1).deposit({ value: depositAmount });
        console.log("✅ Budi deposit 2 ETH");
        
        // Cek saldo
        const saldoBudi = await bank.connect(nasabah1).cekSaldo();
        console.log("💳 Saldo Budi:", ethers.utils.formatEther(saldoBudi), "ETH");
        
        // 3. Transfer
        console.log("\n🔄 Melakukan transfer...");
        const transferAmount = ethers.utils.parseEther("0.5");
        await bank.connect(nasabah1).transfer(nasabah2.address, transferAmount);
        console.log("✅ Budi transfer 0.5 ETH ke Siti");
        
        // Cek saldo setelah transfer
        const saldoBudiSetelah = await bank.connect(nasabah1).cekSaldo();
        const saldoSiti = await bank.connect(nasabah2).cekSaldo();
        
        console.log("💳 Saldo Budi setelah transfer:", ethers.utils.formatEther(saldoBudiSetelah), "ETH");
        console.log("💳 Saldo Siti setelah transfer:", ethers.utils.formatEther(saldoSiti), "ETH");
        
        // 4. Info lengkap nasabah
        console.log("\n📋 Info lengkap nasabah:");
        const infoBudi = await bank.infoNasabah(nasabah1.address);
        console.log("👤 Budi:", {
            nama: infoBudi.nama,
            saldo: ethers.utils.formatEther(infoBudi.saldo) + " ETH",
            aktif: infoBudi.aktif,
            tanggalDaftar: new Date(infoBudi.tanggalDaftar * 1000).toLocaleDateString()
        });
        
        // 5. Statistik bank
        console.log("\n📊 Statistik Bank:");
        const stats = await bank.statistikBank();
        console.log("👥 Jumlah nasabah:", stats.jumlahNasabah.toString());
        console.log("💰 Total saldo bank:", ethers.utils.formatEther(stats.totalSaldoBank), "ETH");
        console.log("🏪 Status bank:", stats.statusBank ? "Buka ✅" : "Tutup ❌");
        
    } catch (error) {
        console.error("❌ Error:", error.message);
    }
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
        console.error(error);
        process.exit(1);
    });
```

---

## Konfigurasi Hardhat untuk Multiple Networks

```javascript
// hardhat.config.js
require("@nomiclabs/hardhat-waffle");
require("dotenv").config();

module.exports = {
    solidity: {
        version: "0.8.19",
        settings: {
            optimizer: {
                enabled: true,
                runs: 200
            }
        }
    },
    networks: {
        // Local network
        localhost: {
            url: "http://127.0.0.1:8545"
        },
        
        // BSC Testnet (gratis untuk testing)
        bscTestnet: {
            url: "https://data-seed-prebsc-1-s1.binance.org:8545/",
            chainId: 97,
            accounts: [process.env.PRIVATE_KEY],
            gasPrice: 20000000000, // 20 gwei
        },
        
        // Ethereum Sepolia Testnet (gratis untuk testing)
        sepolia: {
            url: `https://sepolia.infura.io/v3/${process.env.INFURA_PROJECT_ID}`,
            chainId: 11155111,
            accounts: [process.env.PRIVATE_KEY],
        },
        
        // Polygon Mumbai Testnet
        mumbai: {
            url: "https://rpc-mumbai.maticvigil.com/",
            chainId: 80001,
            accounts: [process.env.PRIVATE_KEY],
        }
    },
    
    // Gas reporter untuk optimasi
    gasReporter: {
        enabled: true,
        currency: 'USD',
        gasPrice: 20
    }
};
```

## Environment Variables (.env)

```bash
# Private key wallet (JANGAN SHARE!)
PRIVATE_KEY=your_private_key_without_0x

# Infura project ID untuk Ethereum networks
INFURA_PROJECT_ID=your_infura_project_id

# API keys untuk verification (optional)
BSCSCAN_API_KEY=your_bscscan_api_key
ETHERSCAN_API_KEY=your_etherscan_api_key
```

---

## Tips untuk Pemula

### 1. **Keamanan Wallet**
```bash
# JANGAN PERNAH commit private key ke Git!
echo ".env" >> .gitignore

# Untuk testing, buat wallet baru yang khusus development
# Jangan pakai wallet utama yang ada asset berharga
```

### 2. **Gas Fee**
- **Testnet**: Gratis, ambil faucet token
- **Mainnet**: Butuh ETH/BNB real untuk gas fee
- **Estimasi**: Simple contract ~50,000-100,000 gas

### 3. **Testing**
```bash
# Selalu test dulu di testnet sebelum mainnet
npx hardhat test                    # Test lokal
npx hardhat test --network localhost # Test di local blockchain
```

### 4. **Debugging**
```solidity
// Pakai console.log untuk debugging (import hardhat)
import "hardhat/console.sol";

contract Debug {
    function testFunction(uint x) public {
        console.log("Nilai x adalah:", x);
    }
}
```

### 5. **Best Practices**
- **Naming**: Gunakan nama yang jelas (camelCase)
- **Comments**: Tulis komentar untuk logika kompleks
- **Modifiers**: Gunakan modifier untuk validasi berulang
- **Events**: Emit event untuk tracking perubahan
- **Error Messages**: Tulis pesan error yang informatif

---

## Latihan Mandiri

### Latihan 1: Voting System
Buat contract untuk sistem voting sederhana:
- Admin bisa menambah kandidat
- User bisa vote (1 kali per address)
- Bisa lihat hasil voting real-time

### Latihan 2: NFT Marketplace Sederhana
Buat marketplace untuk jual-beli item digital:
- User bisa mint NFT dengan metadata
- User bisa jual NFT dengan harga tertentu
- User lain bisa beli NFT

### Latihan 3: Crowdfunding Platform
Buat platform crowdfunding:
- Creator bisa buat campaign dengan target dan deadline
- User bisa donate ke campaign
- Kalau target tercapai, dana ke creator
- Kalau gagal, dana dikembalikan

---

## Kesimpulan Bagian 1

Selamat! Anda sudah mempelajari dasar-dasar Solidity:

✅ **Setup environment** dan tools development
✅ **Syntax dasar** Solidity (variables, functions, control flow)
✅ **Data structures** (arrays, mappings, structs)
✅ **Events dan modifiers** untuk keamanan
✅ **Error handling** yang proper
✅ **Testing dan deployment** ke blockchain
✅ **Contract interaction** dari script

### **Next Steps:**
- **Bagian 2**: ERC20 Token Development
- **Bagian 3**: Contract-to-Contract Interaction
- **Bagian 4**: Cross-Chain Communication

Dengan dasar ini, Anda sudah bisa membuat smart contract sederhana untuk berbagai use case. Lanjutkan ke bagian selanjutnya untuk mempelajari token development yang lebih advanced!

**Happy coding! 🚀**
