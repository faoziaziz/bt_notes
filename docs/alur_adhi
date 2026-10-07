# Bluetooth Data Capture Gateway

## Pengumpulan dan Forwarding Data ESC/POS dari Mesin POS Berbasis MCU MKL82

**Document Type:** Technical Working Document
**Status:** Draft / Engineering Discussion
**Target Device:** Bluetooth Data Capture Gateway
**Source Device:** POS Terminal berbasis MCU MKL82
**Primary Data Format:** ESC/POS / ASCII
**Communication Uplink:** GSM / Wi-Fi via AT Command
**Preferred Protocol:** HTTP POST / HTTP GET
**Fallback Protocol:** TCP/IP
**Document Version:** 0.1

---

# 1. Tujuan

Dokumen ini mendefinisikan konsep dan kebutuhan teknis untuk sebuah perangkat **Bluetooth Data Capture Gateway** yang digunakan untuk mengambil data yang dikirim oleh mesin POS/printer berbasis **MCU MKL82**, kemudian meneruskan data tersebut ke server melalui jaringan **GSM atau Wi-Fi**.

Prinsip utama sistem adalah:

> **POS → Bluetooth → Data Capture Gateway → GSM/Wi-Fi → Internet → Server**

Gateway tidak ditujukan untuk mengubah isi transaksi secara signifikan. Gateway bertindak sebagai **data collector dan communication gateway** yang menangkap data printer dalam format ESC/POS/ASCII kemudian mengirimkannya ke backend.

---

# 2. Latar Belakang

Mesin POS menggunakan MCU MKL82 untuk mengendalikan printer thermal POS. Pada umumnya komunikasi antara MCU POS dan printer menggunakan data berbasis **ESC/POS**, yang terdiri dari:

* ASCII text
* ESC command
* printer formatting command
* line feed
* carriage return
* barcode command
* QR code command
* alignment command
* font/size command
* cut-paper command
* dan command printer lainnya.

Data tersebut pada dasarnya sudah merepresentasikan informasi transaksi yang akan dicetak.

Daripada membuat sistem integrasi POS baru pada level aplikasi, pendekatan yang digunakan adalah mengambil **printer data stream** tersebut sebelum atau ketika dikirim ke printer.

Gateway kemudian melakukan forwarding data tersebut ke server.

Dengan demikian, sistem dapat memperoleh data transaksi tanpa perlu melakukan perubahan besar pada software POS.

---

# 3. Konsep Sistem

Arsitektur dasar:

```text
                    POS MACHINE
                +------------------+
                |      MKL82       |
                |                  |
                | POS Application  |
                +--------+---------+
                         |
                         | ESC/POS
                         | ASCII + ESC Commands
                         v
                +------------------+
                | Printer Interface|
                +--------+---------+
                         |
                         | Bluetooth
                         v
              +----------------------+
              | Bluetooth Data       |
              | Capture Gateway      |
              |                      |
              | MCU                  |
              | Bluetooth            |
              | Buffer               |
              | Protocol Handler     |
              +----------+-----------+
                         |
               +---------+---------+
               |                   |
               v                   v
            Wi-Fi                 GSM
               |                   |
               +---------+---------+
                         |
                         | Internet
                         v
                +------------------+
                | Backend Server   |
                | HTTP / TCP Server|
                +------------------+
```

---

# 4. Prinsip Operasi

Gateway bekerja dalam beberapa tahap:

### Step 1 – POS menghasilkan data printer

MCU MKL82 menghasilkan data ESC/POS.

Contoh:

```text
ESC @
ESC a 1
"      TOKO ABC"
LF
"--------------------------"
LF
"INDOMIE       2 x 3000"
LF
"TOTAL             6000"
LF
GS V 0
```

Secara aktual data yang dikirim berupa byte stream, bukan string ASCII murni.

Contoh:

```text
1B 40
1B 61 01
20 20 20 20 54 4F 4B 4F
...
0A
...
1D 56 00
```

---

# 5. Bluetooth Data Capture

Gateway menerima data melalui Bluetooth.

Bluetooth interface dapat digunakan sebagai:

* Bluetooth Classic
* Bluetooth Low Energy
* Bluetooth SPP-like transport
* atau proprietary Bluetooth protocol

