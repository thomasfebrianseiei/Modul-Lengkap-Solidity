# Bagian 2: ERC20 Token Development untuk Pemula
## Membuat Cryptocurrency Sendiri dari Nol

---

## Apa itu ERC20 Token?

**ERC20** adalah standar untuk membuat **cryptocurrency** di blockchain Ethereum dan blockchain sejenis (BSC, Polygon, dll).

### Analogi Sederhana:
- **ERC20** = Aturan standar untuk membuat uang digital
- **Token** = Uang digital yang Anda buat
- **Smart Contract** = Bank yang mengatur token Anda

### Contoh Token ERC20 Terkenal:
- **USDT** (Tether) - Stablecoin
- **LINK** (Chainlink) - Utility token
- **UNI** (Uniswap) - Governance token
- **SHIB** (Shiba Inu) - Meme token

### Mengapa Perlu Standar ERC20?
- **Kompatibilitas**: Semua wallet dan exchange bisa support
- **Interoperability**: Token bisa dipakai di berbagai DApp
- **Security**: Standar yang sudah teruji dan aman

---

## Fungsi-Fungsi Wajib ERC20

Setiap token ERC20 **HARUS** punya 6 fungsi ini:

```solidity
interface IERC20 {
    // 1. Total supply - jumlah total token yang ada
    function totalSupply() external view returns (uint256);
    
    // 2. Balance of - saldo token seseorang
    function balanceOf(address account) external view returns (uint256);
    
    // 3. Transfer - kirim token ke orang lain
    function transfer(address to, uint256 amount) external returns (bool);
    
    // 4. Allowance - cek berapa token yang diizinkan untuk dipindah
    function allowance(address owner, address spender) external view returns (uint256);
    
    // 5. Approve - izinkan orang lain pindahkan token kita
    function approve(address spender, uint256 amount) external returns (bool);
    
    // 6. Transfer from - pindahkan token atas nama orang lain
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
    
    // Events wajib
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}
```

**Penjelasan Sederhana:**
- **totalSupply()**: "Berapa total token yang ada di dunia?"
- **balanceOf()**: "Berapa token yang dimiliki si A?"
- **transfer()**: "Si A kirim token ke si B"
- **approve()**: "Si A izinkan si C untuk mengambil tokennya"
- **allowance()**: "Berapa token si C boleh ambil dari si A?"
- **transferFrom()**: "Si C ambil token si A dan kirim ke si B"

---

## Token ERC20 Paling Sederhana

Mari buat token pertama yang paling basic:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract MyFirstToken {
    // Informasi token
    string public name = "My First Token";    // Nama token
    string public symbol = "MFT";             // Simbol token (seperti BTC, ETH)
    uint8 public decimals = 18;               // Desimal (seperti 1 ETH = 1000000000000000000 wei)
    uint256 public totalSupply;               // Total supply token
    
    // Penyimpanan saldo setiap address
    mapping(address => uint256) public balanceOf;
    
    // Penyimpanan allowance - siapa boleh ambil berapa dari siapa
    mapping(address => mapping(address => uint256)) public allowance;
    
    // Events
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    
    // Constructor - dijalankan sekali saat deploy
    constructor(uint256 _initialSupply) {
        totalSupply = _initialSupply * 10**decimals;  // Kalikan dengan decimals
        balanceOf[msg.sender] = totalSupply;          // Semua token ke deployer
        
        emit Transfer(address(0), msg.sender, totalSupply);  // Event mint
    }
    
    // Transfer token dari pengirim ke penerima
    function transfer(address _to, uint256 _amount) public returns (bool) {
        // Validasi
        require(_to != address(0), "Tidak bisa kirim ke address 0");
        require(balanceOf[msg.sender] >= _amount, "Saldo tidak cukup");
        
        // Transfer
        balanceOf[msg.sender] -= _amount;
        balanceOf[_to] += _amount;
        
        // Emit event
        emit Transfer(msg.sender, _to, _amount);
        
        return true;
    }
    
    // Approve - izinkan spender untuk mengambil token kita
    function approve(address _spender, uint256 _amount) public returns (bool) {
        allowance[msg.sender][_spender] = _amount;
        
        emit Approval(msg.sender, _spender, _amount);
        
        return true;
    }
    
    // Transfer from - transfer atas nama orang lain
    function transferFrom(address _from, address _to, uint256 _amount) public returns (bool) {
        // Validasi
        require(_to != address(0), "Tidak bisa kirim ke address 0");
        require(balanceOf[_from] >= _amount, "Saldo tidak cukup");
        require(allowance[_from][msg.sender] >= _amount, "Allowance tidak cukup");
        
        // Transfer
        balanceOf[_from] -= _amount;
        balanceOf[_to] += _amount;
        allowance[_from][msg.sender] -= _amount;  // Kurangi allowance
        
        emit Transfer(_from, _to, _amount);
        
        return true;
    }
}
```

---

## Token dengan Vesting (Lock Token) - LENGKAP

Vesting berguna untuk team tokens, investor tokens yang dikunci:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract VestingToken {
    string public name = "Vesting Token";
    string public symbol = "VEST";
    uint8 public decimals = 18;
    uint256 public totalSupply;
    
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    
    // Vesting structure
    struct VestingSchedule {
        uint256 totalAmount;        // Total token yang di-vest
        uint256 releasedAmount;     // Token yang sudah di-release
        uint256 startTime;          // Kapan vesting mulai
        uint256 cliffDuration;      // Periode cliff (tidak bisa claim)
        uint256 vestingDuration;    // Total durasi vesting
        bool revokable;             // Bisa dicabut atau tidak
        bool revoked;               // Sudah dicabut atau belum
    }
    
    mapping(address => VestingSchedule) public vestingSchedules;
    
    address public owner;
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    event VestingCreated(address indexed beneficiary, uint256 amount, uint256 duration);
    event TokensReleased(address indexed beneficiary, uint256 amount);
    event VestingRevoked(address indexed beneficiary);
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner");
        _;
    }
    
    constructor(uint256 _initialSupply) {
        owner = msg.sender;
        totalSupply = _initialSupply * 10**decimals;
        balanceOf[msg.sender] = totalSupply;
        
        emit Transfer(address(0), msg.sender, totalSupply);
    }
    
    // Standard ERC20 functions
    function transfer(address _to, uint256 _amount) public returns (bool) {
        require(_to != address(0), "Transfer to zero address");
        require(balanceOf[msg.sender] >= _amount, "Insufficient balance");
        
        balanceOf[msg.sender] -= _amount;
        balanceOf[_to] += _amount;
        
        emit Transfer(msg.sender, _to, _amount);
        return true;
    }
    
    function approve(address _spender, uint256 _amount) public returns (bool) {
        allowance[msg.sender][_spender] = _amount;
        emit Approval(msg.sender, _spender, _amount);
        return true;
    }
    
    function transferFrom(address _from, address _to, uint256 _amount) public returns (bool) {
        require(_to != address(0), "Transfer to zero address");
        require(balanceOf[_from] >= _amount, "Insufficient balance");
        require(allowance[_from][msg.sender] >= _amount, "Insufficient allowance");
        
        balanceOf[_from] -= _amount;
        balanceOf[_to] += _amount;
        allowance[_from][msg.sender] -= _amount;
        
        emit Transfer(_from, _to, _amount);
        return true;
    }
    
    // Vesting functions
    
    // Buat jadwal vesting untuk beneficiary
    function createVestingSchedule(
        address _beneficiary,
        uint256 _amount,
        uint256 _cliffDuration,    // Cliff dalam detik (contoh: 30 days = 30 * 24 * 60 * 60)
        uint256 _vestingDuration,  // Total vesting dalam detik (contoh: 365 days)
        bool _revokable
    ) public onlyOwner {
        require(_beneficiary != address(0), "Beneficiary cannot be zero address");
        require(_amount > 0, "Amount must be greater than zero");
        require(_vestingDuration > 0, "Vesting duration must be greater than zero");
        require(_cliffDuration <= _vestingDuration, "Cliff duration cannot exceed vesting duration");
        require(vestingSchedules[_beneficiary].totalAmount == 0, "Vesting schedule already exists");
        require(balanceOf[owner] >= _amount, "Insufficient balance for vesting");
        
        // Transfer token dari owner ke contract untuk di-lock
        balanceOf[owner] -= _amount;
        
        // Buat vesting schedule
        vestingSchedules[_beneficiary] = VestingSchedule({
            totalAmount: _amount,
            releasedAmount: 0,
            startTime: block.timestamp,
            cliffDuration: _cliffDuration,
            vestingDuration: _vestingDuration,
            revokable: _revokable,
            revoked: false
        });
        
        emit VestingCreated(_beneficiary, _amount, _vestingDuration);
    }
    
    // Release token yang sudah vested
    function releaseVestedTokens() public {
        VestingSchedule storage schedule = vestingSchedules[msg.sender];
        require(schedule.totalAmount > 0, "No vesting schedule found");
        require(!schedule.revoked, "Vesting has been revoked");
        require(block.timestamp >= schedule.startTime + schedule.cliffDuration, "Cliff period not over");
        
        uint256 releasableAmount = _getReleasableAmount(msg.sender);
        require(releasableAmount > 0, "No tokens available for release");
        
        schedule.releasedAmount += releasableAmount;
        balanceOf[msg.sender] += releasableAmount;
        
        emit Transfer(address(this), msg.sender, releasableAmount);
        emit TokensReleased(msg.sender, releasableAmount);
    }
    
    // Hitung berapa token yang bisa di-release
    function _getReleasableAmount(address _beneficiary) internal view returns (uint256) {
        VestingSchedule memory schedule = vestingSchedules[_beneficiary];
        
        if (schedule.totalAmount == 0 || schedule.revoked) {
            return 0;
        }
        
        // Kalau belum lewat cliff period
        if (block.timestamp < schedule.startTime + schedule.cliffDuration) {
            return 0;
        }
        
        // Kalau sudah lewat vesting period, release semua
        if (block.timestamp >= schedule.startTime + schedule.vestingDuration) {
            return schedule.totalAmount - schedule.releasedAmount;
        }
        
        // Hitung berapa yang sudah vested berdasarkan waktu
        uint256 timeElapsed = block.timestamp - schedule.startTime;
        uint256 vestedAmount = (schedule.totalAmount * timeElapsed) / schedule.vestingDuration;
        
        return vestedAmount - schedule.releasedAmount;
    }
    
    // View function untuk cek releasable amount
    function getReleasableAmount(address _beneficiary) public view returns (uint256) {
        return _getReleasableAmount(_beneficiary);
    }
    
    // Owner bisa revoke vesting (kalau revokable)
    function revokeVesting(address _beneficiary) public onlyOwner {
        VestingSchedule storage schedule = vestingSchedules[_beneficiary];
        require(schedule.totalAmount > 0, "No vesting schedule found");
        require(schedule.revokable, "Vesting is not revokable");
        require(!schedule.revoked, "Vesting already revoked");
        
        // Release token yang sudah vested
        uint256 releasableAmount = _getReleasableAmount(_beneficiary);
        if (releasableAmount > 0) {
            schedule.releasedAmount += releasableAmount;
            balanceOf[_beneficiary] += releasableAmount;
            emit Transfer(address(this), _beneficiary, releasableAmount);
            emit TokensReleased(_beneficiary, releasableAmount);
        }
        
        // Return sisa token ke owner
        uint256 remainingAmount = schedule.totalAmount - schedule.releasedAmount;
        if (remainingAmount > 0) {
            balanceOf[owner] += remainingAmount;
            emit Transfer(address(this), owner, remainingAmount);
        }
        
        schedule.revoked = true;
        emit VestingRevoked(_beneficiary);
    }
    
    // Get full vesting info
    function getVestingInfo(address _beneficiary) public view returns (
        uint256 totalAmount,
        uint256 releasedAmount,
        uint256 releasableAmount,
        uint256 startTime,
        uint256 cliffDuration,
        uint256 vestingDuration,
        bool revokable,
        bool revoked
    ) {
        VestingSchedule memory schedule = vestingSchedules[_beneficiary];
        return (
            schedule.totalAmount,
            schedule.releasedAmount,
            _getReleasableAmount(_beneficiary),
            schedule.startTime,
            schedule.cliffDuration,
            schedule.vestingDuration,
            schedule.revokable,
            schedule.revoked
        );
    }
}
```

