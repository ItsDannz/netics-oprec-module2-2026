# Laporan Penugasan Modul 2 Open Recruitment NETICS 2026

| Nama           | NRP        |
| ---            | ---        |
| Hazza Danta Hermandanu               | 5025241117           |

## Deskripsi Tugas
1. Lakukan instalasi Wazuh Manager pada sebuah VM/VPS berbasis Linux.
2. Lakukan instalasi Wazuh Agent pada satu perangkat lain.
3. Pastikan agent berhasil terhubung ke manager dengan mengecek apakah default events/logs dari agent sudah masuk dan terlihat di dashboard Wazuh.
4. Buat/mencari dan implementasikan minimal 5 custom rules/custom alert pada agent kalian. Jenis rules bebas dan dapat menyesuaikan kreativitas serta konteks keamanan masing-masing.
5. Buat sebuah dokumentasi penjelasan bagaimana kalian menyelesaikan penugasan modul 2 ini termasuk Langkah instalasi manager dan agent, flow/diagram visualisasi deployment, validasi koneksi agent-manager, penjelasan masing-masing custom rule yang ditambahkan, dan screenshot hasil dari dashboard/alert yang menunjukkan rules tersebut aktif pada Github repository

## Langkah Instalasi
1. Buat VPS menggunakan Azure sesuai requirement spec dari Wazuh
    - Create Virtual Machine
    - Konfigurasi Virtual Machine
        <img width="966" height="590" alt="image" src="https://github.com/user-attachments/assets/ab22a41e-d06a-42ec-996e-eb2015b283b3" /><br>
        <img width="959" height="572" alt="image" src="https://github.com/user-attachments/assets/ef5708ef-1644-4330-9a2c-f8424b205a20" /><br>
        <img width="990" height="576" alt="image" src="https://github.com/user-attachments/assets/cb8a5d74-c0ff-4516-93a5-559f2fced8c0" /><br>
    - Konfigurasi storage Virtueal Machine
        <img width="1000" height="633" alt="image" src="https://github.com/user-attachments/assets/d9bb5ee2-04ae-4220-8855-34b924019322" /><br>
    - Review dan Create Virtual Machine
    - Download key SSH dalam bentuk file `*.pem` yang telah digenerate, simpan file `*.pem` di dalam 1 direktori project
    - Setelah selesai, maka akan mendapatkan ip untuk VPS publik<br>
        <img width="497" height="109" alt="image" src="https://github.com/user-attachments/assets/1421d845-7297-4719-91a5-fa7861398965" /><br>
2. Instalasi Wazuh Manager di VPS
    - Download & run Wazuh installation assistent.
      ```bash
      curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
      ```
      <br>
    - Setelah assistent menyelesaikan instalasi, output akan menunjukkan kredensial akses ke Wazuh Dashboard
      ```bash
      INFO: You can access the web interface https://<WAZUH_DASHBOARD_IP_ADDRESS>
        User: admin
        Password: <ADMIN_PASSWORD>
      ```
      <br>
    - Akses dashboard Wazuh dengan mencari `https://<WAZUH_DASHBOARD_IP_ADDRESS>` dan masukkan kredensial yang telah didapatkan untuk login
      
3. Instalasi Wazuh Agent di Virtual Machine
    - Buka port 1514 dan 1515 di VPS untuk Wazuh Agent dan Wazuh Registry.
        <img width="706" height="648" alt="image" src="https://github.com/user-attachments/assets/dd6cd99e-e937-4e70-9b39-6c661d7be68d" /><br>
        <img width="706" height="653" alt="image" src="https://github.com/user-attachments/assets/83ff3003-334c-423b-bd3b-efe7b40f8fc3" /><br>
    - Pergi ke terminal di virtual machine yang akan menjadi Wazuh Agent
    - Install `curl`
      ```bash
      sudo apt update
      sudo apt install curl -y
      ```
      <br>
    - Install GPG Key
      ```bash
      curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
      ```
      <br>
    - Tambahkan Repository
      ```bash
      echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
      ```
      <br>
    - Update informasi package
      ```bash
      apt-get update
      ```
      <br>
    - Deploy Wazuh Agent
      ```bash
      WAZUH_MANAGER="52.175.51.38" apt-get install wazuh-agent -y
      ```
      <br>
    - Start Wazuh Agent
      ```bash
      systemctl daemon-reload
      systemctl enable wazuh-agent
      systemctl start wazuh-agent
      ```
      <br>