Untuk kebutuhan **streaming data ESC/POS**, Bluetooth Classic dengan pendekatan serial/SPP secara konsep merupakan opsi yang sederhana karena model komunikasinya menyerupai UART.

Namun pemilihan Bluetooth controller/module harus ditentukan berdasarkan:

* kemampuan MCU
* operating mode
* pairing requirement
* security
* throughput
* availability
* cost
* dukungan Linux/Android/POS
* kemampuan melakukan connection tanpa mengganggu printer.

---

# 6. Data yang Harus Dipertahankan

Gateway sebaiknya bekerja dalam mode:

> **RAW DATA FORWARDING**

Artinya gateway tidak melakukan parsing ESC/POS pada tahap awal.

Contoh:

```text
POS
 |
 | 1B 40 1B 61 01 54 4F 4B 4F 20 41 42 43 0A ...
 |
 v
Gateway
 |
 | SAME BYTE STREAM
 |
 v
Backend
```

Dengan pendekatan ini gateway tidak bergantung pada jenis printer.

Data:

```text
ASCII
ESC
GS
FS
CR
LF
Binary command
Barcode
QR
Printer control command
```

tetap dipertahankan.

---

# 7. Alasan Menggunakan RAW ESC/POS

Mengirim raw ESC/POS mempunyai beberapa keuntungan.

### 7.1 Tidak perlu mengetahui format transaksi

Gateway tidak perlu memahami:

```text
harga
produk
subtotal
total
pajak
diskon
payment method
```

Gateway hanya menangkap data.

### 7.2 Kompatibilitas printer lebih tinggi

Printer yang menggunakan command ESC/POS tetap dapat digunakan.

### 7.3 Firmware lebih sederhana

Tidak diperlukan parser kompleks pada versi awal.

### 7.4 Debugging lebih mudah

Data dapat dibandingkan:

```text
POS TX
vs
Gateway RX
vs
Gateway TX
vs
Server RX
```

---

# 8. Uplink Communication

Gateway memiliki dua kemungkinan koneksi:

```text
                    +----------------+
                    | Gateway        |
                    +-------+--------+
                            |
                  +---------+---------+
                  |                   |
                  v                   v
                Wi-Fi                GSM
```

Keduanya dikontrol menggunakan AT Command.

---

# 9. Wi-Fi Mode

Pada Wi-Fi mode, gateway dapat menggunakan Wi-Fi module yang memiliki AT Command interface.

Contoh konsep:

```text
MCU
 |
 | UART
 |
 v
Wi-Fi Module
 |
 | TCP/IP
 |
 v
Internet
```

MCU memberikan command:

```text
AT+...
```

kepada module.

Module bertanggung jawab terhadap:

* Wi-Fi association
* DHCP
* TCP/IP
* DNS
* HTTP
* TLS jika didukung
* connection management

---

# 10. GSM Mode

Untuk GSM/LTE:

```text
MCU
 |
 | UART
 |
 v
Cellular Module
 |
 | LTE
 |
 v
Internet
```

AT command dapat digunakan untuk:

```text
SIM initialization
network registration
PDP context
IP connection
HTTP
TCP
TLS
```

Contoh alur konseptual:

```text
AT
AT+CPIN?
AT+CSQ
AT+CREG?
AT+CGATT=1
AT+CGDCONT=...
```

Command aktual tergantung modem yang digunakan.

---

# 11. Prioritas Protocol

Urutan implementasi yang direkomendasikan:

### Priority 1

**HTTP POST**

```text
POST /api/v1/transaction
Content-Type: application/octet-stream

<RAW ESC/POS DATA>
```

Ini adalah pilihan utama.

---

### Priority 2

**HTTP POST dengan Base64**

Jika backend atau modem sulit menangani binary payload secara langsung:

```json
{
    "device_id": "POS001",
    "timestamp": "2026-10-07T15:30:00+07:00",
    "data": "G0BBGmE..."
}
```

Data ESC/POS diubah menjadi Base64.

Kekurangan:

* ukuran data bertambah sekitar 33%
* membutuhkan encoding/decoding.

---

### Priority 3

**HTTP GET**