**Penjelasan Vesting:**
- **Cliff Period**: Waktu di mana tidak ada token yang bisa di-claim
- **Vesting Duration**: Total waktu untuk release semua token
- **Linear Release**: Token di-release secara bertahap setiap detik
- **Revokable**: Owner bisa batalkan vesting jika diperlukan

**Contoh Penggunaan:**
```solidity
// Team tokens: 100,000 token, cliff 6 bulan, vesting 2 tahun
createVestingSchedule(teamAddress, 100000 * 10**18, 180 days, 730 days, true);

// Investor tokens: 50,000 token, cliff 3 bulan, vesting 1 tahun
createVestingSchedule(investorAddress, 50000 * 10**18, 90 days, 365 days, false);
```

---

## Token dengan OpenZeppelin (Recommended)

Untuk production, lebih baik pakai OpenZeppelin library yang sudah teruji:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

contract MyProductionToken is ERC20, Ownable, Pausable, ERC20Burnable {
    uint256 public constant MAX_SUPPLY = 1000000000 * 10**18; // 1 billion tokens
    
    // Mapping untuk whitelist (bebas fee)
    mapping(address => bool) public feeWhitelist;
    
    // Transfer fee dalam basis points (100 = 1%)
    uint256 public transferFee = 0;
    address public feeRecipient;
    
    event FeeWhitelistUpdated(address indexed account, bool status);
    event TransferFeeUpdated(uint256 newFee);
    event FeeRecipientUpdated(address indexed newRecipient);
    
    constructor(
        string memory name,
        string memory symbol,
        uint256 initialSupply,
        address owner
    ) ERC20(name, symbol) {
        require(initialSupply <= MAX_SUPPLY, "Initial supply exceeds max supply");
        
        _transferOwnership(owner);
        feeRecipient = owner;
        
        // Mint initial supply ke owner
        _mint(owner, initialSupply);
        
        // Whitelist owner dari fee
        feeWhitelist[owner] = true;
    }
    
    // Override transfer untuk implement fee
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal override whenNotPaused {
        super._beforeTokenTransfer(from, to, amount);
    }
    
    function transfer(address to, uint256 amount) public override returns (bool) {
        address owner = _msgSender();
        
        if (transferFee > 0 && !feeWhitelist[owner] && !feeWhitelist[to]) {
            uint256 feeAmount = (amount * transferFee) / 10000;
            uint256 transferAmount = amount - feeAmount;
            
            // Transfer fee ke fee recipient
            _transfer(owner, feeRecipient, feeAmount);
            // Transfer amount dikurangi fee
            _transfer(owner, to, transferAmount);
        } else {
            _transfer(owner, to, amount);
        }
        
        return true;
    }
    
    function transferFrom(address from, address to, uint256 amount) public override returns (bool) {
        address spender = _msgSender();
        _spendAllowance(from, spender, amount);
        
        if (transferFee > 0 && !feeWhitelist[from] && !feeWhitelist[to]) {
            uint256 feeAmount = (amount * transferFee) / 10000;
            uint256 transferAmount = amount - feeAmount;
            
            _transfer(from, feeRecipient, feeAmount);
            _transfer(from, to, transferAmount);
        } else {
            _transfer(from, to, amount);
        }
        
        return true;
    }
    
    // Minting functions (only owner, with max supply check)
    function mint(address to, uint256 amount) public onlyOwner {
        require(totalSupply() + amount <= MAX_SUPPLY, "Exceeds max supply");
        _mint(to, amount);
    }
    
    function batchMint(address[] memory recipients, uint256[] memory amounts) public onlyOwner {
        require(recipients.length == amounts.length, "Arrays length mismatch");
        
        uint256 totalMintAmount = 0;
        for (uint256 i = 0; i < amounts.length; i++) {
            totalMintAmount += amounts[i];
        }
        
        require(totalSupply() + totalMintAmount <= MAX_SUPPLY, "Exceeds max supply");
        
        for (uint256 i = 0; i < recipients.length; i++) {
            _mint(recipients[i], amounts[i]);
        }
    }
    
    // Fee management
    function setTransferFee(uint256 newFee) public onlyOwner {
        require(newFee <= 1000, "Fee cannot exceed 10%"); // max 10%
        transferFee = newFee;
        emit TransferFeeUpdated(newFee);
    }
    
    function setFeeRecipient(address newRecipient) public onlyOwner {
        require(newRecipient != address(0), "Fee recipient cannot be zero address");
        feeRecipient = newRecipient;
        emit FeeRecipientUpdated(newRecipient);
    }
    
    function updateFeeWhitelist(address account, bool status) public onlyOwner {
        feeWhitelist[account] = status;
        emit FeeWhitelistUpdated(account, status);
    }
    
    function batchUpdateFeeWhitelist(address[] memory accounts, bool status) public onlyOwner {
        for (uint256 i = 0; i < accounts.length; i++) {
            feeWhitelist[accounts[i]] = status;
            emit FeeWhitelistUpdated(accounts[i], status);
        }
    }
    
    // Pause functions
    function pause() public onlyOwner {
        _pause();
    }
    
    function unpause() public onlyOwner {
        _unpause();
    }
    
    // Emergency functions
    function emergencyWithdraw(address token, uint256 amount) public onlyOwner {
        if (token == address(0)) {
            payable(owner()).transfer(amount);
        } else {
            IERC20(token).transfer(owner(), amount);
        }
    }
    
    // View functions
    function getTokenMetrics() public view returns (
        uint256 currentSupply,
        uint256 maxSupply,
        uint256 remainingMintable,
        uint256 currentFee,
        address currentFeeRecipient,
        bool isPaused
    ) {
        return (
            totalSupply(),
            MAX_SUPPLY,
            MAX_SUPPLY - totalSupply(),
            transferFee,
            feeRecipient,
            paused()
        );
    }
}
```

**Keuntungan OpenZeppelin:**
- ✅ **Battle-tested**: Sudah diaudit dan dipakai jutaan contract
- ✅ **Gas optimized**: Efficient dan murah gas
- ✅ **Modular**: Bisa pilih fitur yang dibutuhkan
- ✅ **Upgradeable**: Support untuk proxy pattern
- ✅ **Community**: Dokumentasi lengkap dan support bagus

---

## Testing Token ERC20

Buat test comprehensive untuk token:

```javascript
// test/MyProductionToken.test.js
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("MyProductionToken", function () {
    let Token, token, owner, addr1, addr2, addr3;
    const initialSupply = ethers.utils.parseEther("1000000"); // 1M tokens
    const maxSupply = ethers.utils.parseEther("1000000000"); // 1B tokens
    
    beforeEach(async function () {
        Token = await ethers.getContractFactory("MyProductionToken");
        [owner, addr1, addr2, addr3] = await ethers.getSigners();
        
        token = await Token.deploy(
            "My Production Token",
            "MPT",
            initialSupply,
            owner.address
        );
        await token.deployed();
    });
    
    describe("Deployment", function () {
        it("Should set correct token info", async function () {
            expect(await token.name()).to.equal("My Production Token");
            expect(await token.symbol()).to.equal("MPT");
            expect(await token.decimals()).to.equal(18);
            expect(await token.totalSupply()).to.equal(initialSupply);
            expect(await token.MAX_SUPPLY()).to.equal(maxSupply);
        });
        
        it("Should assign initial supply to owner", async function () {
            const ownerBalance = await token.balanceOf(owner.address);
            expect(ownerBalance).to.equal(initialSupply);
        });
        
        it("Should whitelist owner from fees", async function () {
            expect(await token.feeWhitelist(owner.address)).to.equal(true);
        });
    });
    
    describe("Basic ERC20 Functions", function () {
        it("Should transfer tokens between accounts", async function () {
            const transferAmount = ethers.utils.parseEther("100");
            
            await token.transfer(addr1.address, transferAmount);
            expect(await token.balanceOf(addr1.address)).to.equal(transferAmount);
            
            await token.connect(addr1).transfer(addr2.address, transferAmount);
            expect(await token.balanceOf(addr2.address)).to.equal(transferAmount);
            expect(await token.balanceOf(addr1.address)).to.equal(0);
        });
        
        it("Should handle approve and transferFrom", async function () {
            const approveAmount = ethers.utils.parseEther("100");
            const transferAmount = ethers.utils.parseEther("50");
            
            await token.transfer(addr1.address, approveAmount);
            await token.connect(addr1).approve(addr2.address, approveAmount);
            
            expect(await token.allowance(addr1.address, addr2.address)).to.equal(approveAmount);
            
            await token.connect(addr2).transferFrom(addr1.address, addr3.address, transferAmount);
            
            expect(await token.balanceOf(addr3.address)).to.equal(transferAmount);
            expect(await token.allowance(addr1.address, addr2.address)).to.equal(approveAmount.sub(transferAmount));
        });
    });
    
    describe("Fee System", function () {
        beforeEach(async function () {
            // Set 1% transfer fee
            await token.setTransferFee(100); // 100 basis points = 1%
            await token.setFeeRecipient(addr3.address);
        });
        
        it("Should charge fee on transfers", async function () {
            const transferAmount = ethers.utils.parseEther("100");
            const expectedFee = ethers.utils.parseEther("1"); // 1% of 100
            const expectedReceived = transferAmount.sub(expectedFee);
            
            // Transfer dari owner (whitelisted) ke addr1 - no fee
            await token.transfer(addr1.address, transferAmount);
            expect(await token.balanceOf(addr1.address)).to.equal(transferAmount);
            
            // Transfer dari addr1 (not whitelisted) ke addr2 - with fee
            await token.connect(addr1).transfer(addr2.address, transferAmount);
            
            expect(await token.balanceOf(addr2.address)).to.equal(expectedReceived);
            expect(await token.balanceOf(addr3.address)).to.equal(expectedFee); // fee recipient
        });
        
        it("Should not charge fee for whitelisted addresses", async function () {
            const transferAmount = ethers.utils.parseEther("100");
            
            // Add addr1 to whitelist
            await token.updateFeeWhitelist(addr1.address, true);
            
            await token.transfer(addr1.address, transferAmount);
            await token.connect(addr1).transfer(addr2.address, transferAmount);
            
            // No fee charged
            expect(await token.balanceOf(addr2.address)).to.equal(transferAmount);
            expect(await token.balanceOf(addr3.address)).to.equal(0);
        });
    });
    
    describe("Minting", function () {
        it("Should allow owner to mint tokens", async function () {
            const mintAmount = ethers.utils.parseEther("1000");
            const initialSupply = await token.totalSupply();
            
            await token.mint(addr1.address, mintAmount);
            
            expect(await token.balanceOf(addr1.address)).to.equal(mintAmount);
            expect(await token.totalSupply()).to.equal(initialSupply.add(mintAmount));
        });
        
        it("Should not exceed max supply", async function () {
            const remainingMintable = maxSupply.sub(await token.totalSupply());
            const excessAmount = remainingMintable.add(ethers.utils.parseEther("1"));
            
            await expect(
                token.mint(addr1.address, excessAmount)
            ).to.be.revertedWith("Exceeds max supply");
        });
        
        it("Should allow batch minting", async function () {
            const recipients = [addr1.address, addr2.address];
            const amounts = [ethers.utils.parseEther("100"), ethers.utils.parseEther("200")];
            
            await token.batchMint(recipients, amounts);
            
            expect(await token.balanceOf(addr1.address)).to.equal(amounts[0]);
            expect(await token.balanceOf(addr2.address)).to.equal(amounts[1]);
        });
    });
    
    describe("Burning", function () {
        beforeEach(async function () {
            const transferAmount = ethers.utils.parseEther("1000");
            await token.transfer(addr1.address, transferAmount);
        });
        
        it("Should allow users to burn their tokens", async function () {
            const burnAmount = ethers.utils.parseEther("100");
            const initialBalance = await token.balanceOf(addr1.address);
            const initialSupply = await token.totalSupply();
            
            await token.connect(addr1).burn(burnAmount);
            
            expect(await token.balanceOf(addr1.address)).to.equal(initialBalance.sub(burnAmount));
            expect(await token.totalSupply()).to.equal(initialSupply.sub(burnAmount));
        });
        
        it("Should allow burnFrom with allowance", async function () {
            const burnAmount = ethers.utils.parseEther("100");
            
            await token.connect(addr1).approve(addr2.address, burnAmount);
            await token.connect(addr2).burnFrom(addr1.address, burnAmount);
            
            const balance = await token.balanceOf(addr1.address);
            expect(balance).to.equal(ethers.utils.parseEther("900")); // 1000 - 100
        });
    });
    
    describe("Pause Functionality", function () {
        it("Should pause and unpause transfers", async function () {
            const transferAmount = ethers.utils.parseEther("100");
            
            // Pause contract
            await token.pause();
            
            await expect(
                token.transfer(addr1.address, transferAmount)
            ).to.be.revertedWith("Pausable: paused");
            
            // Unpause contract
            await token.unpause();
            
            await token.transfer(addr1.address, transferAmount);
            expect(await token.balanceOf(addr1.address)).to.equal(transferAmount);
        });
    });
    
    describe("Access Control", function () {
        it("Should only allow owner to mint", async function () {
            const mintAmount = ethers.utils.parseEther("100");
            
            await expect(
                token.connect(addr1).mint(addr2.address, mintAmount)
            ).to.be.revertedWith("Ownable: caller is not the owner");
        });
        
        it("Should only allow owner to set fees", async function () {
            await expect(
                token.connect(addr1).setTransferFee(500)
            ).to.be.revertedWith("Ownable: caller is not the owner");
        });
        
        it("Should transfer ownership", async function () {
            await token.transferOwnership(addr1.address);
            expect(await token.owner()).to.equal(addr1.address);
        });
    });
    
    describe("View Functions", function () {
        it("Should return correct token metrics", async function () {
            const metrics = await token.getTokenMetrics();
            
            expect(metrics.currentSupply).to.equal(initialSupply);
            expect(metrics.maxSupply).to.equal(maxSupply);
            expect(metrics.remainingMintable).to.equal(maxSupply.sub(initialSupply));
            expect(metrics.currentFee).to.equal(0);
            expect(metrics.isPaused).to.equal(false);
        });
    });
});
```

---

## Deploy Script untuk Token

```javascript
// scripts/deploy-token.js
async function main() {
    console.log("🚀 Deploying MyProductionToken...");
    
    const [deployer] = await ethers.getSigners();
    console.log("📝 Deploying with account:", deployer.address);
    
    const balance = await deployer.getBalance();
    console.log("💰 Account balance:", ethers.utils.formatEther(balance), "ETH");
    
    // Token parameters
    const tokenParams = {
        name: "My Awesome Token",
        symbol: "MAT",
        initialSupply: ethers.utils.parseEther("1000000"), // 1M tokens
        owner: deployer.address
    };
    
    console.log("🔧 Token parameters:");
    console.log("   Name:", tokenParams.name);
    console.log("   Symbol:", tokenParams.symbol);
    console.log("   Initial Supply:", ethers.utils.formatEther(tokenParams.initialSupply));
    console.log("   Owner:", tokenParams.owner);
    
    // Deploy contract
    const Token = await ethers.getContractFactory("MyProductionToken");
    const token = await Token.deploy(
        tokenParams.name,
        tokenParams.symbol,
        tokenParams.initialSupply,
        tokenParams.owner
    );
    
    await token.deployed();
    
    console.log("✅ Token deployed successfully!");
    console.log("📍 Contract address:", token.address);
    
    // Verify deployment
    console.log("\n🔍 Verifying deployment...");
    const deployedName = await token.name();
    const deployedSymbol = await token.symbol();
    const deployedSupply = await token.totalSupply();
    const deployedOwner = await token.owner();
    
    console.log("✅ Verification successful:");
    console.log("   Name:", deployedName);
    console.log("   Symbol:", deployedSymbol);
    console.log("   Total Supply:", ethers.utils.formatEther(deployedSupply));
    console.log("   Owner:", deployedOwner);
    
    // Save deployment info
    const deploymentInfo = {
        network: network.name,
        chainId: network.config.chainId,
        contractAddress: token.address,
        tokenName: deployedName,
        tokenSymbol: deployedSymbol,
        initialSupply: ethers.utils.formatEther(deployedSupply),
        owner: deployedOwner,
        deployer: deployer.address,
        deploymentTime: new Date().toISOString(),
        transactionHash: token.deployTransaction.hash,
        blockNumber: token.deployTransaction.blockNumber
    };
    
    // Write to file
    const fs = require('fs');
    const deploymentDir = './deployments';
    if (!fs.existsSync(deploymentDir)) {
        fs.mkdirSync(deploymentDir);
    }
    
    fs.writeFileSync(
        `${deploymentDir}/${network.name}-token-deployment.json`,
        JSON.stringify(deploymentInfo, null, 2)
    );
    
    console.log("\n💾 Deployment info saved to:", `deployments/${network.name}-token-deployment.json`);
    
    // Contract verification command
    console.log("\n📋 To verify on block explorer, run:");
    console.log(`npx hardhat verify --network ${network.name} ${token.address} "${tokenParams.name}" "${tokenParams.symbol}" "${tokenParams.initialSupply}" "${tokenParams.owner}"`);
    
    return token;
}

