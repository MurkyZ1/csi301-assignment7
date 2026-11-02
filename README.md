# Bank DApp (Infura + Web3.js)

เว็บ DApp สำหรับใช้งาน Smart Contract ระบบธนาคาร เชื่อมต่อบล็อกเชน Sepolia ผ่าน **Infura** ด้วย **Web3.js**

- **รหัสนักศึกษา:** 66102316
- **ชื่อ-สกุล:** คิรากร สุวรรณโกสุม
- **เว็บไซต์:** https://murkyz1.github.io/csi301_assignment6/
- **Contract address:** `0xd943ABb1cE2b884FE06Fa3A2c7F6723F0724E1Ea`
- **Network:** Sepolia Testnet

## ฟังก์ชันที่รองรับ

| ฟังก์ชัน | คำอธิบาย | Web3.js | ค่า Gas |
|---|---|---|---|
| Connect | เชื่อมต่อ Infura และ Smart Contract | `HttpProvider`, `Contract` | ไม่เสีย |
| Check Balance | ดูยอดเงินของบัญชี | `methods.checkBlance().call()` | ไม่เสีย (view) |
| Deposit | ฝาก ETH เข้า Contract | `methods.deposit().send({value})` | เสีย |
| Withdraw | ถอน ETH กลับเข้ากระเป๋า | `methods.withdraw(amount).send()` | เสีย |
| Disconnect | ออกจากระบบ ล้างบัญชีออกจากหน้าเว็บ | `wallet.clear()` | ไม่เสีย |

หน้าเว็บแสดงรหัสนักศึกษาและชื่อ-สกุลที่ส่วนหัว มีกล่องแจ้งสถานะธุรกรรม (Pending / Success / Error) และรายการฝาก-ถอนในเซสชัน

## การทำงาน (4 ขั้นตอนตามสไลด์)

1. **Configure** ตั้งค่า `INFURA_URL` และ `PRIVATE_KEY`
2. **Connect** สร้าง Provider: `new Web3(new Web3.providers.HttpProvider(INFURA_URL))`
3. **Instantiate** สร้าง Contract: `new web3.eth.Contract(abi, CONTRACT_ADDRESS)`
4. **Authenticate** แปลง key เป็นบัญชี: `privateKeyToAccount` แล้ว `wallet.add`

```
GitHub Pages (Frontend) → Web3.js → Infura (RPC) → Bank Smart Contract (Sepolia)
```

- อ่านข้อมูลใช้ `methods.call()` ฟรี ทันที ไม่ต้องใช้ key
- เขียนข้อมูลใช้ `methods.send()` ต้องเซ็นด้วยบัญชีจาก `PRIVATE_KEY` และเสีย Gas

> โค้ดเชื่อมต่อ MetaMask ยังอยู่ในไฟล์ `index.html` เป็น comment (ค้นหา `MetaMask (ปิดไว้)`) เปิดใช้ได้โดยเอา comment ออก

## เทคโนโลยีที่ใช้

- HTML / CSS / JavaScript (ไฟล์เดียว `index.html`)
- [Web3.js 1.10](https://web3js.readthedocs.io/)
- [Infura](https://www.infura.io/) เป็น RPC / Node Provider
- Solidity ^0.8.0 (`bank.sol`)
- GitHub Pages สำหรับ Hosting

## วิธีติดตั้งและรันเอง

### 1. Deploy Smart Contract
1. เปิด [Remix IDE](https://remix.ethereum.org) แล้ววางโค้ด `bank.sol`
2. Compile ด้วย Solidity 0.8.x
3. แท็บ Deploy เลือก **Injected Provider - MetaMask** (บน Sepolia) แล้วกด Deploy
4. คัดลอก address ของ Contract

### 2. เตรียม Infura และกระเป๋าทดสอบ
1. สมัครที่ infura.io สร้าง API key แล้วคัดลอก Sepolia endpoint
2. สร้างบัญชีใหม่ใน MetaMask สำหรับทดสอบโดยเฉพาะ แล้วขอ Sepolia ETH จาก faucet
3. คัดลอก private key ของบัญชีทดสอบนั้น

### 3. ตั้งค่า `index.html`
```js
const STUDENT_ID   = "66102316";
const STUDENT_NAME = "คิรากร สุวรรณโกสุม";
const CONTRACT_ADDRESS = "0x...";   // address ที่ได้จากการ Deploy
const INFURA_URL   = "https://sepolia.infura.io/v3/YOUR_INFURA_PROJECT_ID";
const PRIVATE_KEY  = "YOUR_PRIVATE_KEY";   // กระเป๋าทดสอบเท่านั้น
```

### 4. ทดสอบในเครื่อง
```bash
python -m http.server 8000
# เปิด http://localhost:8000
```

### 5. นำขึ้น GitHub Pages
1. Push `index.html`, `bank.sol`, `README.md` ขึ้น repository
2. Settings → Pages → Deploy from a branch → `main` → `/ (root)` → Save
3. เปิด URL ของเว็บ แล้วนำส่งในระบบ dlearning

## Smart Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Bank {
    mapping (address => uint) _balances;
    event Deposit(address indexed owner, uint amount);
    event Withdraw(address indexed owner, uint amount);

    function deposit() public payable {
        require(msg.value > 0, "Deposit money is zero");
        _balances[msg.sender] += msg.value;
        emit Deposit(msg.sender, msg.value);
    }

    function withdraw(uint amount) public {
        require(amount > 0 && amount <= _balances[msg.sender], "not enough money");
        payable(msg.sender).transfer(amount);
        _balances[msg.sender] -= amount;
        emit Withdraw(msg.sender, amount);
    }

    function checkBlance() public view returns (uint balance) {
        return _balances[msg.sender];
    }
}
```

## ข้อควรระวังด้านความปลอดภัย

- repository นี้เป็น **Public** ค่า `PRIVATE_KEY` ในไฟล์ใครก็เห็นได้ จึงใช้ได้เฉพาะ **กระเป๋าทดสอบบน Sepolia** ที่ไม่มีเงินจริงเท่านั้น
- ห้ามนำ private key ของกระเป๋าหลักหรือกระเป๋าที่มีทรัพย์สินจริงมาใส่เด็ดขาด
- ควรจำกัด Infura API key ให้ใช้ได้เฉพาะ Sepolia ใน Dashboard
- Contract ใช้ตามโจทย์ที่กำหนด โดยโอนเงินก่อนหักยอด ในระบบจริงควรหักยอดก่อนแล้วค่อยโอน เพื่อป้องกัน reentrancy