GET hanya direkomendasikan untuk:

* konfigurasi
* device status
* heartbeat
* polling
* command retrieval

Tidak direkomendasikan untuk data printer berukuran besar.

Contoh:

```http
GET /api/v1/device/POS001/config
```

---

### Priority 4

**Raw TCP/IP**

Jika HTTP terlalu berat atau modem memiliki keterbatasan:

```text
Gateway
   |
   | TCP
   |
   v
Server
```

Format packet dapat dibuat sendiri.

---

# 12. Recommended HTTP POST

Format awal yang direkomendasikan:

```http
POST /api/v1/pos/print-data
Content-Type: application/octet-stream
X-Device-ID: POS001
X-Sequence: 12345
X-Timestamp: 2026-10-07T15:30:00+07:00
```

Body:

```text
<RAW ESC/POS DATA>
```

Contoh:

```text
+---------------------------+
| HTTP Header               |
+---------------------------+
| Device ID                 |
+---------------------------+
| Sequence Number           |
+---------------------------+
| Timestamp                 |
+---------------------------+
| RAW ESC/POS DATA          |
+---------------------------+
```

---

# 13. Alternatif JSON Protocol

Jika backend membutuhkan metadata yang lebih mudah diproses:

```json
{
    "device_id": "POS001",
    "sequence": 12345,
    "timestamp": "2026-10-07T15:30:00+07:00",
    "encoding": "base64",
    "data": "..."
}
```

Namun untuk versi pertama, **binary HTTP POST lebih direkomendasikan** jika modem mendukungnya.

---

# 14. TCP Protocol

Jika HTTP tidak memungkinkan, dapat dibuat proprietary TCP protocol.

Contoh:

```text
+--------+--------+--------+--------+-------------+
| MAGIC  | VERSION| TYPE   | LENGTH | PAYLOAD     |
+--------+--------+--------+--------+-------------+
| 2 byte | 1 byte | 1 byte | 2 byte | N bytes     |
+--------+--------+--------+--------+-------------+
```

Contoh:

```text
MAGIC      = 0xAA55
VERSION    = 0x01
TYPE       = 0x01
LENGTH     = 256
PAYLOAD    = ESC/POS DATA
```

Packet:

```text
AA 55 01 01 01 00 <ESC/POS DATA>
```

---

# 15. Packet Fragmentation

Karena data printer dapat lebih besar daripada ukuran packet jaringan, gateway harus memiliki mekanisme fragmentation.

Misalnya:

```text
Transaction #123

Chunk 1
Chunk 2
Chunk 3
Chunk 4
```

Format:

```text
DEVICE_ID
TRANSACTION_ID
CHUNK_ID
TOTAL_CHUNK
PAYLOAD
CRC
```

Contoh:

```text
POS001
TX12345
CHUNK 1/4
DATA
CRC
```

---

# 16. Buffering

Gateway harus memiliki buffer antara Bluetooth dan network.

```text
Bluetooth RX
     |
     v
+------------+
| RX Buffer  |
+------------+
     |
     v
+------------+
| Queue      |
+------------+
     |
     v
Network TX
```

Hal ini penting karena:

```text
Bluetooth speed != GSM speed
```

dan:

```text
printer data arrival != network availability
```

---

# 17. Store-and-Forward

Salah satu fitur penting adalah kemampuan menyimpan data ketika koneksi Internet tidak tersedia.

Contoh:

```text
POS
 |
 | ESC/POS
 v
Gateway
 |
 | NO INTERNET
 v
Flash / External Storage
 |
 | connection restored
 v
Server
```

Gateway tidak boleh kehilangan transaksi hanya karena:

* Wi-Fi disconnect
* GSM loss
* server timeout
* HTTP failure
* modem reboot.

---

# 18. Transaction Queue

Contoh queue:

```text
+-------------------------+
| Transaction Queue       |
+-------------------------+
| TX000001 - SENT         |
| TX000002 - SENT         |
| TX000003 - PENDING      |
| TX000004 - PENDING      |
| TX000005 - PENDING      |
+-------------------------+
```

Setiap transaction memiliki:

```text
transaction_id
timestamp
payload
payload_length
retry_count
status
```

---