main()
    .then(() => {
        console.log("🎉 Deployment completed successfully!");
        process.exit(0);
    })
    .catch((error) => {
        console.error("❌ Deployment failed:", error);
        process.exit(1);
    });
```

---

## Interaksi dengan Token

```javascript
// scripts/token-interaction.js
async function main() {
    // Load deployed token
    const tokenAddress = "0x..."; // Replace with actual address
    const Token = await ethers.getContractFactory("MyProductionToken");
    const token = Token.attach(tokenAddress);
    
    const [owner, user1, user2] = await ethers.getSigners();
    
    console.log("🪙 Token Interaction Demo");
    console.log("📍 Token address:", tokenAddress);
    
    try {
        // 1. Get token info
        console.log("\n📊 Token Information:");
        const name = await token.name();
        const symbol = await token.symbol();
        const totalSupply = await token.totalSupply();
        const maxSupply = await token.MAX_SUPPLY();
        
        console.log(`   Name: ${name}`);
        console.log(`   Symbol: ${symbol}`);
        console.log(`   Total Supply: ${ethers.utils.formatEther(totalSupply)} ${symbol}`);
        console.log(`   Max Supply: ${ethers.utils.formatEther(maxSupply)} ${symbol}`);
        
        // 2. Check balances
        console.log("\n💰 Current Balances:");
        const ownerBalance = await token.balanceOf(owner.address);
        const user1Balance = await token.balanceOf(user1.address);
        const user2Balance = await token.balanceOf(user2.address);
        
        console.log(`   Owner: ${ethers.utils.formatEther(ownerBalance)} ${symbol}`);
        console.log(`   User1: ${ethers.utils.formatEther(user1Balance)} ${symbol}`);
        console.log(`   User2: ${ethers.utils.formatEther(user2Balance)} ${symbol}`);
        
        // 3. Transfer tokens
        console.log("\n🔄 Transferring tokens...");
        const transferAmount = ethers.utils.parseEther("1000");
        
        const transferTx = await token.transfer(user1.address, transferAmount);
        await transferTx.wait();
        console.log(`   ✅ Transferred ${ethers.utils.formatEther(transferAmount)} ${symbol} to User1`);
        
        // 4. Approve and transferFrom
        console.log("\n📝 Testing approve/transferFrom...");
        const approveAmount = ethers.utils.parseEther("500");
        
        const approveTx = await token.connect(user1).approve(user2.address, approveAmount);
        await approveTx.wait();
        console.log(`   ✅ User1 approved ${ethers.utils.formatEther(approveAmount)} ${symbol} to User2`);
        
        const allowance = await token.allowance(user1.address, user2.address);
        console.log(`   📋 Current allowance: ${ethers.utils.formatEther(allowance)} ${symbol}`);
        
        const transferFromAmount = ethers.utils.parseEther("200");
        const transferFromTx = await token.connect(user2).transferFrom(
            user1.address, 
            user2.address, 
            transferFromAmount
        );
        await transferFromTx.wait();
        console.log(`   ✅ User2 transferred ${ethers.utils.formatEther(transferFromAmount)} ${symbol} from User1`);
        
        // 5. Mint new tokens (owner only)
        console.log("\n🏭 Minting new tokens...");
        const mintAmount = ethers.utils.parseEther("5000");
        
        const mintTx = await token.mint(user1.address, mintAmount);
        await mintTx.wait();
        console.log(`   ✅ Minted ${ethers.utils.formatEther(mintAmount)} ${symbol} to User1`);
        
        // 6. Set transfer fee
        console.log("\n💸 Setting transfer fee...");
        const feePercentage = 100; // 1% = 100 basis points
        
        const setFeeTx = await token.setTransferFee(feePercentage);
        await setFeeTx.wait();
        console.log(`   ✅ Set transfer fee to ${feePercentage / 100}%`);
        
        const feeRecipientTx = await token.setFeeRecipient(owner.address);
        await feeRecipientTx.wait();
        console.log(`   ✅ Set fee recipient to owner`);
        
        // 7. Test transfer with fee
        console.log("\n💰 Testing transfer with fee...");
        const transferWithFeeAmount = ethers.utils.parseEther("1000");
        
        const beforeOwnerBalance = await token.balanceOf(owner.address);
        const beforeUser1Balance = await token.balanceOf(user1.address);
        const beforeUser2Balance = await token.balanceOf(user2.address);
        
        const transferWithFeeTx = await token.connect(user1).transfer(user2.address, transferWithFeeAmount);
        await transferWithFeeTx.wait();
        
        const afterOwnerBalance = await token.balanceOf(owner.address);
        const afterUser1Balance = await token.balanceOf(user1.address);
        const afterUser2Balance = await token.balanceOf(user2.address);
        
        const feeCollected = afterOwnerBalance.sub(beforeOwnerBalance);
        const amountReceived = afterUser2Balance.sub(beforeUser2Balance);
        
        console.log(`   📤 User1 sent: ${ethers.utils.formatEther(transferWithFeeAmount)} ${symbol}`);
        console.log(`   📥 User2 received: ${ethers.utils.formatEther(amountReceived)} ${symbol}`);
        console.log(`   💸 Fee collected: ${ethers.utils.formatEther(feeCollected)} ${symbol}`);
        
        // 8. Burn tokens
        console.log("\n🔥 Burning tokens...");
        const burnAmount = ethers.utils.parseEther("500");
        
        const beforeBurnBalance = await token.balanceOf(user1.address);
        const beforeBurnSupply = await token.totalSupply();
        
        const burnTx = await token.connect(user1).burn(burnAmount);
        await burnTx.wait();
        
        const afterBurnBalance = await token.balanceOf(user1.address);
        const afterBurnSupply = await token.totalSupply();
        
        console.log(`   🔥 Burned: ${ethers.utils.formatEther(burnAmount)} ${symbol}`);
        console.log(`   📉 Supply reduced by: ${ethers.utils.formatEther(beforeBurnSupply.sub(afterBurnSupply))} ${symbol}`);
        
        // 9. Final balances and metrics
        console.log("\n📊 Final State:");
        const finalMetrics = await token.getTokenMetrics();
        
        console.log("   Token Metrics:");
        console.log(`     Current Supply: ${ethers.utils.formatEther(finalMetrics.currentSupply)} ${symbol}`);
        console.log(`     Max Supply: ${ethers.utils.formatEther(finalMetrics.maxSupply)} ${symbol}`);
        console.log(`     Remaining Mintable: ${ethers.utils.formatEther(finalMetrics.remainingMintable)} ${symbol}`);
        console.log(`     Transfer Fee: ${finalMetrics.currentFee / 100}%`);
        console.log(`     Is Paused: ${finalMetrics.isPaused}`);
        
        console.log("\n   Final Balances:");
        const finalOwnerBalance = await token.balanceOf(owner.address);
        const finalUser1Balance = await token.balanceOf(user1.address);
        const finalUser2Balance = await token.balanceOf(user2.address);
        
        console.log(`     Owner: ${ethers.utils.formatEther(finalOwnerBalance)} ${symbol}`);
        console.log(`     User1: ${ethers.utils.formatEther(finalUser1Balance)} ${symbol}`);
        console.log(`     User2: ${ethers.utils.formatEther(finalUser2Balance)} ${symbol}`);
        
    } catch (error) {
        console.error("❌ Error during interaction:", error.message);
    }
}