4. Visualisasi Deployment
   <img width="1980" height="1980" alt="Device (Windows) WSL Terminal" src="https://github.com/user-attachments/assets/a14c1106-865b-459e-bb6a-aff22397ca2d" /><br>
5. Validasi Koneksi Agent-Manager
    - Di VPS (Wazuh Manager)
      ```bash
      ssh -i ~/wazuh/danu-wazuh_key.pem azureuser@52.175.51.38
      sudo /var/ossec/bin/agent_control -l
      ```
      <br>
      <img width="630" height="117" alt="image" src="https://github.com/user-attachments/assets/f89e2c17-8b52-4140-9e10-ee4db0e6df00" /><br>
    - Di VM (Wazuh Agent)
      ```bash
      sudo tail -f /var/ossec/logs/ossec.log
      ```
      <br>
      <img width="807" height="484" alt="image" src="https://github.com/user-attachments/assets/0c2d4499-ed1d-4602-80a5-ce2f70008a09" /><br>

## Custom Rule
- Buka `/var/ossec/etc/rules/local_rules.xml` untuk menambah custom rule
  ```bash
  nano /var/ossec/etc/rules/local_rules.xml
  ```
  <br>
- Gunakan `rule id` antara 100000 sampai 120000
  
1. Brute Force Detection
   ```xml
   <rule id="100001" level="12" frequency="5" timeframe="120">
      <if_matched_sid>5760</if_matched_sid>
      <same_source_ip />
      <description>SSH Brute Force detected from $(srcip) - Direct attack!</description>
      <mitre>
        <id>T1110</id>
      </mitre>
      <group>brute_force</group>
    </rule>
   ```
   
    - Konteks Keamanan
      Attacker menggunakan tools seperti Hydra untuk mencoba kombinasi password secara otomatis ke port SSH.
    - Penjelasan
        -  Header
            ```xml
            <rule id="100001" level="12" frequency="5" timeframe="120">
            ```

            - Set `rule id`, `level`, `frequency`, dan `timeframe`
                - `rule id = "100001"`: unique id untuk rule yang dibuat
                - `level="12"`: tingkat bahaya adalah 12
                - `frequency="5"`: Rule akan aktif jika kejadian yang dimaksud aktif 5 kali
                - `timeframe="120"`: Kejadian tersebut harus terjadi dalam waktu <= 120 detik
            - Trigger
              ```xml
              <if_matched_sid>5760</if_matched_sid>
              ```
              - Jika ada kejadian 5760 sebanyak 5 kali dalam 120 detik, maka rule 100001 akan aktif
            - `<same_source_ip />`
              - Memastikan bahwa 5 kejadian tadi berasal dari ip yang sama
            - Deskripsi pesan
              ```xml
              <description>SSH Brute Force detected from $(srcip) - Direct attack!</description>
              ```
              - Pesan akan muncul jika rule aktif, `$(srcip)` otomatis mengambil ip penyerang
            - Framework Penyerangan
              ```xml
              <mitre>
                <id>T1110</id>
              </mitre>
              ```
              - Menggunakan teknik `T1110` atau bruteforce
            - Grouping
              ```xml
              <group>brute_force</group>
              ```
              - Mengkategorikan rule sebagai `brute_force`
    - Testing
        - Install hydra
          ```bash
          sudo apt install hydra -y
          ```
        - Downlaod file `RockYou.txt` yang berisi kumpulan password untuk testing hydra
          ```bash
          wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt -O ~/rockyou.txt
          ```
        - Install `openssh` dan start
          ```bash
          sudo apt install openssh-server -y
          sudo systemctl start ssh
          ```
        - Lakukan Attacking
          ```bash
          hydra -l root -P ~/rockyou.txt 52.175.51.38 ssh -t 4 -V
          ```
        - Hasil Alert
          <img width="1600" height="311" alt="image" src="https://github.com/user-attachments/assets/3d7cb8a7-520d-4ad2-b3d1-7bcdda703901" /><br>