# 19. ACK Mechanism

Server harus memberikan response yang jelas.

Contoh:

```json
{
    "status": "OK",
    "transaction_id": "TX000123"
}
```

Gateway hanya menghapus data dari queue jika server memberikan ACK.

Flow:

```text
Gateway
   |
   | POST TX123
   v
Server
   |
   | 200 OK
   | TX123 received
   v
Gateway
   |
   | Delete TX123
   v
Queue
```

Jika:

```text
HTTP 500
timeout
connection lost
```

maka:

```text
KEEP DATA
RETRY
```

---

# 20. Retry Mechanism

Retry tidak boleh dilakukan terus menerus tanpa delay.

Contoh:

```text
Retry 1 : 5 sec
Retry 2 : 15 sec
Retry 3 : 30 sec
Retry 4 : 60 sec
Retry 5 : 5 min
```

Dapat digunakan exponential backoff.

---

# 21. Device Identification

Setiap gateway harus mempunyai ID unik.

Contoh:

```text
DEVICE_ID = BDC000001
```

atau:

```text
MAC Address
IMEI
Serial Number
UUID
```

Recommended:

```text
Device Serial Number
+
Hardware UUID
```

---

# 22. Security

Jika menggunakan Internet, komunikasi sebaiknya menggunakan:

```text
HTTPS
```

bukan HTTP plain text.

Recommended:

```text
Gateway
   |
   | TLS
   v
HTTPS Server
```

Authentication dapat menggunakan:

```text
API Key
Bearer Token
Device Certificate
HMAC
```

Untuk prototype:

```text
Device ID
+
API Key
```

sudah cukup sebagai tahap awal.

Untuk production:

```text
TLS
+
per-device credential
+
server authentication
```

lebih baik.

---

# 23. Heartbeat

Gateway perlu mengirim heartbeat.

Contoh:

```http
POST /api/v1/device/heartbeat
```

Payload:

```json
{
    "device_id": "BDC000001",
    "firmware": "1.0.0",
    "network": "LTE",
    "signal": 21,
    "queue": 3,
    "uptime": 124523
}
```

Heartbeat digunakan untuk mengetahui apakah device masih online.

---

# 24. Device Status

Parameter minimum:

```text
Device ID
Firmware Version
Bluetooth Status
Wi-Fi Status
GSM Status
Signal Strength
IP Address
Queue Size
Free Storage
Uptime
Last Transaction
```

---

# 25. Bluetooth Connection

State machine yang disarankan:

```text
             +---------+
             | INIT    |
             +----+----+
                  |
                  v
             +---------+
             | SCAN    |
             +----+----+
                  |
                  v
             +---------+
             | CONNECT |
             +----+----+
                  |
                  v
             +---------+
             | CAPTURE |
             +----+----+
                  |
             disconnect
                  |
                  v
             +---------+
             | RECONNECT
             +---------+
```

---

# 26. Data Capture State Machine

```text
IDLE
 |
 | Bluetooth data received
 v
RECEIVING
 |
 | timeout / end-of-print
 v
FRAME_COMPLETE
 |
 v
QUEUE
 |
 v
UPLOAD
 |
 +---- success ----> ACK
 |
 +---- failure ----> RETRY
```

---

# 27. Penentuan End-of-Transaction

Salah satu masalah penting adalah menentukan kapan sebuah print job selesai.

Ada beberapa pendekatan.

### Option A – Idle timeout

Jika tidak ada data selama:

```text
100–500 ms
```

maka dianggap satu print job selesai.

Kelebihan:

* sederhana.

Kekurangan:

* belum tentu aman untuk semua printer.

---

### Option B – ESC/POS command

Mendeteksi command seperti:

```text
GS V
```

atau printer cut command.

Ini lebih reliable apabila printer selalu melakukan paper cut.

---

### Option C – Explicit transaction marker

Jika firmware MKL82 dapat dimodifikasi, tambahkan marker:

```text
START_TRANSACTION
ESC/POS DATA
END_TRANSACTION
```

Ini adalah solusi paling reliable.

---

# 28. Recommended Capture Strategy

Untuk prototype:

```text
RAW BYTE BUFFER
+
IDLE TIMEOUT
+
OPTIONAL ESC/POS CUT DETECTION
```