main()
    .then(() => {
        console.log("\n🎉 Token interaction completed!");
        process.exit(0);
    })
    .catch((error) => {
        console.error(error);
        process.exit(1);
    });
```

---

## Frontend Integration untuk Token

Contoh cara integrate token dengan frontend React:

```javascript
// frontend/hooks/useToken.js
import { useState, useEffect } from 'react';
import { ethers } from 'ethers';

const TOKEN_ABI = [
    "function name() view returns (string)",
    "function symbol() view returns (string)",
    "function decimals() view returns (uint8)",
    "function totalSupply() view returns (uint256)",
    "function balanceOf(address) view returns (uint256)",
    "function transfer(address, uint256) returns (bool)",
    "function approve(address, uint256) returns (bool)",
    "function allowance(address, address) view returns (uint256)",
    "function transferFrom(address, address, uint256) returns (bool)",
    "function mint(address, uint256)",
    "function burn(uint256)",
    "function setTransferFee(uint256)",
    "function getTokenMetrics() view returns (uint256, uint256, uint256, uint256, address, bool)",
    "event Transfer(address indexed from, address indexed to, uint256 value)",
    "event Approval(address indexed owner, address indexed spender, uint256 value)"
];

export function useToken(tokenAddress) {
    const [tokenInfo, setTokenInfo] = useState(null);
    const [userBalance, setUserBalance] = useState('0');
    const [isLoading, setIsLoading] = useState(false);
    const [provider, setProvider] = useState(null);
    const [signer, setSigner] = useState(null);
    const [contract, setContract] = useState(null);
    
    // Connect wallet
    const connectWallet = async () => {
        if (typeof window.ethereum !== 'undefined') {
            try {
                await window.ethereum.request({ method: 'eth_requestAccounts' });
                const provider = new ethers.providers.Web3Provider(window.ethereum);
                const signer = provider.getSigner();
                const contract = new ethers.Contract(tokenAddress, TOKEN_ABI, signer);
                
                setProvider(provider);
                setSigner(signer);
                setContract(contract);
                
                return { provider, signer, contract };
            } catch (error) {
                console.error('Failed to connect wallet:', error);
                throw error;
            }
        } else {
            throw new Error('MetaMask not installed');
        }
    };
    
    // Load token info
    const loadTokenInfo = async () => {
        if (!contract) return;
        
        setIsLoading(true);
        try {
            const [name, symbol, decimals, metrics] = await Promise.all([
                contract.name(),
                contract.symbol(),
                contract.decimals(),
                contract.getTokenMetrics()
            ]);
            
            setTokenInfo({
                name,
                symbol,
                decimals,
                totalSupply: ethers.utils.formatUnits(metrics.currentSupply, decimals),
                maxSupply: ethers.utils.formatUnits(metrics.maxSupply, decimals),
                transferFee: metrics.currentFee.toNumber() / 100, // Convert to percentage
                isPaused: metrics.isPaused
            });
        } catch (error) {
            console.error('Failed to load token info:', error);
        } finally {
            setIsLoading(false);
        }
    };
    
    // Load user balance
    const loadUserBalance = async () => {
        if (!contract || !signer) return;
        
        try {
            const address = await signer.getAddress();
            const balance = await contract.balanceOf(address);
            const decimals = await contract.decimals();
            
            setUserBalance(ethers.utils.formatUnits(balance, decimals));
        } catch (error) {
            console.error('Failed to load balance:', error);
        }
    };
    
    // Transfer tokens
    const transfer = async (to, amount) => {
        if (!contract || !tokenInfo) throw new Error('Contract not initialized');
        
        try {
            const amountWei = ethers.utils.parseUnits(amount.toString(), tokenInfo.decimals);
            const tx = await contract.transfer(to, amountWei);
            
            setIsLoading(true);
            await tx.wait();
            
            // Reload balance after transfer
            await loadUserBalance();
            
            return tx;
        } catch (error) {
            console.error('Transfer failed:', error);
            throw error;
        } finally {
            setIsLoading(false);
        }
    };
    
    // Approve tokens
    const approve = async (spender, amount) => {
        if (!contract || !tokenInfo) throw new Error('Contract not initialized');
        
        try {
            const amountWei = ethers.utils.parseUnits(amount.toString(), tokenInfo.decimals);
            const tx = await contract.approve(spender, amountWei);
            
            setIsLoading(true);
            await tx.wait();
            
            return tx;
        } catch (error) {
            console.error('Approve failed:', error);
            throw error;
        } finally {
            setIsLoading(false);
        }
    };
    
    // Get allowance
    const getAllowance = async (owner, spender) => {
        if (!contract || !tokenInfo) throw new Error('Contract not initialized');
        
        try {
            const allowance = await contract.allowance(owner, spender);
            return ethers.utils.formatUnits(allowance, tokenInfo.decimals);
        } catch (error) {
            console.error('Failed to get allowance:', error);
            throw error;
        }
    };
    
    // Listen to events
    const listenToTransfers = (callback) => {
        if (!contract) return;
        
        const filter = contract.filters.Transfer();
        contract.on(filter, (from, to, amount, event) => {
            callback({
                from,
                to,
                amount: ethers.utils.formatUnits(amount, tokenInfo?.decimals || 18),
                transactionHash: event.transactionHash,
                blockNumber: event.blockNumber
            });
        });
        
        // Return cleanup function
        return () => contract.removeAllListeners(filter);
    };
    
    useEffect(() => {
        if (contract) {
            loadTokenInfo();
            loadUserBalance();
        }
    }, [contract]);
    
    return {
        tokenInfo,
        userBalance,
        isLoading,
        connectWallet,
        transfer,
        approve,
        getAllowance,
        listenToTransfers,
        loadUserBalance,
        contract
    };
}
```

---

## React Component untuk Token Dashboard

```jsx
// frontend/components/TokenDashboard.jsx
import React, { useState, useEffect } from 'react';
import { useToken } from '../hooks/useToken';

