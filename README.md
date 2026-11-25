# FMS Onboard Monitoring

Dashboard pemantauan telemetri real-time berbasis web (ASP.NET Core .NET 8 & SignalR) untuk memantau status operasional, kesehatan jaringan perangkat onboard, dan satelit GPS pada armada tambang.

## Cakupan Armada & Parameter

Aplikasi memfilter dan menampilkan unit aktif yang memiliki konfigurasi IP perangkat onboard berdasarkan prefix unit:

- **Loader**: `CB`, `HB`, `KB`
- **Hauler (HD)**: `CT`, `KT`

Setiap kartu unit menampilkan 6 indikator status secara real-time:

| Kode | Parameter | Sumber Data | Keterangan |
| :--- | :--- | :--- | :--- |
| **OP** | Operasional | MineStar | Status mesin aktif (`RUN`), mati (`OFF`), delay (`DLY`), atau breakdown (`BD`) |
| **HS** | Health / Delay | MineStar | Status kondisi operasi normal atau kode delay aktif (`DelayClassName`) |
| **R** | Radio | Ping Collector | Status konektivitas jaringan dan packet loss perangkat Radio onboard |
| **D** | Display | Ping Collector | Status konektivitas jaringan dan packet loss perangkat Display onboard |
| **P** | PLE | Ping Collector | Status konektivitas jaringan dan packet loss perangkat PLE onboard |
| **G** | GPS Satelit | MineStar | Jumlah satelit GPS terhadap ambang batas `GpsRedBelow` dan `GpsYellowMax` |

## Klasifikasi Status Unit

Unit dikelompokkan otomatis ke dalam empat kategori prioritas:

1. **Need Check**: Minimal satu perangkat jaringan (`R`, `D`, atau `P`) mengalami Request Time Out (`Red`), dilengkapi penghitung durasi RTO.
2. **Monitoring**: Terdapat degradasi jaringan (`Yellow` atau `Orange`) pada perangkat onboard tanpa status `Red`.
3. **Breakdown**: Unit berada dalam status breakdown (`is_bd`).
4. **Normal**: Seluruh perangkat jaringan dan status unit beroperasi normal.

## Fitur Utama

- **Live Telemetry Stream**: Pembaruan status kartu unit dan peringatan status collector secara otomatis melalui SignalR (`/telemetryHub`).
- **Pelacakan Durasi RTO**: Menghitung dan menyimpan waktu mulai gangguan (`RTO`) per unit secara persisten di browser maupun dari data backend.
- **Filter & Pencarian Cepat**: Penyaringan berdasarkan kategori status, prefix unit (`CB`, `HB`, `KB`, `CT`, `KT`), kondisi GPS (`GPS normal` / `GPS error`), serta pencarian nama unit atau OID.
- **Panel Detail Unit (Drawer)**: Menampilkan alamat IP, status ping, persentase availability, packet loss per perangkat, jumlah satelit GPS, serta metrik Cycle dan Payload (`>= 67t`) untuk unit Hauler.
- **Availability Dashboard**: Halaman laporan availability historis berbasis SQLite pada rute `/report/sqlite`.

## Catatan Perhitungan Availability

- Periode jam istirahat operasional (`12:00-13:00`) dikecualikan dari kalkulasi availability.
- Unit yang berstatus delay, breakdown (`BD`), atau `IDWP` tidak dihitung sebagai target unit normal dalam ringkasan availability.

## Struktur Direktori Deploy

```text
FmsOnboard/
├── web/                        # Output publish aplikasi ASP.NET Core (.NET 8)
│   ├── FmsOnboard.Web.dll      # Assembly utama aplikasi web & Razor Pages
│   ├── appsettings.json        # Konfigurasi koneksi database & ambang parameter
│   └── wwwroot/                # Aset statis (JS SignalR, CSS, dll.)
├── .gitignore
└── README.md