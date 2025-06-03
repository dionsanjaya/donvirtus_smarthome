# donvirtus_smarthome
graph TD
    %% Definisi Warna untuk Node
    classDef ui fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;
    classDef controller fill:#4CAF50,stroke:#333,stroke-width:2px,color:#fff;
    classDef device fill:#FFCA28,stroke:#333,stroke-width:2px,color:#000;
    classDef network fill:#78909C,stroke:#333,stroke-width:2px,color:#fff;
    classDef automation fill:#F44336,stroke:#333,stroke-width:2px,color:#fff;

    %% User Interface
    A[User: Mas Dion<br>Akses dashboard untuk kontrol & monitoring]:::ui -->|Login & kontrol perangkat<br>stroke:#1976D2| B[AINVR Flask Dashboard<br>Web interface di localhost:5000<br>Tampilkan stream, hasil AI, kontrol IoT]:::ui
    A -->|Akses via mobile<br>stroke:#1976D2| C[Home Assistant<br>App di homeassistant.local:8123<br>Kontrol semua perangkat & lihat notifikasi]:::ui

    %% Kontroler Pusat
    B -->|REST API: Ambil data AI & kirim perintah<br>stroke:#1976D2| D[Raspberry Pi<br>Jalankan Home Assistant & Mosquitto<br>Koordinasi perangkat, AI, & otomatisasi]:::controller
    C -->|HTTP/MQTT: Koordinasi & kontrol<br>stroke:#1976D2| D
    D -->|MQTT Broker: Kirim/terima pesan<br>stroke:#FF5722| E[Mosquitto<br>MQTT broker di localhost:1883<br>Hub komunikasi IoT]:::controller

    %% Perangkat IoT: Kamera
    D -->|RTSP via Wi-Fi: Setup stream<br>stroke:#AB47BC| F[MediaMTX<br>Server streaming RTSP di localhost:8554<br>Proses video dari kamera IP]:::controller
    F -->|RTSP Stream: Video feed 1080p<br>stroke:#AB47BC| G[Kamera IP Cam1<br>RTSP: rtsp://camera1_ip:554<br>Stream video untuk AI & monitoring]:::device
    F -->|RTSP Stream: Video feed 1080p<br>stroke:#AB47BC| H[Kamera IP Cam2<br>RTSP: rtsp://camera2_ip:554<br>Stream video untuk AI & monitoring]:::device

    %% AI Processing
    G -->|Frame Video: Input untuk AI<br>stroke:#AB47BC| I[AINVR AI Processing<br>YOLOv8: Deteksi objek person, car<br>dlib: Face detection & embedding<br>FAISS: Face search cepat]:::controller
    H -->|Frame Video: Input untuk AI<br>stroke:#AB47BC| I
    I -->|Hasil Deteksi: Timestamp, kelas, confidence<br>stroke:#FF5722| J[SQLite Database<br>detections.db<br>Simpan hasil AI: objek & wajah]:::controller
    I -->|MQTT Publish: Kirim deteksi<br>stroke:#FF5722| E
    J -->|Hasil AI: Tampilkan laporan<br>stroke:#1976D2| B

    %% Detail Alur Face Recognition (dlib/FAISS)
    I -->|Face Detection: Deteksi wajah dengan dlib<br>stroke:#FF5722| K1[Face Embedding<br>dlib: Ekstrak vektor 128D<br>Data wajah di data/faces/]:::controller
    K1 -->|Face Search: Cari kecocokan dengan FAISS<br>stroke:#FF5722| K2[FAISS Index<br>Indeks IVF/HNSW untuk search cepat<br>Match wajah: Person ID atau Unknown]:::controller
    K2 -->|Hasil Wajah: ID, timestamp<br>stroke:#FF5722| J
    K2 -->|MQTT Publish: Wajah dikenali<br>stroke:#FF5722| E

    %% Detail Alur Object Detection (YOLOv8)
    I -->|Object Detection: Deteksi person, car<br>stroke:#FF5722| L1[YOLOv8 Processing<br>Deteksi objek dengan confidence<br>Contoh: Person 0.92, Car 0.87]:::controller
    L1 -->|Hasil Objek: Kelas, confidence<br>stroke:#FF5722| J
    L1 -->|MQTT Publish: Objek terdeteksi<br>stroke:#FF5722| E

    %% Perangkat IoT: Pintu
    D -->|MQTT via Wi-Fi: Lock/unlock<br>stroke:#FF5722| M[Smart Lock<br>August/Yale, kontrol pintu utama<br>Wi-Fi enabled, status locked/unlocked]:::device
    D -->|MQTT via Zigbee: Status pintu<br>stroke:#2E7D32| N[Door Sensor<br>Aqara, deteksi buka/tutup<br>Zigbee, hemat daya, status open/closed]:::device
    M -->|Status: locked/unlocked<br>stroke:#FF5722| E
    N -->|Status: open/closed<br>stroke:#2E7D32| E

    %% Perangkat IoT: Jendela
    D -->|MQTT via Wi-Fi: Open/close<br>stroke:#FF5722| O[Motorized Window<br>Shelly 1, buka/tutup jendela<br>Wi-Fi relay, status open/closed]:::device
    D -->|MQTT via Zigbee: Status jendela<br>stroke:#2E7D32| P[Window Sensor<br>Aqara, deteksi buka/tutup<br>Zigbee, hemat daya, status open/closed]:::device
    O -->|Status: open/closed<br>stroke:#FF5722| E
    P -->|Status: open/closed<br>stroke:#2E7D32| E

    %% Perangkat IoT: Perabot
    D -->|MQTT via Wi-Fi: On/off<br>stroke:#FF5722| Q[Smart Plug<br>TP-Link Kasa, kontrol lampu/AC<br>Wi-Fi enabled, status on/off]:::device
    D -->|MQTT via Zigbee: Suhu & kelembapan<br>stroke:#2E7D32| R[Temp/Humidity Sensor<br>Aqara, monitor lingkungan<br>Zigbee, suhu > 28°C]:::device
    Q -->|Status: on/off<br>stroke:#FF5722| E
    R -->|Data: Suhu > 28°C<br>stroke:#2E7D32| E

    %% Otomatisasi
    E -->|Person Detected: Dari YOLOv8<br>stroke:#C62828| S[Otomatisasi: Buka Pintu<br>Buka smart lock jika orang terdeteksi]:::automation
    S -->|MQTT Command: unlock<br>stroke:#FF5722| M
    E -->|Face Recognized: Dari FAISS<br>stroke:#C62828| T[Otomatisasi: Buka Pintu (Wajah)<br>Buka smart lock jika wajah dikenali<br>Contoh: Person ID 001]:::automation
    T -->|MQTT Command: unlock<br>stroke:#FF5722| M
    E -->|Person + Sunset: Dari YOLOv8 & waktu<br>stroke:#C62828| U[Otomatisasi: Nyalakan Lampu<br>Nyalakan lampu saat malam & orang terdeteksi]:::automation
    U -->|MQTT Command: on<br>stroke:#FF5722| Q
    E -->|Suhu > 28°C: Dari sensor<br>stroke:#C62828| V[Otomatisasi: Buka Jendela<br>Buka jendela jika suhu tinggi]:::automation
    V -->|MQTT Command: open<br>stroke:#FF5722| O
    E -->|Person Detected: Dari YOLOv8<br>stroke:#C62828| W[Notifikasi Telegram<br>Kirim pesan: 'Person at front door!']:::automation
    E -->|Face Recognized: Dari FAISS<br>stroke:#C62828| X[Notifikasi Telegram (Wajah)<br>Kirim pesan: 'Person ID 001 at door!']:::automation
    W -->|Pesan Terkirim<br>stroke:#1976D2| A
    X -->|Pesan Terkirim<br>stroke:#1976D2| A

    %% Jaringan
    subgraph Jaringan
        G -->|Wi-Fi: Transfer video<br>stroke:#455A64| Y[Router Wi-Fi<br>TP-Link Archer, hubungkan perangkat<br>2.4/5 GHz]:::network
        H -->|Wi-Fi: Transfer video<br>stroke:#455A64| Y
        M -->|Wi-Fi: Kirim perintah<br>stroke:#455A64| Y
        O -->|Wi-Fi: Kirim perintah<br>stroke:#455A64| Y
        Q -->|Wi-Fi: Kirim perintah<br>stroke:#455A64| Y
        D -->|Wi-Fi/Ethernet: Koordinasi<br>stroke:#455A64| Y
        N -->|Zigbee: Data hemat daya<br>stroke:#455A64| Z[Zigbee Hub<br>Conbee II, hubungkan sensor<br>Via USB ke Raspberry Pi]:::network
        P -->|Zigbee: Data hemat daya<br>stroke:#455A64| Z
        R -->|Zigbee: Data hemat daya<br>stroke:#455A64| Z
        Z -->|USB: Koneksi ke Pi<br>stroke:#455A64| D
    end