const TokenDashboard = ({ tokenAddress }) => {
    const {
        tokenInfo,
        userBalance,
        isLoading,
        connectWallet,
        transfer,
        approve,
        getAllowance
    } = useToken(tokenAddress);
    
    const [transferTo, setTransferTo] = useState('');
    const [transferAmount, setTransferAmount] = useState('');
    const [approveSpender, setApproveSpender] = useState('');
    const [approveAmount, setApproveAmount] = useState('');
    const [isConnected, setIsConnected] = useState(false);
    
    const handleConnect = async () => {
        try {
            await connectWallet();
            setIsConnected(true);
        } catch (error) {
            alert('Failed to connect wallet: ' + error.message);
        }
    };
    
    const handleTransfer = async (e) => {
        e.preventDefault();
        if (!transferTo || !transferAmount) return;
        
        try {
            const tx = await transfer(transferTo, transferAmount);
            alert(`Transfer successful! Tx: ${tx.hash}`);
            setTransferTo('');
            setTransferAmount('');
        } catch (error) {
            alert('Transfer failed: ' + error.message);
        }
    };
    
    const handleApprove = async (e) => {
        e.preventDefault();
        if (!approveSpender || !approveAmount) return;
        
        try {
            const tx = await approve(approveSpender, approveAmount);
            alert(`Approval successful! Tx: ${tx.hash}`);
            setApproveSpender('');
            setApproveAmount('');
        } catch (error) {
            alert('Approval failed: ' + error.message);
        }
    };
    
    if (!isConnected) {
        return (
            <div className="token-dashboard">
                <h2>Token Dashboard</h2>
                <button onClick={handleConnect}>Connect Wallet</button>
            </div>
        );
    }
    
    if (isLoading && !tokenInfo) {
        return <div>Loading token information...</div>;
    }
    
    return (
        <div className="token-dashboard">
            <h2>Token Dashboard</h2>
            
            {/* Token Info */}
            {tokenInfo && (
                <div className="token-info">
                    <h3>Token Information</h3>
                    <p><strong>Name:</strong> {tokenInfo.name}</p>
                    <p><strong>Symbol:</strong> {tokenInfo.symbol}</p>
                    <p><strong>Total Supply:</strong> {Number(tokenInfo.totalSupply).toLocaleString()} {tokenInfo.symbol}</p>
                    <p><strong>Max Supply:</strong> {Number(tokenInfo.maxSupply).toLocaleString()} {tokenInfo.symbol}</p>
                    <p><strong>Transfer Fee:</strong> {tokenInfo.transferFee}%</p>
                    <p><strong>Status:</strong> {tokenInfo.isPaused ? '⏸️ Paused' : '✅ Active'}</p>
                </div>
            )}
            
            {/* User Balance */}
            <div className="user-balance">
                <h3>Your Balance</h3>
                <p className="balance">
                    {Number(userBalance).toLocaleString()} {tokenInfo?.symbol || 'Tokens'}
                </p>
            </div>
            
            {/* Transfer Form */}
            <div className="transfer-section">
                <h3>Transfer Tokens</h3>
                <form onSubmit={handleTransfer}>
                    <div>
                        <label>To Address:</label>
                        <input
                            type="text"
                            value={transferTo}
                            onChange={(e) => setTransferTo(e.target.value)}
                            placeholder="0x..."
                            required
                        />
                    </div>
                    <div>
                        <label>Amount:</label>
                        <input
                            type="number"
                            value={transferAmount}
                            onChange={(e) => setTransferAmount(e.target.value)}
                            placeholder="0.0"
                            min="0"
                            step="any"
                            required
                        />
                    </div>
                    <button type="submit" disabled={isLoading}>
                        {isLoading ? 'Processing...' : 'Transfer'}
                    </button>
                </form>
            </div>
            
            {/* Approve Form */}
            <div className="approve-section">
                <h3>Approve Tokens</h3>
                <form onSubmit={handleApprove}>
                    <div>
                        <label>Spender Address:</label>
                        <input
                            type="text"
                            value={approveSpender}
                            onChange={(e) => setApproveSpender(e.target.value)}
                            placeholder="0x..."
                            required
                        />
                    </div>
                    <div>
                        <label>Amount:</label>
                        <input
                            type="number"
                            value={approveAmount}
                            onChange={(e) => setApproveAmount(e.target.value)}
                            placeholder="0.0"
                            min="0"
                            step="any"
                            required
                        />
                    </div>
                    <button type="submit" disabled={isLoading}>
                        {isLoading ? 'Processing...' : 'Approve'}
                    </button>
                </form>
            </div>
        </div>
    );
};