2. Brute Force Success
   ```xml
   <rule id="100002" level="14">
    <if_matched_sid>100001</if_matched_sid>
    <if_sid>5715</if_sid>
    <same_source_ip />
    <description>Brute force SUCCESS! Login after multiple failures from $(srcip)</description>
    <mitre>
      <id>T1110</id>
      <id>T1078</id>
    </mitre>
    <group>brute_force</group>
   </rule>
   ```
   - Konteks Keamanan
      Setelah bebera[a percobaan, attacker akhirnya berhasil menebak password yang benar.
    - Penjelasan
        -  Header
            ```xml
            <rule id="100002" level="14">
            ```

            - Set `rule id` dan `level`
                - `rule id = "100002"`: unique id untuk rule yang dibuat
                - `level="14"`: tingkat bahaya adalah 14
            - Trigger
              ```xml
              <if_matched_sid>100001</if_matched_sid>
              <if_sid>5715</if_sid>
              ```
              - Rule akan aktif jika sebelumnya rule 100001 aktif dan rule 5715 terjadi (attacker berhasil login)
            - `<same_source_ip />`
              - Memastikan bahwa ip yang gagal masuk berkali-kali tadi berasal dari ip yang sama
            - Deskripsi pesan
              ```xml
              <description>Brute force SUCCESS! Login after multiple failures from $(srcip)</description>
              ```
              - Pesan akan muncul jika rule aktif, `$(srcip)` otomatis mengambil ip penyerang
            - Framework Penyerangan
              ```xml
              <mitre>
                <id>T1110</id>
                <id>T1078</id>
              </mitre>
              ```
              - Menggunakan teknik `T1110` atau bruteforce
              - Menambahkan `T1078` yaitu valid account yang artinya attacker sekarang mempunyai akun valid untuk masuk
            - Grouping
              ```xml
              <group>brute_force</group>
              ```
              - Mengkategorikan rule sebagai `brute_force`
            - `</rule>` untuk menutup rule
    - Testing
        - Aktifkan rule 100001 sebelumnya
        - Masuk ke VPS (Wazuh Manager) melalui VM (Wazuh Agent) dan masukkan password yang benar
          ```bash
          ssh azureuser@52.175.51.38
          ```
        - Hasil Alert
          <img width="1894" height="92" alt="image" src="https://github.com/user-attachments/assets/57be4bdb-27bc-4b7f-911f-ef025486c9c5" /><br>
3. Mass File Modification
   ```xml
   <rule id="100003" level="10" frequency="20" timeframe="60">
    <if_matched_sid>550</if_matched_sid>
    <description>Mass file modification detected - $(frequency) files changed. Possible ransomware!</description>
    <group>ransomware,fim,mass_change</group>
   </rule>
   ```
   - Konteks Keamanan
      Attacker mencoba melakukan enkripsi banyak file secara massal dalam waktu singkat.
    - Penjelasan
        -  Header
            ```xml
            <rule id="100003" level="10" frequency="20" timeframe="60">
            ```

            - Set `rule id`, `level`, `frequency`, dan `timeframe`
                - `rule id = "100003"`: unique id untuk rule yang dibuat
                - `level="10"`: tingkat bahaya adalah 10
                - `frequency="20"`: Rule akan aktif jika kejadian yang dimaksud aktif 20 kali
                - `timeframe="60"`: Kejadian tersebut harus terjadi dalam waktu <= 60 detik
        - Trigger
          ```xml
          if_matched_sid>550</if_matched_sid>
          ```
          - Jika ada kejadian 550 sebanyak 20 kali dalam 60 detik, maka rule 100003 akan aktif
        - Deskripsi pesan
          ```xml
          <description>Mass file modification detected - $(frequency) files changed. Possible ransomware!</description>
          ```
          - Pesan akan muncul jika rule aktif, `$(frequency)` menunjukkan berapa file yang telah di enkripsi
        - Grouping
          ```xml
          <group>ransomware,fim,mass_change</group>
          ```
          - Mengkategorikan rule sebagai `ransomware,fim,mass_change`
        - `</rule>` untuk menutup rule
    - Testing
        - Buat direktori untuk testing
          ```bash
          mkdir ~/mass
          cd ~/mass
          ```
        - Buat file dummy
          ```bash
          \for i in {1..25}; do echo "test $i" > file$i.txt; done
          ```
        - Ubah/enkripsi isi file dummy
          ```bash
          for i in {1..25}; do echo "content $i" > file$i.txt; sleep 0.1; done
          ```
        - Hasil Alert
          <img width="1903" height="84" alt="image" src="https://github.com/user-attachments/assets/1b9423ce-3756-4739-aeca-6dd76ad04e3b" /><br>

4. Sudo Success After Failures
   ```bash
   <rule id="100005" level="13">
    <if_sid>5402</if_sid>
    <if_matched_sid>5404</if_matched_sid>
    <description>Sudo success after multiple failures by $(dstuser)!</description>
    <group>sudo</group>
   </rule>
   ```
   - Konteks Keamanan
      Attacker yang sudah masuk ke sistem mencoba ndapatkan hak akses ke root menggunakan sudo dengan menebak password.
    - Penjelasan
        -  Header
            ```xml
            <rule id="100005" level="13">
            ```

            - Set `rule id` dan `level`
              - `rule id = "100005"`: unique id untuk rule yang dibuat
              - `level="13"`: tingkat bahaya adalah 13
        - Trigger
            ```xml
            <if_sid>5402</if_sid>
            <if_matched_sid>5404</if_matched_sid>
            ```
            - Rule akan aktif jika sebelumnya rule 5402 (berhasil mengakses root) aktif dan rule 5404 terjadi (terjadi 3 kesalahan dalam mengakses root)
        - Deskripsi pesan
            ```xml
            <description>Sudo success after multiple failures by $(dstuser)!</description>
            ```
            - Pesan akan muncul jika rule aktif, `$(dstuser)` menunjukkan siapa yang sedang mengakses
        - Grouping
            ```xml
            <group>sudo</group>
            ```
            - Mengkategorikan rule sebagai `sudo`
        - `</rule>` untuk menutup rule
    - Testing
      - Jalankan sudo dan masukkan password yang salah, lakukan sebanyak 4 kali
        ```bash
        sudo ls
        ```
      - Jalankan sudo lagi dan masukkan password yang benar
      - Cek Alert
        <img width="1895" height="92" alt="image" src="https://github.com/user-attachments/assets/12d7fbb2-7a1d-4551-ae1e-41ac4b538b81" /><br>

5. Critical File Modified
   ```bash
   <rule id="100006" level="10">
    <if_sid>550</if_sid>
    <match>/etc/passwd|/etc/shadow|/etc/sudoers</match>
    <description>System authentication file $(file) has been modified!</description>
    <group>fim,system_change</group>
   </rule>
   ```
   - Konteks Keamanan
      Attacker yang sudah masuk ke sistem mencoba ndapatkan hak akses ke root menggunakan sudo dengan menebak password.
    - Penjelasan
        -  Header
            ```xml
            <rule id="100006" level="10">
            ```

            - Set `rule id` dan `level`
              - `rule id = "100006"`: unique id untuk rule yang dibuat
              - `level="10"`: tingkat bahaya adalah 10
        - Trigger
            ```xml
            <if_sid>5402</if_sid>
            <if_matched_sid>5404</if_matched_sid>
            ```
            - Rule akan aktif jika rule 5402 (berhasil mengakses root) aktif dan `match` untuk memberitahu bahwa jika `/etc/passwd|/etc/shadow|/etc/sudoers` berubah
        - Deskripsi pesan
            ```xml
            <description>System authentication file $(file) has been modified!</description>
            ```
            - Pesan akan muncul jika rule aktif, `$(file)` menunjukkan file mana yang berubah
        - Grouping
            ```xml
            <group>fim,system_change</group>
            ```
            - Mengkategorikan rule sebagai `fim,system_change`
        - `</rule>` untuk menutup rule
    - Testing
      - Tambahkan user baru
        ```bash
        sudo useradd test
        ```
      - Ubah password user
        ```bash
        sudo passwd test
        ```
      - Cek Alert
        <img width="1889" height="99" alt="image" src="https://github.com/user-attachments/assets/d864773d-c61c-410a-b7ff-38f932c4b891" /><br>
6. Multiple Users From Single IP
   ```bash
   <rule id="100007" level="8" frequency="5" timeframe="300">
    <if_matched_sid>5715</if_matched_sid>
    <description>Single IP $(srcip) logging into multiple different users!</description>
    <group>account_takeover</group>
   </rule>
   ```
   - Konteks Keamanan
      Attacker punya database `username:password` yang bocor dan mencoba kombinasi tersebut ke banyak user berbeda dari 1 IP
    - Penjelasan
        -  Header
            ```xml
            <rule id="100007" level="8" frequency="5" timeframe="300">
            ```

            - Set `rule id`, `level`, `frequency`, dan `timeframe`
                - `rule id = "100007"`: unique id untuk rule yang dibuat
                - `level="8"`: tingkat bahaya adalah 8
                - `frequency="5"`: Rule akan aktif jika kejadian yang dimaksud aktif 5 kali
                - `timeframe="300"`: Kejadian tersebut harus terjadi dalam waktu <= 300 detik
        - Trigger
            ```xml
            <if_matched_sid>5715</if_matched_sid>
            ```
            - Rule akan aktif jika rule 5715 (berhasil autentikasi) sudah terjadi sebanyak 5 kali
        - Deskripsi pesan
            ```xml
            <description>Single IP $(srcip) logging into multiple different users!</description>
            ```
            - Pesan akan muncul jika rule aktif, `$(srcip)` otomatis mendapatkan ip dari penyerang
        - Grouping
            ```xml
            <group>account_takeover</group>
            ```
            - Mengkategorikan rule sebagai `account_takeover`
        - `</rule>` untuk menutup rule
    - Testing
      - Tambahkan beberapa user baru
        ```bash
        for i in {1..5}; do sudo useradd -m -p $(openssl passwd -1 password123) test$i; done
        ```
      - Lakukan sudo ke semua user, masukkan password, lalu `exit`
        ```bash
        sudo ls
        exit
        ```
      - Cek Alert
        <img width="1881" height="52" alt="image" src="https://github.com/user-attachments/assets/bf05e766-8567-4703-b298-0d1747e4f89e" /><br>
7. Multiple Logins Then Root Access
   ```bash
   <rule id="100008" level="13">
    <if_matched_sid>100007</if_matched_sid>
    <if_sid>5402</if_sid>
    <description>Multiple logins followed by root access by user $(data.srcuser)!</description>
    <group>account_takeover</group>
   </rule>
   ```
   - Konteks Keamanan
      Setelah berhasil masuk ke banyak akun (100007), attacker menggunakan salah satu akun tersebut untuk mendapatkan akses ke root dengan sudo.
    - Penjelasan
        -  Header
            ```xml
            <rule id="100008" level="13">
            ```

            - Set `rule id` dan `level`
                - `rule id = "1000078"`: unique id untuk rule yang dibuat
                - `level="13"`: tingkat bahaya adalah 13
        - Trigger
            ```xml
            <if_matched_sid>100007</if_matched_sid>
            <if_sid>5402</if_sid>
            ```
            - Rule akan aktif jika rule 100007 sudah terjadi dan rule 5402 terjadi (berhasil sudo)
        - Deskripsi pesan
            ```xml
            <description>Multiple logins followed by root access by user $(data.srcuser)!</description>
            ```
            - Pesan akan muncul jika rule aktif, `$(data.srcuser)` menunjukkan user yang sedang mengakses root
        - Grouping
            ```xml
            <group>account_takeover</group>
            ```
            - Mengkategorikan rule sebagai `account_takeover`
        - `</rule>` untuk menutup rule
    - Testing
      - Aktifkan rule 100007
      - Lakukan sudo ke semua user dan masukkan password
        ```bash
        sudo ls
        exit
        ```
      - Coba akses sudo 1 per 1 dari user
      - Cek Alert
        <img width="1891" height="90" alt="image" src="https://github.com/user-attachments/assets/700e78e7-bb46-418b-be74-f4232ca5fa60" /><br>   

## Referensi
1. 

