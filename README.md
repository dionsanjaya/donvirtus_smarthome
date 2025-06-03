# donvirtus_smarthome
```mermaid
---
config:
  layout: elk
  theme: redux
---
flowchart TD
 subgraph Jaringan["Jaringan"]
        U["Router Wi-Fi<br>TP-Link Archer, hubungkan perangkat<br>2.4/5 GHz"]
        G["Kamera IP Cam1<br>RTSP: rtsp://camera1_ip:554<br>Stream video 1080p"]
        H["Kamera IP Cam2<br>RTSP: rtsp://camera2_ip:554<br>Stream video 1080p"]
        K["Smart Lock<br>August/Yale, kontrol pintu utama<br>Wi-Fi enabled"]
        M["Motorized Window<br>Shelly 1, buka/tutup jendela<br>Wi-Fi relay"]
        O["Smart Plug<br>TP-Link Kasa, kontrol lampu/AC<br>Wi-Fi enabled"]
        D["Raspberry Pi<br>Jalankan Home Assistant &amp; Mosquitto<br>Koordinasi perangkat &amp; AI"]
        V["Zigbee Hub<br>Conbee II, hubungkan sensor<br>Via USB ke Raspberry Pi"]
        L["Door Sensor<br>Aqara, deteksi buka/tutup<br>Zigbee, hemat daya"]
        N["Window Sensor<br>Aqara, deteksi buka/tutup<br>Zigbee, hemat daya"]
        P["Temp/Humidity Sensor<br>Aqara, monitor lingkungan<br>Zigbee, hemat daya"]
  end
    A["User: Mas Dion<br>Akses dashboard untuk kontrol &amp; monitoring"] -- Login & kontrol perangkat --> B["AINVR Flask Dashboard<br>Web interface di localhost:5000<br>Tampilkan stream &amp; hasil AI"]
    A -- Akses via mobile --> C["Home Assistant<br>App di homeassistant.local:8123<br>Kontrol semua perangkat"]
    B -- REST API: Ambil data AI --> D
    C -- HTTP/MQTT --> D
    D -- MQTT Broker: Kirim/terima pesan --> E["Mosquitto<br>MQTT broker di localhost:1883<br>Hub komunikasi IoT"]
    D -- "RTSP via Wi-Fi" --> F["MediaMTX<br>Server streaming RTSP di localhost:8554<br>Proses video dari kamera"]
    F -- RTSP Stream: Video feed --> G & H
    G -- Frame Video: Input untuk AI --> I["AINVR AI Processing<br>YOLOv8: Deteksi objek<br>dlib: Face recognition"]
    H -- Frame Video: Input untuk AI --> I
    I -- Hasil Deteksi: Timestamp, kelas --> J["SQLite Database<br>detections.db<br>Simpan hasil AI"]
    I -- MQTT Publish: Kirim deteksi --> E
    J -- Hasil AI: Tampilkan laporan --> B
    D -- "MQTT via Wi-Fi: Lock/unlock" --> K
    D -- MQTT via Zigbee: Status pintu --> L
    L -- Status: open/closed --> E
    K -- Status: locked/unlocked --> E
    D -- "MQTT via Wi-Fi: Open/close" --> M
    D -- MQTT via Zigbee: Status jendela --> N
    N -- Status: open/closed --> E
    M -- Status: open/closed --> E
    D -- "MQTT via Wi-Fi: On/off" --> O
    D -- MQTT via Zigbee: Suhu & kelembapan --> P
    O -- Status: on/off --> E
    P -- Data: Suhu > 28°C --> E
    E -- Person Detected: Dari YOLOv8 --> Q["Otomatisasi: Buka Pintu<br>Buka smart lock jika orang terdeteksi"] & T@{ label: "Notifikasi Telegram<br>Kirim pesan ke Mas Dion<br>Contoh: 'Person at front door!'" }
    Q -- MQTT Command: unlock --> K
    E -- Person + Sunset: Dari YOLOv8 & waktu --> R["Otomatisasi: Nyalakan Lampu<br>Nyalakan lampu saat malam &amp; orang terdeteksi"]
    R -- MQTT Command: on --> O
    E -- Suhu > 28°C: Dari sensor --> S["Otomatisasi: Buka Jendela<br>Buka jendela jika suhu tinggi"]
    S -- MQTT Command: open --> M
    T -- Pesan Terkirim --> A
    G -- "Wi-Fi: Transfer video" --> U
    H -- "Wi-Fi: Transfer video" --> U
    K -- "Wi-Fi: Kirim perintah" --> U
    M -- "Wi-Fi: Kirim perintah" --> U
    O -- "Wi-Fi: Kirim perintah" --> U
    D -- "Wi-Fi/Ethernet: Koordinasi" --> U
    L -- Zigbee: Data hemat daya --> V
    N -- Zigbee: Data hemat daya --> V
    P -- Zigbee: Data hemat daya --> V
    V -- USB: Koneksi ke Pi --> D
    T@{ shape: rect}
     U:::network
     G:::device
     H:::device
     K:::device
     M:::device
     O:::device
     D:::controller
     V:::network
     L:::device
     N:::device
     P:::device
     A:::ui
     B:::ui
     C:::ui
     E:::controller
     F:::controller
     I:::controller
     J:::controller
     Q:::automation
     T:::automation
     R:::automation
     S:::automation
    classDef ui fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff
    classDef controller fill:#4CAF50,stroke:#333,stroke-width:2px,color:#fff
    classDef device fill:#FFCA28,stroke:#333,stroke-width:2px,color:#000
    classDef network fill:#78909C,stroke:#333,stroke-width:2px,color:#fff
    classDef automation fill:#F44336,stroke:#333,stroke-width:2px,color:#fff
'''