export default TokenDashboard;
```

---

## Package.json Scripts untuk Token

```json
{
  "name": "erc20-token-project",
  "version": "1.0.0",
  "scripts": {
    "compile": "npx hardhat compile",
    "test": "npx hardhat test",
    "test:coverage": "npx hardhat coverage",
    "deploy:localhost": "npx hardhat run scripts/deploy-token.js --network localhost",
    "deploy:bsc-testnet": "npx hardhat run scripts/deploy-token.js --network bscTestnet",
    "deploy:sepolia": "npx hardhat run scripts/deploy-token.js --network sepolia",
    "deploy:bsc": "npx hardhat run scripts/deploy-token.js --network bsc",
    "deploy:ethereum": "npx hardhat run scripts/deploy-token.js --network ethereum",
    "interact:localhost": "npx hardhat run scripts/token-interaction.js --network localhost",
    "interact:bsc-testnet": "npx hardhat run scripts/token-interaction.js --network bscTestnet",
    "verify:bsc-testnet": "npx hardhat verify --network bscTestnet",
    "verify:sepolia": "npx hardhat verify --network sepolia",
    "node": "npx hardhat node",
    "clean": "npx hardhat clean"
  }
}
```

---

## Estimasi Biaya Deployment

```javascript
// scripts/estimate-deployment-cost.js
async function main() {
    console.log("💰 Estimating token deployment cost...");
    
    const [deployer] = await ethers.getSigners();
    const balance = await deployer.getBalance();
    
    console.log("📍 Network:", network.name);
    console.log("👤 Deployer:", deployer.address);
    console.log("💳 Current balance:", ethers.utils.formatEther(balance), "ETH");
    
    // Get current gas price
    const gasPrice = await deployer.getGasPrice();
    console.log("⛽ Current gas price:", ethers.utils.formatUnits(gasPrice, "gwei"), "gwei");
    
    // Estimate deployment gas
    const Token = await ethers.getContractFactory("MyProductionToken");
    const deploymentData = Token.getDeployTransaction(
        "Test Token",
        "TEST",
        ethers.utils.parseEther("1000000"),
        deployer.address
    );
    
    const estimatedGas = await deployer.estimateGas(deploymentData);
    const estimatedCost = gasPrice.mul(estimatedGas);
    
    console.log("\n📊 Deployment Estimation:");
    console.log("⛽ Estimated gas:", estimatedGas.toString());
    console.log("💰 Estimated cost:", ethers.utils.formatEther(estimatedCost), "ETH");
    
    // Price estimation in USD (you can use API to get real ETH price)
    const ethPriceUSD = 2000; // Example price
    const costUSD = parseFloat(ethers.utils.formatEther(estimatedCost)) * ethPriceUSD;
    console.log("💵 Estimated cost:", costUSD.toFixed(2), "USD");
    
    // Check if balance is sufficient
    if (balance.gte(estimatedCost)) {
        console.log("✅ Sufficient balance for deployment");
    } else {
        console.log("❌ Insufficient balance for deployment");
        const needed = estimatedCost.sub(balance);
        console.log("💸 Additional needed:", ethers.utils.formatEther(needed), "ETH");
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

## Best Practices untuk Token Development

### 1. **Security Checklist**

```solidity
// ✅ Use OpenZeppelin contracts
// ✅ Implement proper access control
// ✅ Add pause functionality for emergencies
// ✅ Set maximum supply to prevent unlimited minting
// ✅ Use safe math (built-in in Solidity 0.8+)
// ✅ Emit events for all important actions
// ✅ Validate all inputs (require statements)
// ✅ Follow checks-effects-interactions pattern
```

### 2. **Gas Optimization**

```solidity
// ✅ Use uint256 instead of smaller uints for storage
// ✅ Pack struct variables efficiently
// ✅ Use events instead of storage for logging
// ✅ Avoid unnecessary external calls
// ✅ Cache storage reads in memory
// ✅ Use ++i instead of i++ in loops
```

### 3. **Testing Strategy**

```javascript
// ✅ Test all ERC20 standard functions
// ✅ Test edge cases (zero amounts, zero addresses)
// ✅ Test access control (only owner functions)
// ✅ Test events emission
// ✅ Test gas limits and optimization
// ✅ Test integration with other contracts
// ✅ Use fuzz testing for edge cases
```

### 4. **Documentation**

```solidity
/**
 * @title MyProductionToken
 * @dev ERC20 token with advanced features
 * @author Your Name
 * @notice This token includes minting, burning, fees, and pause functionality
 */
contract MyProductionToken {
    /**
     * @dev Transfers tokens with optional fee deduction
     * @param to The address to transfer to
     * @param amount The amount to transfer
     * @return success Whether the transfer was successful
     */
    function transfer(address to, uint256 amount) public override returns (bool) {
        // Implementation
    }
}
```

---

## Token Economics (Tokenomics)

Penting untuk merancang ekonomi token yang sustainable:

### 1. **Supply Distribution**

```solidity
// Contoh distribusi token yang umum
contract TokenomicsExample {
    uint256 public constant TOTAL_SUPPLY = 1000000000 * 10**18; // 1B tokens
    
    // Distribution percentages
    uint256 public constant PUBLIC_SALE = 30;      // 30% - Public sale
    uint256 public constant PRIVATE_SALE = 20;     // 20% - Private investors
    uint256 public constant TEAM = 15;             // 15% - Team (vested)
    uint256 public constant ADVISORS = 5;          // 5% - Advisors (vested)
    uint256 public constant MARKETING = 10;        // 10% - Marketing & partnerships
    uint256 public constant DEVELOPMENT = 10;      // 10% - Development fund
    uint256 public constant LIQUIDITY = 10;        // 10% - DEX liquidity
    
    function distributeTokens() public onlyOwner {
        address publicSaleWallet = 0x...;
        address privateSaleWallet = 0x...;
        address teamWallet = 0x...;
        // etc...
        
        // Distribute tokens according to percentages
        _mint(publicSaleWallet, (TOTAL_SUPPLY * PUBLIC_SALE) / 100);
        _mint(privateSaleWallet, (TOTAL_SUPPLY * PRIVATE_SALE) / 100);
        
        // Team tokens dengan vesting
        createVestingSchedule(
            teamWallet,
            (TOTAL_SUPPLY * TEAM) / 100,
            365 days,  // 1 year cliff
            1460 days, // 4 years total vesting
            true       // revokable
        );
    }
}
```

### 2. **Fee Structure**

```solidity
// Struktur fee yang fleksibel
contract FlexibleFeeToken is ERC20 {
    struct FeeStructure {
        uint256 transferFee;     // Fee untuk transfer biasa
        uint256 buyFee;          // Fee untuk pembelian dari DEX
        uint256 sellFee;         // Fee untuk penjualan ke DEX
        uint256 stakingReward;   // Persentase untuk staking rewards
        uint256 burnAmount;      // Persentase untuk auto-burn
        uint256 treasury;        // Persentase untuk treasury
    }
    
    FeeStructure public fees = FeeStructure({
        transferFee: 100,    // 1%
        buyFee: 200,         // 2%
        sellFee: 300,        // 3%
        stakingReward: 40,   // 40% dari fee untuk staking
        burnAmount: 30,      // 30% dari fee di-burn
        treasury: 30         // 30% dari fee ke treasury
    });
    
    mapping(address => bool) public isDEX;      // DEX addresses
    mapping(address => bool) public isExempt;   // Fee exempt addresses
    
    function _transfer(address from, address to, uint256 amount) internal override {
        uint256 feeAmount = 0;
        
        if (!isExempt[from] && !isExempt[to]) {
            if (isDEX[from]) {
                // Buy transaction
                feeAmount = (amount * fees.buyFee) / 10000;
            } else if (isDEX[to]) {
                // Sell transaction
                feeAmount = (amount * fees.sellFee) / 10000;
            } else {
                // Regular transfer
                feeAmount = (amount * fees.transferFee) / 10000;
            }
            
            if (feeAmount > 0) {
                _distributeFee(from, feeAmount);
                amount -= feeAmount;
            }
        }
        
        super._transfer(from, to, amount);
    }
    
    function _distributeFee(address from, uint256 feeAmount) private {
        uint256 stakingAmount = (feeAmount * fees.stakingReward) / 100;
        uint256 burnAmount = (feeAmount * fees.burnAmount) / 100;
        uint256 treasuryAmount = feeAmount - stakingAmount - burnAmount;
        
        // Transfer ke staking contract
        super._transfer(from, stakingContract, stakingAmount);
        
        // Burn tokens
        _burn(from, burnAmount);
        
        // Transfer ke treasury
        super._transfer(from, treasury, treasuryAmount);
    }
}
```

---

## Anti-Bot dan Anti-Whale Mechanisms

Proteksi dari bot trading dan whale manipulation:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract AntiWhaleToken is ERC20, Ownable {
    // Anti-whale settings
    uint256 public maxTransactionAmount;
    uint256 public maxWalletAmount;
    
    // Anti-bot settings
    uint256 public tradingStartTime;
    uint256 public antiMEVBlocks = 3;
    mapping(address => uint256) private lastTransactionBlock;
    
    // Blacklist untuk bot
    mapping(address => bool) public blacklisted;
    
    // Whitelist untuk bypassing limits
    mapping(address => bool) public whitelisted;
    
    bool public limitsInEffect = true;
    bool public transferDelayEnabled = true;
    
    event BlacklistUpdated(address indexed account, bool status);
    event LimitsRemoved();
    
    constructor(
        string memory name,
        string memory symbol,
        uint256 totalSupply
    ) ERC20(name, symbol) {
        uint256 supply = totalSupply * 10**decimals();
        
        // Set limits (contoh: 1% dari total supply)
        maxTransactionAmount = (supply * 1) / 100;
        maxWalletAmount = (supply * 2) / 100;
        
        // Trading belum dimulai
        tradingStartTime = 0;
        
        // Mint semua token ke owner
        _mint(msg.sender, supply);
        
        // Whitelist owner dan contract
        whitelisted[msg.sender] = true;
        whitelisted[address(this)] = true;
    }
    
    function _transfer(address from, address to, uint256 amount) internal override {
        require(!blacklisted[from] && !blacklisted[to], "Address is blacklisted");
        
        // Cek apakah trading sudah dimulai
        if (tradingStartTime == 0 && to != owner()) {
            require(from == owner(), "Trading not started yet");
        } else if (tradingStartTime > 0) {
            require(block.timestamp >= tradingStartTime, "Trading not started yet");
        }
        
        // Anti-MEV protection
        if (transferDelayEnabled && !whitelisted[from]) {
            require(
                lastTransactionBlock[from] + antiMEVBlocks < block.number,
                "Transfer delay active"
            );
            lastTransactionBlock[from] = block.number;
        }
        
        // Apply limits jika masih aktif
        if (limitsInEffect && !whitelisted[from] && !whitelisted[to]) {
            // Cek max transaction amount
            require(amount <= maxTransactionAmount, "Exceeds max transaction amount");
            
            // Cek max wallet amount (untuk penerima)
            if (to != address(0)) {
                require(
                    balanceOf(to) + amount <= maxWalletAmount,
                    "Exceeds max wallet amount"
                );
            }
        }
        
        super._transfer(from, to, amount);
    }
    
    // Owner functions
    function startTrading() external onlyOwner {
        require(tradingStartTime == 0, "Trading already started");
        tradingStartTime = block.timestamp;
    }
    
    function updateBlacklist(address account, bool status) external onlyOwner {
        blacklisted[account] = status;
        emit BlacklistUpdated(account, status);
    }
    
    function batchUpdateBlacklist(address[] calldata accounts, bool status) external onlyOwner {
        for (uint256 i = 0; i < accounts.length; i++) {
            blacklisted[accounts[i]] = status;
            emit BlacklistUpdated(accounts[i], status);
        }
    }
    
    function updateWhitelist(address account, bool status) external onlyOwner {
        whitelisted[account] = status;
    }
    
    function removeLimits() external onlyOwner {
        limitsInEffect = false;
        transferDelayEnabled = false;
        emit LimitsRemoved();
    }
    
    function updateLimits(uint256 maxTxAmount, uint256 maxWalletAmt) external onlyOwner {
        require(maxTxAmount >= (totalSupply() * 1) / 1000, "Max tx too low"); // min 0.1%
        require(maxWalletAmt >= (totalSupply() * 5) / 1000, "Max wallet too low"); // min 0.5%
        
        maxTransactionAmount = maxTxAmount;
        maxWalletAmount = maxWalletAmt;
    }
    
    function setAntiMEVBlocks(uint256 blocks) external onlyOwner {
        require(blocks <= 10, "Too many blocks");
        antiMEVBlocks = blocks;
    }
    
    // Emergency function untuk menghapus stuck ETH
    function rescueETH() external onlyOwner {
        payable(owner()).transfer(address(this).balance);
    }
    
    // Emergency function untuk menghapus stuck tokens
    function rescueTokens(address tokenAddress) external onlyOwner {
        IERC20 token = IERC20(tokenAddress);
        token.transfer(owner(), token.balanceOf(address(this)));
    }
}
```

---

## Token dengan Staking Mechanism

Implementasi staking langsung di dalam token:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract StakingToken is ERC20, Ownable, ReentrancyGuard {
    struct StakeInfo {
        uint256 amount;          // Jumlah token yang di-stake
        uint256 startTime;       // Kapan mulai stake
        uint256 lastClaim;       // Kapan terakhir claim reward
        uint256 totalClaimed;    // Total reward yang sudah di-claim
    }
    
    mapping(address => StakeInfo) public stakes;
    
    // Staking parameters
    uint256 public stakingAPY = 1200; // 12% APY (dalam basis points)
    uint256 public minStakingPeriod = 7 days;
    uint256 public totalStaked;
    uint256 public rewardPool;
    
    // Events
    event Staked(address indexed user, uint256 amount);
    event Unstaked(address indexed user, uint256 amount);
    event RewardClaimed(address indexed user, uint256 reward);
    event RewardPoolFunded(uint256 amount);
    
    constructor(
        string memory name,
        string memory symbol,
        uint256 totalSupply
    ) ERC20(name, symbol) {
        _mint(msg.sender, totalSupply * 10**decimals());
    }
    
    // Stake tokens
    function stake(uint256 amount) external nonReentrant {
        require(amount > 0, "Amount must be greater than 0");
        require(balanceOf(msg.sender) >= amount, "Insufficient balance");
        
        StakeInfo storage userStake = stakes[msg.sender];
        
        // Claim pending rewards jika ada
        if (userStake.amount > 0) {
            _claimRewards(msg.sender);
        }
        
        // Transfer tokens ke contract
        _transfer(msg.sender, address(this), amount);
        
        // Update staking info
        userStake.amount += amount;
        userStake.startTime = block.timestamp;
        userStake.lastClaim = block.timestamp;
        
        totalStaked += amount;
        
        emit Staked(msg.sender, amount);
    }
    
    // Unstake tokens
    function unstake(uint256 amount) external nonReentrant {
        StakeInfo storage userStake = stakes[msg.sender];
        require(userStake.amount >= amount, "Insufficient staked amount");
        require(
            block.timestamp >= userStake.startTime + minStakingPeriod,
            "Minimum staking period not met"
        );
        
        // Claim pending rewards
        _claimRewards(msg.sender);
        
        // Update staking info
        userStake.amount -= amount;
        totalStaked -= amount;
        
        // Transfer tokens back
        _transfer(address(this), msg.sender, amount);
        
        emit Unstaked(msg.sender, amount);
    }
    
    // Claim rewards tanpa unstake
    function claimRewards() external nonReentrant {
        _claimRewards(msg.sender);
    }
    
    // Internal function untuk claim rewards
    function _claimRewards(address user) internal {
        StakeInfo storage userStake = stakes[user];
        
        if (userStake.amount == 0) return;
        
        uint256 reward = calculateReward(user);
        if (reward > 0 && reward <= rewardPool) {
            userStake.lastClaim = block.timestamp;
            userStake.totalClaimed += reward;
            rewardPool -= reward;
            
            // Mint reward tokens
            _mint(user, reward);
            
            emit RewardClaimed(user, reward);
        }
    }
    
    // Calculate pending rewards
    function calculateReward(address user) public view returns (uint256) {
        StakeInfo memory userStake = stakes[user];
        
        if (userStake.amount == 0) return 0;
        
        uint256 stakingDuration = block.timestamp - userStake.lastClaim;
        uint256 annualReward = (userStake.amount * stakingAPY) / 10000;
        uint256 reward = (annualReward * stakingDuration) / 365 days;
        
        return reward;
    }
    
    // Get staking info
    function getStakingInfo(address user) external view returns (
        uint256 stakedAmount,
        uint256 stakingStartTime,
        uint256 pendingReward,
        uint256 totalClaimedReward
    ) {
        StakeInfo memory userStake = stakes[user];
        return (
            userStake.amount,
            userStake.startTime,
            calculateReward(user),
            userStake.totalClaimed
        );
    }
    
    // Owner functions
    function setStakingAPY(uint256 newAPY) external onlyOwner {
        require(newAPY <= 5000, "APY too high"); // max 50%
        stakingAPY = newAPY;
    }
    
    function setMinStakingPeriod(uint256 newPeriod) external onlyOwner {
        require(newPeriod <= 30 days, "Period too long");
        minStakingPeriod = newPeriod;
    }
    
    function fundRewardPool(uint256 amount) external onlyOwner {
        require(balanceOf(msg.sender) >= amount, "Insufficient balance");
        _transfer(msg.sender, address(this), amount);
        rewardPool += amount;
        
        emit RewardPoolFunded(amount);
    }
    
    // Emergency function
    function emergencyWithdraw() external {
        StakeInfo storage userStake = stakes[msg.sender];
        require(userStake.amount > 0, "No staked tokens");
        
        uint256 amount = userStake.amount;
        userStake.amount = 0;
        totalStaked -= amount;
        
        _transfer(address(this), msg.sender, amount);
        
        emit Unstaked(msg.sender, amount);
    }
}
```

---

## Deployment Guide untuk Multiple Networks

### Hardhat Config untuk Multiple Networks

```javascript
// hardhat.config.js
require("@nomiclabs/hardhat-waffle");
require("@nomiclabs/hardhat-etherscan");
require("hardhat-gas-reporter");
require("solidity-coverage");
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
    // Local development
    localhost: {
      url: "http://127.0.0.1:8545"
    },
    
    // Testnets
    sepolia: {
      url: `https://sepolia.infura.io/v3/${process.env.INFURA_PROJECT_ID}`,
      accounts: [process.env.PRIVATE_KEY],
      gasPrice: 20000000000, // 20 gwei
    },
    bscTestnet: {
      url: "https://data-seed-prebsc-1-s1.binance.org:8545",
      accounts: [process.env.PRIVATE_KEY],
      gasPrice: 10000000000, // 10 gwei
    },
    mumbai: {
      url: `https://polygon-mumbai.infura.io/v3/${process.env.INFURA_PROJECT_ID}`,
      accounts: [process.env.PRIVATE_KEY],
      gasPrice: 35000000000, // 35 gwei
    },
    
    // Mainnets
    ethereum: {
      url: `https://mainnet.infura.io/v3/${process.env.INFURA_PROJECT_ID}`,
      accounts: [process.env.PRIVATE_KEY],
      gasPrice: 25000000000, // 25 gwei
    },
    bsc: {
      url: "https://bsc-dataseed1.binance.org",
      accounts: [process.env.PRIVATE_KEY],
      gasPrice: 5000000000, // 5 gwei
    },
    polygon: {
      url: `https://polygon-mainnet.infura.io/v3/${process.env.INFURA_PROJECT_ID}`,
      accounts: [process.env.PRIVATE_KEY],
      gasPrice: 35000000000, // 35 gwei
    }
  },
  etherscan: {
    apiKey: {
      mainnet: process.env.ETHERSCAN_API_KEY,
      sepolia: process.env.ETHERSCAN_API_KEY,
      bsc: process.env.BSCSCAN_API_KEY,
      bscTestnet: process.env.BSCSCAN_API_KEY,
      polygon: process.env.POLYGONSCAN_API_KEY,
      polygonMumbai: process.env.POLYGONSCAN_API_KEY
    }
  },
  gasReporter: {
    enabled: process.env.REPORT_GAS !== undefined,
    currency: "USD"
  }
};
```

### Environment Variables (.env)

```bash
# .env
PRIVATE_KEY=your_private_key_here
INFURA_PROJECT_ID=your_infura_project_id
ETHERSCAN_API_KEY=your_etherscan_api_key
BSCSCAN_API_KEY=your_bscscan_api_key
POLYGONSCAN_API_KEY=your_polygonscan_api_key

