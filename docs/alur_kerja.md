# DOKUMEN ALUR KERJA PENGEMBANGAN

## Bluetooth Thermal Printer dengan HTTP API, FC41D Wi-Fi Module, dan MKL82 MCU

**Dokumen:** Technical Development Workflow
**Platform MCU:** NXP MKL82
**Modul Wi-Fi:** FC41D
**Interface MCU–Wi-Fi:** UART / AT Command
**Interface MCU–Printer:** Bluetooth UART / Bluetooth Printer Interface
**Printer:** Thermal Printer dengan dukungan ESC/POS
**Data Utama:** QR String dari HTTP API

---

# 1. Tujuan

Dokumen ini mendefinisikan alur kerja pengembangan sistem printer thermal berbasis Bluetooth yang memperoleh data QR melalui HTTP API menggunakan modul Wi-Fi FC41D.

Sistem menggunakan MKL82 sebagai **main controller** yang bertanggung jawab terhadap:

1. komunikasi dengan modul Wi-Fi FC41D;
2. pengiriman HTTP request melalui AT Command;
3. penerimaan response HTTP;
4. parsing response;
5. ekstraksi QR string;
6. validasi data;
7. formatting data untuk printer;
8. konversi QR string menjadi ESC/POS command;
9. pengiriman command ke Bluetooth thermal printer;
10. monitoring error dan status sistem.

---

# 2. Arsitektur Sistem

Arsitektur utama sistem:

```text
                    INTERNET
                       │
                       │ HTTP / HTTPS
                       ▼
                ┌───────────────┐
                │   HTTP API    │
                │ QR Generator  │
                └───────┬───────┘
                        │
                        │ Wi-Fi
                        ▼
                ┌───────────────┐
                │    FC41D      │
                │ Wi-Fi Module  │
                │ AT Command    │
                └───────┬───────┘
                        │
                        │ UART
                        │ AT Command / Response
                        ▼
                ┌───────────────┐
                │    MKL82      │
                │     MCU       │
                │               │
                │ HTTP Control  │
                │ Parser        │
                │ QR Formatter  │
                │ ESC/POS       │
                │ State Machine │
                └───────┬───────┘
                        │
                        │ UART / Bluetooth Interface
                        ▼
                ┌───────────────┐
                │   Bluetooth   │
                │ Thermal Printer│
                └───────┬───────┘
                        │
                        ▼
                 ┌─────────────┐
                 │ QR / Receipt│
                 │   Printed   │
                 └─────────────┘
```

---

# 3. Pembagian Fungsi

## 3.1 MKL82

MKL82 berfungsi sebagai controller utama.

Tanggung jawab:

* system initialization;
* UART driver;
* FC41D command manager;
* Wi-Fi connection manager;
* HTTP transaction manager;
* response buffer;
* JSON/string parser;
* QR validation;
* ESC/POS command generator;
* printer communication;
* timeout management;
* retry mechanism;
* error handling;
* system state machine;
* logging/debugging.

MKL82 **tidak perlu menangani protokol TCP/IP secara langsung** apabila seluruh fungsi networking telah disediakan oleh FC41D melalui AT Command.

---

## 3.2 FC41D

FC41D berfungsi sebagai network interface.

Tanggung jawab:

* Wi-Fi scanning;
* Wi-Fi association;
* DHCP;
* DNS;
* TCP/TLS apabila diperlukan;
* HTTP/HTTPS communication;
* menerima command dari MKL82 melalui UART;
* mengembalikan response kepada MKL82.

Secara konsep:

```text
MKL82
  │
  │ AT Command
  ▼
FC41D
  │
  │ TCP/IP
  ▼
HTTP Server
```

---

# 4. Alur Data Utama

Alur normal sistem:

```text
START
  │
  ▼
Initialize MKL82
  │
  ▼
Initialize UART
  │
  ▼
Initialize FC41D
  │
  ▼
Connect Wi-Fi
  │
  ▼
Wi-Fi Connected?
  │
  ├── NO ──► Retry / Error
  │
  ▼ YES
HTTP Request
  │
  ▼
Receive HTTP Response
  │
  ▼
Validate HTTP Status
  │
  ▼
Parse Response
  │
  ▼
Extract QR String
  │
  ▼
Validate QR String
  │
  ▼
Generate ESC/POS Command
  │
  ▼
Send Command to Printer
  │
  ▼
Printer Status OK?
  │
  ├── NO ──► Retry / Error
  │
  ▼ YES
Print Completed
  │
  ▼
END / READY
```

---

# 5. Tahapan Pengembangan

## Phase 1 — Requirement Definition

Sebelum firmware dibuat, parameter sistem harus ditentukan.

### Parameter FC41D

* UART baud rate
* UART format
* AT Command set
* Wi-Fi SSID
* Wi-Fi authentication
* DHCP/static IP
* DNS
* HTTP support
* HTTPS support
* TLS certificate requirement
* maximum HTTP response size
* timeout
* retry count

### Parameter MKL82

* clock frequency
* UART peripheral
* UART buffer size
* RAM availability
* Flash availability
* watchdog
* RTC jika diperlukan
* non-volatile configuration

### Parameter Printer

* Bluetooth profile/interface
* baud rate jika menggunakan serial bridge
* ESC/POS compatibility
* paper width
* printer resolution
* QR capability
* maximum QR data length
* status feedback capability.

---

# 6. Phase 2 — Hardware Development

Blok hardware:

```text
              ┌─────────────┐
              │    MKL82    │
              └──────┬──────┘
                     │
              UART / GPIO
                     │
             ┌───────▼───────┐
             │     FC41D     │
             │ Wi-Fi Module  │
             └───────────────┘

MKL82
  │
  │ UART / Bluetooth interface
  ▼
Bluetooth Thermal Printer
```

Hal-hal yang harus diperhatikan:

### Power Supply

Pastikan:

* tegangan MKL82 sesuai requirement;
* tegangan FC41D sesuai requirement;
* supply printer terpisah apabila membutuhkan arus besar;
* ground antarperangkat common;
* regulator mampu menangani peak current Wi-Fi;
* printer tidak menyebabkan voltage dip pada sistem MCU.

### UART

Minimal:

```text
MKL82 TX ─────► FC41D RX
MKL82 RX ◄───── FC41D TX
GND     ─────── GND
```

Untuk printer:

```text
MKL82 TX ─────► Printer RX
MKL82 RX ◄───── Printer TX
GND     ─────── GND
```

Jika level tegangan berbeda, gunakan level translator yang sesuai.

---

# 7. Phase 3 — FC41D Bring-Up

Pengembangan firmware sebaiknya dimulai dari komunikasi paling dasar.

Test sequence:

```text
MKL82
 │
 ├── AT
 │
 ├── AT+...
 │
 ├── Wi-Fi scan
 │
 ├── Set SSID
 │
 ├── Set password
 │
 ├── Connect Wi-Fi
 │
 └── Check IP
```

Target hasil:

```text
FC41D
   │
   ├── UART communication OK
   ├── Wi-Fi connected
   ├── IP acquired
   ├── DNS working
   └── Internet reachable
```

Jangan langsung menggabungkan HTTP dan printer pada tahap ini.

---

# 8. Phase 4 — HTTP API Development

Setelah koneksi Wi-Fi stabil, implementasikan HTTP API.

Contoh request konseptual:

```text
GET /api/qr?id=123456
Host: api.example.com
```

Server mengembalikan data misalnya:

```json
{
    "status": "success",
    "qr": "00020101021226670016COM.EXAMPLE..."
}
```

Data yang diperlukan MCU:

```text
qr =
00020101021226670016COM.EXAMPLE...
```

---

# 9. HTTP Response Handling

MKL82 harus memisahkan beberapa bagian response:

```text
HTTP Response
     │
     ├── Status Line
     │
     ├── Headers
     │
     └── Body
             │
             ▼
          JSON Data
             │
             ▼
          QR String
```

Contoh:

```text
HTTP/1.1 200 OK

Content-Type: application/json

{
    "status":"success",
    "qr":"000201010212..."
}
```

Parser hanya mengambil:

```text
000201010212...
```

---

# 10. QR String Parser

Parser tidak boleh hanya mencari seluruh response secara sembarangan.

Tahapan:

```text
HTTP Response
      │
      ▼
Find JSON Body
      │
      ▼
Check "status"
      │
      ▼
Find "qr"
      │
      ▼
Extract string
      │
      ▼
Remove quotation mark
      │
      ▼
Validate length
      │
      ▼
Validate character
      │
      ▼
QR String Ready
```

Contoh:

```text
Input:

{"status":"success","qr":"ABC123456789"}

Output:

ABC123456789
```

---

# 11. Validasi QR String

Sebelum dikirim ke printer:

```text
QR String
   │
   ├── NULL?
   │
   ├── Empty?
   │
   ├── Length > buffer?
   │
   ├── Invalid character?
   │
   └── Valid?
```

Jika invalid:

```text
QR_INVALID
```

Jika valid:

```text
QR_VALID
```

Validasi penting untuk mencegah:

* buffer overflow;
* corrupted data;
* printer command corruption;
* QR tidak dapat dibaca;
* data HTTP ikut tercetak.

---

# 12. Phase 5 — Bluetooth Printer Bring-Up

Printer harus diuji secara terpisah dari HTTP.

Test:

```text
MKL82
   │
   │ "HELLO"
   ▼
Bluetooth Printer
```

Kemudian:

```text
MKL82
   │
   │ ESC/POS command
   ▼
Printer
   │
   ▼
Printed Test Page
```

Test minimum:

1. text;
2. newline;
3. alignment;
4. bold;
5. font size;
6. QR;
7. feed;
8. cut apabila printer mendukung cutter.

---

# 13. ESC/POS Layer

Firmware sebaiknya memiliki abstraction layer:

```text
Application
     │
     ▼
Printer Manager
     │
     ▼
ESC/POS Generator
     │
     ▼
Bluetooth Transport
     │
     ▼
Printer
```

Jangan mencampur kode HTTP dengan kode ESC/POS.

Contoh struktur:

```text
app/
    main.c
    printer_manager.c
    qr_manager.c
    http_manager.c

drivers/
    uart.c
    fc41d.c
    bluetooth.c

protocol/
    at_command.c
    http_parser.c
    json_parser.c
    escpos.c
```

---

# 14. Contoh ESC/POS QR Flow

Data:

```text
QR String:
ABC123456789
```

menjadi:

```text
ESC/POS Command
       │
       ├── Initialize
       │
       ├── Set alignment
       │
       ├── Set QR model
       │
       ├── Set QR size
       │
       ├── Set error correction
       │
       ├── Send QR data
       │
       ├── Print QR
       │
       └── Feed
```

Secara konseptual:

```text
QR STRING
   │
   ▼
ESC/POS QR Encoder
   │
   ▼
Binary Command Stream
   │
   ▼
Bluetooth Transport
   │
   ▼
Thermal Printer
```

Perintah ESC/POS yang digunakan harus mengikuti **command set printer yang sebenarnya**, karena dukungan QR dan format command dapat berbeda antarprinter.

---

# 15. State Machine Firmware

Firmware disarankan menggunakan state machine.

Contoh:

```text
STATE_INIT
    │
    ▼
STATE_WIFI_INIT
    │
    ▼
STATE_WIFI_CONNECT
    │
    ├──────── FAIL ────────► STATE_WIFI_RETRY
    │
    ▼
STATE_WIFI_READY
    │
    ▼
STATE_HTTP_REQUEST
    │
    ▼
STATE_HTTP_RECEIVE
    │
    ▼
STATE_HTTP_PARSE
    │
    ├──────── FAIL ────────► STATE_HTTP_ERROR
    │
    ▼
STATE_QR_VALIDATE
    │
    ├──────── FAIL ────────► STATE_QR_ERROR
    │
    ▼
STATE_PRINTER_CONNECT
    │
    ├──────── FAIL ────────► STATE_PRINTER_ERROR
    │
    ▼
STATE_PRINT
    │
    ▼
STATE_PRINT_VERIFY
    │
    ├──────── FAIL ────────► STATE_PRINT_ERROR
    │
    ▼
STATE_PRINT_DONE
    │
    ▼
STATE_READY
```

Pendekatan state machine lebih mudah di-debug daripada satu fungsi `main()` yang melakukan seluruh proses secara blocking.

---

# 16. Sequence Diagram

Sequence normal:

```text
MKL82          FC41D          HTTP API        Printer
  │               │               │              │
  │── AT ────────►│               │              │
  │◄── OK ────────│               │              │
  │               │               │              │
  │── Wi-Fi ─────►│               │              │
  │◄── CONNECTED ─│               │              │
  │               │               │              │
  │── HTTP GET ──►│               │              │
  │               │── HTTP GET ──►│              │
  │               │◄── Response ──│              │
  │◄── Response ──│               │              │
  │               │               │              │
  │── Parse QR ───│               │              │
  │               │               │              │
  │─────────────────────────────────────────────►│
  │              ESC/POS                         │
  │◄─────────────────────────────────────────────│
  │               │               │              │
  │              PRINT                          │
  │               │               │              │
```

---

# 17. Error Handling

Sistem harus membedakan error berdasarkan layer.

## Wi-Fi Error

```text
WIFI_DISCONNECTED
WIFI_AUTH_FAILED
WIFI_TIMEOUT
WIFI_NO_IP
```

## HTTP Error

```text
HTTP_TIMEOUT
HTTP_CONNECTION_FAILED
HTTP_400
HTTP_401
HTTP_404
HTTP_500
```

## Parser Error

```text
INVALID_JSON
QR_NOT_FOUND
QR_EMPTY
QR_TOO_LONG
QR_INVALID_CHARACTER
```

## Printer Error

```text
BT_NOT_CONNECTED
PRINTER_TIMEOUT
PRINTER_BUSY
PAPER_OUT
PRINTER_ERROR
```

---

# 18. Timeout dan Retry

Setiap komunikasi eksternal harus memiliki timeout.

Contoh:

```text
AT Command
    │
    ├── Response received → SUCCESS
    │
    └── Timeout → RETRY
```

Retry tidak boleh infinite.

Contoh:

```text
Retry = 0

Retry < 3 ?
   │
   ├── YES → retry
   │
   └── NO → ERROR
```

Untuk HTTP:

```text
HTTP Request
     │
     ▼
Timeout
     │
     ▼
Retry #1
     │
     ▼
Retry #2
     │
     ▼
Retry #3
     │
     ▼
HTTP_ERROR
```

---

# 19. Buffer Management

Karena MKL82 adalah embedded MCU, buffer harus dirancang sejak awal.

Minimal buffer:

```text
UART RX Buffer
UART TX Buffer
HTTP Header Buffer
HTTP Body Buffer
JSON Buffer
QR Buffer
Printer TX Buffer
```

Contoh:

```text
FC41D
  │
  ▼
UART RX
  │
  ▼
HTTP Buffer
  │
  ▼
Parser
  │
  ▼
QR Buffer
  │
  ▼
ESC/POS Buffer
  │
  ▼
Printer
```

Jangan menggunakan dynamic allocation secara berlebihan untuk data yang sebenarnya dapat ditangani dengan fixed-size buffer.

---

# 20. Logging dan Debugging

Selama development, tambahkan debug log.

Contoh:

```text
[INFO] System Init
[INFO] UART Init
[INFO] FC41D Init
[INFO] WiFi Connecting...
[INFO] WiFi Connected
[INFO] HTTP Request
[INFO] HTTP Status: 200
[INFO] JSON Parse OK
[INFO] QR Length: 87
[INFO] Printer Connecting
[INFO] Printer Connected
[INFO] Sending ESC/POS
[INFO] Print Complete
```

Untuk error:

```text
[ERROR] FC41D Timeout
[ERROR] HTTP Request Failed
[ERROR] QR Field Not Found
[ERROR] Printer Connection Failed
```

---

# 21. Test Plan

## Test 1 — MCU UART

**Input:** AT Command

**Expected:**

```text
AT
→ OK
```

---

## Test 2 — Wi-Fi

**Input:** valid SSID/password

**Expected:**

```text
Wi-Fi Connected
IP Address Obtained
```

---

## Test 3 — HTTP

**Input:** API endpoint

**Expected:**

```text
HTTP 200
Response Received
```

---

## Test 4 — QR Parsing

**Input:**

```json
{
    "qr": "ABC123"
}
```

**Expected:**

```text
QR = ABC123
```

---

## Test 5 — Printer Text

**Input:**

```text
HELLO PRINTER
```

**Expected:**

```text
HELLO PRINTER
```

tercetak pada thermal printer.

---

## Test 6 — QR Printing

**Input:**

```text
ABC123456789
```

**Expected:**

QR code tercetak dan dapat dibaca menggunakan QR scanner.

---

## Test 7 — API → Printer End-to-End

```text
HTTP API
   │
   ▼
FC41D
   │
   ▼
MKL82
   │
   ▼
Parser
   │
   ▼
ESC/POS
   │
   ▼
Bluetooth
   │
   ▼
Printer
```

Expected:

**QR dari API tercetak identik dengan data yang diterima.**

---

# 22. Development Milestone

Pengembangan disarankan dibagi menjadi milestone berikut:

### M1 — Hardware Bring-Up

* MKL82 boot
* UART working
* power supply verified

### M2 — FC41D Communication

* AT command working
* Wi-Fi connected
* IP acquired

### M3 — HTTP Client

* HTTP request
* response reception
* timeout/retry

### M4 — Parser

* HTTP body extraction
* JSON parsing
* QR extraction
* QR validation

### M5 — Printer

* Bluetooth connection
* text printing
* ESC/POS initialization

### M6 — QR Printing

* QR command generation
* QR printing
* QR scanner verification

### M7 — End-to-End Integration

```text
API
 ↓
FC41D
 ↓
MKL82
 ↓
Parser
 ↓
ESC/POS
 ↓
Bluetooth Printer
```

### M8 — Reliability Test

* Wi-Fi disconnect
* API timeout
* invalid response
* printer disconnect
* printer out of paper
* MCU reset
* FC41D reset
* repeated printing
* power-cycle test.

---

# 23. Acceptance Criteria

Sistem dianggap berhasil apabila:

1. MKL82 dapat berkomunikasi dengan FC41D melalui UART.
2. FC41D dapat terhubung ke jaringan Wi-Fi.
3. MKL82 dapat melakukan HTTP transaction melalui FC41D.
4. HTTP response dapat diterima tanpa corruption.
5. QR string dapat diparsing secara konsisten.
6. QR string dapat divalidasi.
7. MKL82 dapat berkomunikasi dengan Bluetooth thermal printer.
8. ESC/POS command dapat dikirim dengan benar.
9. QR dapat tercetak.
10. QR hasil cetak dapat dibaca oleh scanner.
11. Error HTTP tidak menyebabkan firmware hang.
12. Disconnect Wi-Fi dapat ditangani.
13. Disconnect printer dapat ditangani.
14. Timeout dan retry bekerja sesuai konfigurasi.
15. Sistem dapat kembali ke state READY setelah transaksi selesai.

---

# 24. Struktur Firmware yang Disarankan