Kemudian pada production:

```text
TRANSACTION FRAMING
+
SEQUENCE NUMBER
+
CRC
```

---

# 29. Hardware Block Diagram

Konsep hardware:

```text
                 +----------------+
                 | MKL82 POS      |
                 +-------+--------+
                         |
                    Bluetooth
                         |
                         v
                 +---------------+
                 | Bluetooth     |
                 | Module        |
                 +-------+-------+
                         |
                        UART
                         |
                         v
                 +---------------+
                 | Gateway MCU   |
                 |               |
                 | Buffer        |
                 | Queue         |
                 | Protocol      |
                 +---+-------+---+
                     |       |
                     |       |
                   UART     UART
                     |       |
                     v       v
                  Wi-Fi     GSM
                  Module    Module
```

Optional:

```text
                +----------------+
                | External Flash |
                | / EEPROM       |
                +----------------+
```

untuk store-and-forward.

---

# 30. MCU Gateway Requirement

MCU gateway minimal membutuhkan:

```text
UART >= 2
RAM >= 64 KB
Flash >= 256 KB
Timer
Watchdog
DMA
RTC optional
```

Lebih ideal:

```text
RAM >= 128 KB
Flash >= 512 KB
DMA
Crypto accelerator
USB optional
Ethernet optional
```

Kebutuhan RAM terutama ditentukan oleh:

* buffer ESC/POS
* network buffer
* TLS
* JSON
* modem communication.

---

# 31. Firmware Architecture

Firmware dapat dibagi:

```text
+--------------------------------+
| Application                    |
|                                |
| Transaction Manager            |
| Upload Manager                 |
| Device Manager                 |
+--------------------------------+
| Protocol Layer                 |
|                                |
| HTTP Client                    |
| TCP Client                     |
| ESC/POS Capture                |
+--------------------------------+
| Communication Layer            |
|                                |
| Bluetooth Driver               |
| Wi-Fi Driver                   |
| GSM Driver                     |
| UART Driver                    |
+--------------------------------+
| Hardware Abstraction Layer     |
+--------------------------------+
```

---

# 32. Firmware Task

Jika menggunakan RTOS:

```text
Bluetooth Task
Network Task
Upload Task
Storage Task
Watchdog Task
Heartbeat Task
```

Contoh:

```text
Bluetooth Task
      |
      v
Transaction Queue
      |
      v
Upload Task
      |
      v
Network
```

---

# 33. Error Handling

Error harus diklasifikasikan.

### Bluetooth

```text
BT_DISCONNECTED
BT_TIMEOUT
BT_INVALID_DATA
BT_BUFFER_OVERFLOW
```

### Network

```text
WIFI_DISCONNECTED
GSM_DISCONNECTED
NO_IP
DNS_ERROR
TCP_TIMEOUT
HTTP_ERROR
TLS_ERROR
```

### Storage

```text
FLASH_FULL
WRITE_ERROR
READ_ERROR
CRC_ERROR
```

---

# 34. Watchdog

Gateway harus menggunakan hardware watchdog.

Jika:

```text
Bluetooth driver hang
Modem hang
Network stack hang
```

maka device melakukan reset.

Setelah reset, queue yang belum terkirim harus tetap tersedia.

---

# 35. Backend API

Minimal API:

```text
POST /api/v1/pos/print-data

POST /api/v1/device/heartbeat

GET  /api/v1/device/config

GET  /api/v1/device/command
```

Optional:

```text
POST /api/v1/device/log
POST /api/v1/device/status
```

---

# 36. Backend Data Model

Contoh:

```text
devices
-------------------------
device_id
serial_number
firmware_version
last_seen
network_type
signal_strength
status
created_at
```

dan:

```text
print_transactions
-------------------------
transaction_id
device_id
timestamp
payload
payload_length
status
received_at
```

---

# 37. Observability

Backend sebaiknya menyimpan:

```text
device online/offline
last heartbeat
transaction count
failed transaction
retry count
firmware version
signal strength
```

Dashboard:

```text
+------------------------------------+
| DEVICE BDC000001                   |
+------------------------------------+
| Status       ONLINE                |
| Network      LTE                   |
| Signal       -73 dBm               |
| Firmware     1.0.3                 |
| Queue        0                     |
| Last Print   14:32:21              |
+------------------------------------+
```

---

# 38. Firmware Update

Tahap berikutnya dapat ditambahkan:

```text
OTA Firmware Update
```

Flow:

```text
Server
  |
  | firmware available
  v
Gateway
  |
  | download
  v
Flash
  |
  | verify CRC/signature
  v
Bootloader
  |
  v
New Firmware
```

OTA tidak wajib untuk prototype awal, tetapi sangat direkomendasikan untuk production.

---

# 39. Prototype Development Stages

## Phase 1 – Bluetooth Capture

Target:

```text
POS
 |
 | Bluetooth
 v
Gateway
 |
 | UART debug
 v
PC
```

Tujuan:

memastikan data ESC/POS dapat ditangkap secara byte-perfect.

---

## Phase 2 – Network Upload

```text
POS
 |
Bluetooth
 |
Gateway
 |
Wi-Fi
 |
HTTP POST
 |
Server
```

Target:

server menerima raw ESC/POS.

---

## Phase 3 – Store and Forward

Tambahkan:

```text
Flash
Queue
Retry
ACK
```

---

## Phase 4 – GSM

Tambahkan cellular modem.

Test:

```text
Wi-Fi OFF
GSM ON
```

Gateway tetap mengirim data.

---

## Phase 5 – Production Protocol

Tambahkan:

```text
TLS
Authentication
CRC
Sequence Number
Transaction ID
Watchdog
OTA
Remote Configuration
```

---

# 40. Test Plan

### Test 1 – ASCII

```text
HELLO WORLD
```

Pastikan:

```text
TX == RX
```

---

### Test 2 – ESC/POS

Test:

```text
ESC @
ESC a
ESC E
GS !
GS V
```

---

### Test 3 – Large Print

Test receipt:

```text
1 KB
5 KB
10 KB
50 KB
```

---

### Test 4 – Network Disconnect

```text
Print
↓
Internet OFF
↓
Print
↓
Internet ON
```

Expected:

```text
All transactions eventually reach server.
```

---

### Test 5 – Gateway Reset

```text
Print
↓
Gateway reset
↓
Boot
```

Expected:

```text
No transaction loss.
```

---

### Test 6 – GSM/Wi-Fi Failover

```text
Wi-Fi ON
↓
Print
↓
Wi-Fi OFF
↓
GSM ON
↓
Print
```

Expected:

```text
Transaction delivered.
```

---

# 41. Critical Engineering Questions

Sebelum hardware difinalisasi, beberapa hal harus dipastikan.

### 1. Bagaimana Bluetooth terhubung ke MKL82?

Apakah:

```text
MKL82 → Bluetooth Module
```

atau:

```text
MKL82 → Bluetooth Gateway
```

?

---

### 2. Apakah gateway menjadi printer Bluetooth?

Ini sangat penting.

Ada dua kemungkinan arsitektur:

### Architecture A

```text
POS
 |
Bluetooth
 |
Gateway
```

Gateway bertindak sebagai receiver.

### Architecture B

```text
POS
 |
Bluetooth
 |
Gateway
 |
Bluetooth
 |
Printer
```

Gateway menjadi **Bluetooth MITM / bridge**.

Architecture B jauh lebih kompleks karena gateway harus meneruskan data ke printer sekaligus melakukan capture.

---

# 42. Arsitektur yang Direkomendasikan

Jika memungkinkan, desain:

```text
                 +----------------+
                 | MKL82 POS      |
                 +-------+--------+
                         |
                         | Bluetooth
                         v
                 +---------------+
                 | Data Capture  |
                 | Gateway       |
                 +-------+-------+
                         |
                         | Wi-Fi/GSM
                         v
                    INTERNET
                         |
                         v
                      SERVER
```

Jika printer masih membutuhkan Bluetooth connection:

```text
                 +----------------+
                 | MKL82 POS      |
                 +-------+--------+
                         |
                    Bluetooth
                         |
                         v
                +------------------+
                | Capture Gateway  |
                |                  |
                | BT RX            |
                | BT TX            |
                +--------+---------+
                         |
                    Bluetooth
                         |
                         v
                     PRINTER
```