# Token Configuration
TOKEN_NAME="My Awesome Token"
TOKEN_SYMBOL="MAT"
INITIAL_SUPPLY=1000000
MAX_SUPPLY=1000000000

# Deployment Settings
VERIFY_CONTRACTS=true
SAVE_DEPLOYMENTS=true
```

### Deployment Script dengan Multi-Network Support

```javascript
// scripts/deploy-multi-network.js
const { ethers, network } = require("hardhat");
const fs = require("fs");
const path = require("path");

async function main() {
    console.log(`🚀 Deploying to ${network.name}...`);
    
    const [deployer] = await ethers.getSigners();
    console.log("📝 Deploying with account:", deployer.address);
    
    const balance = await deployer.getBalance();
    console.log("💰 Account balance:", ethers.utils.formatEther(balance), "ETH");
    
    // Load configuration
    const config = getNetworkConfig(network.name);
    console.log("🔧 Using configuration:", config);
    
    // Deploy token
    const Token = await ethers.getContractFactory("MyProductionToken");
    const token = await Token.deploy(
        config.name,
        config.symbol,
        ethers.utils.parseEther(config.initialSupply.toString()),
        deployer.address
    );
    
    await token.deployed();
    console.log("✅ Token deployed to:", token.address);
    
    // Save deployment info
    await saveDeployment(network.name, token, config);
    
    // Verify contract if on supported network
    if (shouldVerify(network.name)) {
        console.log("🔍 Verifying contract...");
        await verifyContract(token.address, [
            config.name,
            config.symbol,
            ethers.utils.parseEther(config.initialSupply.toString()),
            deployer.address
        ]);
    }
    
    // Post-deployment setup
    await postDeploymentSetup(token, config);
    
    console.log("🎉 Deployment completed successfully!");
}

