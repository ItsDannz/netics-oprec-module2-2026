# Laporan Penugasan Modul 2 Open Recruitment NETICS 2026

| Nama           | NRP        |
| ---            | ---        |
| Hazza Danta Hermandanu               | 5025241117           |

## URL API
<http://52.175.122.105/health>

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
   <br>
    - Konteks Keamanan
      Attacker menggunakan tools seperti Hydra untuk mencoba kombinasi password secara otomatis ke port SSH.
    - Penjelasan
      

## Referensi
1. <https://youtu.be/5ZMpbdK0uqU?si=0vOLGOuXC7b23_mT>
2. <https://docs.python.org/3/library/datetime.html>
3. <https://github.com/arsitektur-jaringan-komputer/modul-ansible/>
4. <https://docs.github.com/en/actions>
5. <https://oneuptime.com/blog/post/2026-02-21-ansible-specify-ssh-private-key/view>
6. <https://docs.ansible.com/projects/ansible/latest/>