Pada arsitektur kedua, gateway harus mendukung **Bluetooth proxy/bridge**.

---

# 43. Prinsip Desain Utama

Sistem sebaiknya mengikuti prinsip:

> **Capture first, understand later.**

Gateway tidak perlu memahami transaksi pada tahap awal.

Yang penting:

```text
CAPTURE
    ↓
BUFFER
    ↓
STORE
    ↓
FORWARD
    ↓
ACK
```

Parsing transaksi dapat dilakukan di backend.

Dengan demikian firmware gateway tetap sederhana.

---

# 44. Recommended Initial Architecture

Untuk prototype pertama:

```text
                 MKL82 POS
                     |
                     |
                 Bluetooth
                     |
                     v
              +-------------+
              | Gateway MCU |
              +------+------+
                     |
              +------+------+
              |             |
             UART          UART
              |             |
              v             v
            Wi-Fi          GSM
              |             |
              +------+------+
                     |
                   HTTPS
                     |
                     v
              +-------------+
              | Backend API |
              +-------------+
```

Protocol:

```text
Bluetooth
    ↓
RAW ESC/POS
    ↓
Binary Buffer
    ↓
HTTP POST
    ↓
HTTPS
```

Fallback:

```text
HTTP unavailable
       ↓
TCP/IP
```

Persistence:

```text
External/Internal Flash
       ↓
Transaction Queue
```

---

# 45. Minimum Viable Prototype

MVP tidak perlu langsung mempunyai seluruh fitur.

Minimum hardware:

```text
MCU
Bluetooth
Wi-Fi OR GSM
Flash
UART
```

Minimum firmware:

```text
Bluetooth RX
ESC/POS Buffer
HTTP Client
Retry
Sequence Number
Watchdog
```

Minimum backend:

```text
POST endpoint
Device authentication
Raw data storage
Transaction acknowledgement
```

Flow MVP:

```text
MKL82
  |
  | ESC/POS
  v
Bluetooth
  |
  v
Gateway
  |
  | HTTP POST
  v
Server
```

Jika flow tersebut sudah stabil, baru ditambahkan:

```text
GSM
Store-and-forward
TLS
OTA
Dashboard
Remote configuration
```

---

# 46. Acceptance Criteria

Prototype dianggap berhasil apabila:

1. Gateway dapat menerima data ESC/POS dari POS.
2. Data yang diterima identik dengan data yang dikirim POS.
3. Gateway dapat menyimpan data sementara.
4. Gateway dapat mengirim data menggunakan HTTP POST.
5. Server dapat memberikan ACK.
6. Gateway dapat melakukan retry.
7. Tidak terjadi kehilangan data ketika koneksi Internet sementara terputus.
8. Gateway dapat melakukan reconnect Bluetooth.
9. Gateway dapat recovery setelah reset.
10. Gateway dapat mengirim minimal satu transaksi secara end-to-end.

---

# 47. Kesimpulan

Bluetooth Data Capture Gateway dirancang sebagai **communication bridge** antara POS berbasis MKL82 dan backend cloud.

Konsep utama:

```text
                 CAPTURE
                    ↓
              ESC/POS DATA
                    ↓
                 BUFFER
                    ↓
                 QUEUE
                    ↓
             HTTP POST / HTTPS
                    ↓
                 SERVER
                    ↓
                   ACK
```

HTTP POST menjadi protocol utama karena lebih mudah diintegrasikan dengan backend dan infrastruktur Internet.

TCP/IP proprietary menjadi fallback apabila keterbatasan modem atau requirement sistem membuat HTTP tidak praktis.

Arsitektur ini juga memungkinkan pengembangan bertahap dari prototype sederhana menjadi produk IoT gateway yang memiliki:

* GSM
* Wi-Fi
* Bluetooth
* secure communication
* transaction queue
* store-and-forward
* OTA firmware
* remote configuration
* monitoring
* device management.

**Prioritas engineering pada tahap pertama bukan parsing transaksi, tetapi memastikan bahwa byte stream ESC/POS dapat ditangkap dan dikirim secara lossless.**