function getNetworkConfig(networkName) {
    const configs = {
        localhost: {
            name: "Test Token",
            symbol: "TEST",
            initialSupply: 1000000,
            maxSupply: 10000000,
            transferFee: 0
        },
        sepolia: {
            name: "My Token Testnet",
            symbol: "MTT",
            initialSupply: 100000,
            maxSupply: 1000000,
            transferFee: 100
        },
        bscTestnet: {
            name: "My Token BSC Testnet",
            symbol: "MTBSC",
            initialSupply: 100000,
            maxSupply: 1000000,
            transferFee: 50
        },
        ethereum: {
            name: process.env.TOKEN_NAME || "My Production Token",
            symbol: process.env.TOKEN_SYMBOL || "MPT",
            initialSupply: parseInt(process.env.INITIAL_SUPPLY) || 1000000,
            maxSupply: parseInt(process.env.MAX_SUPPLY) || 1000000000,
            transferFee: 200
        },
        bsc: {
            name: process.env.TOKEN_NAME || "My BSC Token",
            symbol: process.env.TOKEN_SYMBOL || "MBT",
            initialSupply: parseInt(process.env.INITIAL_SUPPLY) || 1000000,
            maxSupply: parseInt(process.env.MAX_SUPPLY) || 1000000000,
            transferFee: 100
        }
    };
    
    return configs[networkName] || configs.localhost;
}

async function saveDeployment(networkName, token, config) {
    const deploymentInfo = {
        network: networkName,
        address: token.address,
        deployer: await token.signer.getAddress(),
        deploymentTime: new Date().toISOString(),
        transactionHash: token.deployTransaction.hash,
        blockNumber: token.deployTransaction.blockNumber,
        gasUsed: (await token.deployTransaction.wait()).gasUsed.toString(),
        config: config
    };
    
    const deploymentsDir = path.join(__dirname, "../deployments");
    if (!fs.existsSync(deploymentsDir)) {
        fs.mkdirSync(deploymentsDir);
    }
    
    fs.writeFileSync(
        path.join(deploymentsDir, `${networkName}.json`),
        JSON.stringify(deploymentInfo, null, 2)
    );
    
    console.log(`💾 Deployment info saved to deployments/${networkName}.json`);
}

function shouldVerify(networkName) {
    const supportedNetworks = ["sepolia", "ethereum", "bsc", "bscTestnet", "polygon", "mumbai"];
    return supportedNetworks.includes(networkName) && process.env.VERIFY_CONTRACTS === "true";
}

async function verifyContract(address, constructorArguments) {
    try {
        await run("verify:verify", {
            address: address,
            constructorArguments: constructorArguments,
        });
        console.log("✅ Contract verified successfully");
    } catch (error) {
        if (error.message.toLowerCase().includes("already verified")) {
            console.log("✅ Contract is already verified");
        } else {
            console.error("❌ Verification failed:", error.message);
        }
    }
}

async function postDeploymentSetup(token, config) {
    console.log("\n🔧 Running post-deployment setup...");
    
    try {
        // Set transfer fee if specified
        if (config.transferFee > 0) {
            const tx = await token.setTransferFee(config.transferFee);
            await tx.wait();
            console.log(`✅ Transfer fee set to ${config.transferFee / 100}%`);
        }
        
        // Add any additional setup here
        console.log("✅ Post-deployment setup completed");
        
    } catch (error) {
        console.error("❌ Post-deployment setup failed:", error.message);
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

## Kesimpulan Bagian 2

Selamat! Anda telah menguasai pengembangan token ERC20 secara komprehensif:

### **Yang Sudah Dipelajari:**

✅ **Memahami standar ERC20** dan fungsi-fungsi wajibnya  
✅ **Membuat token basic** dari nol dengan implementasi manual  
✅ **Token advanced** dengan fitur mint, burn, fee, pause, blacklist  
✅ **Vesting system** untuk lock dan release token bertahap  
✅ **OpenZeppelin integration** untuk production-ready tokens  
✅ **Anti-bot dan anti-whale** mechanisms untuk proteksi  
✅ **Staking mechanism** built-in di dalam token  
✅ **Tokenomics design** untuk ekonomi token yang sustainable  
✅ **Comprehensive testing** dengan berbagai skenario  
✅ **Multi-network deployment** untuk berbagai blockchain  
✅ **Frontend integration** dengan React dan Web3  
✅ **Best practices** untuk security dan gas optimization  

### **Fitur-Fitur yang Telah Dikuasai:**

🎯 **Core ERC20**: Transfer, approve, allowance  
🎯 **Advanced Features**: Mint, burn, pause, blacklist  
🎯 **Economic Models**: Fee system, vesting, staking  
🎯 **Security**: Anti-bot, anti-whale, access control  
🎯 **Development**: Testing, deployment, verification  
🎯 **Integration**: Frontend, multi-network support  

### **Real-World Applications:**

- **Utility Tokens**: Untuk DApps dan platform  
- **Governance Tokens**: Untuk DAO voting  
- **Reward Tokens**: Untuk loyalty dan incentive programs  
- **Stablecoins**: Dengan pegging mechanisms  
- **Meme Tokens**: Dengan community features  
- **DeFi Tokens**: Untuk yield farming dan liquidity mining  

### **Next Level:**

Dengan pengetahuan ini, Anda sudah bisa:
- Membuat cryptocurrency profesional untuk startup  
- Mengintegrasikan token dengan DeFi protocols  
- Membangun ekonomi token yang kompleks  
- Deploy ke multiple blockchains  
- Membuat token dengan fitur advanced seperti staking dan vesting  

Anda sekarang memiliki foundation yang solid untuk menjadi **Blockchain Developer** yang dapat membuat token berkualitas production! 🚀💰