```text
firmware/
│
├── app/
│   ├── main.c
│   ├── app_state.c
│   ├── app_state.h
│   ├── qr_manager.c
│   └── qr_manager.h
│
├── network/
│   ├── fc41d.c
│   ├── fc41d.h
│   ├── http_client.c
│   ├── http_client.h
│   ├── http_parser.c
│   └── http_parser.h
│
├── printer/
│   ├── printer_manager.c
│   ├── printer_manager.h
│   ├── escpos.c
│   ├── escpos.h
│   ├── bluetooth.c
│   └── bluetooth.h
│
├── drivers/
│   ├── uart.c
│   ├── uart.h
│   ├── timer.c
│   ├── timer.h
│   ├── gpio.c
│   └── gpio.h
│
└── config/
    ├── config.h
    └── product_config.h
```

---

# 25. Prinsip Arsitektur Software

Firmware sebaiknya menggunakan prinsip:

```text
Application
    │
    ├──────── Network Service
    │
    └──────── Printer Service

Network Service
    │
    └── FC41D Driver
            │
            └── UART Driver

Printer Service
    │
    ├── ESC/POS
    │
    └── Bluetooth Driver
            │
            └── UART/Transport Driver
```

Dengan demikian apabila suatu saat:

```text
FC41D → ESP32
```

atau:

```text
Bluetooth Printer → USB Printer
```

diganti, bagian application tidak perlu ditulis ulang seluruhnya.

---

# 26. Alur Operasional Produk

Pada kondisi normal:

```text
             USER / SYSTEM
                   │
                   │ Request Print
                   ▼
             ┌─────────────┐
             │   MKL82     │
             └──────┬──────┘
                    │
                    │ AT Command
                    ▼
             ┌─────────────┐
             │   FC41D     │
             └──────┬──────┘
                    │
                    │ Wi-Fi
                    ▼
             ┌─────────────┐
             │ HTTP Server │
             └──────┬──────┘
                    │
                    │ QR JSON
                    ▼
             ┌─────────────┐
             │   FC41D     │
             └──────┬──────┘
                    │ UART
                    ▼
             ┌─────────────┐
             │    MKL82    │
             │             │
             │ JSON Parser │
             │     ↓       │
             │ QR String   │
             │     ↓       │
             │ ESC/POS     │
             └──────┬──────┘
                    │
                    │ Bluetooth
                    ▼
             ┌─────────────┐
             │   Printer   │
             └──────┬──────┘
                    │
                    ▼
                QR PRINT
```

---

# 27. Fokus Risiko Teknis

Risiko utama proyek:

| Area        | Risiko                           |
| ----------- | -------------------------------- |
| FC41D       | AT command timeout               |
| Wi-Fi       | Connection drop                  |
| HTTP        | Response terlalu besar           |
| JSON        | Format response berubah          |
| RAM         | Buffer overflow                  |
| UART        | Data loss                        |
| Bluetooth   | Connection instability           |
| ESC/POS     | Command tidak kompatibel         |
| QR          | Data terlalu panjang             |
| Printer     | Paper out / overheating          |
| Power       | Printer menyebabkan voltage drop |
| Reliability | System hang setelah error        |

Prioritas engineering adalah memastikan setiap layer dapat diuji secara independen sebelum integrasi penuh.

---

# 28. Definition of Done

Produk prototype dinyatakan selesai apabila:

```text
[✓] MKL82 boot
[✓] FC41D communication
[✓] Wi-Fi connection
[✓] HTTP API
[✓] HTTP response parsing
[✓] QR extraction
[✓] QR validation
[✓] Bluetooth connection
[✓] ESC/POS generation
[✓] QR printing
[✓] Error handling
[✓] Retry mechanism
[✓] Power-cycle test
[✓] Long-duration test
[✓] End-to-end test
```

Target akhir:

**HTTP API → FC41D → UART → MKL82 → QR Parsing → ESC/POS → Bluetooth → Thermal Printer**

harus berjalan sebagai satu pipeline yang stabil dan dapat recovery ketika salah satu koneksi eksternal mengalami kegagalan.

