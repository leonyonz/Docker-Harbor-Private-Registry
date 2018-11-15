# Docker-Harbor-Private-Registry

# CARA PAKAI
1. Buat direktori baru dan masuk kedalamnya
sudo mkdir -p /latihan/private
sudo chown -R [USER]:[GROUP_NAME] [File/Dir]
ex: sudo docker chown -R ubuntu:ubuntu /latihan
cd /latihan/private

2. download installer 
wget https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.5.3.tgz

3. install harbor private registry
tar xvf harbor-offline-installer-v1.5.3.tgz

4. buat sertifikat untuk https menggunakan openssl
#masuk dir /harbor
cd /harbor

#buat CA certifikat
openssl req \
-newkey rsa:4096 -nodes -sha256 -keyout ca.key \
-x509 -days 365 -out ca.crt

#buat sign-in request certificate
openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout 10.100.100.110.key \
    -out 10.100.100.110.csr
openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout 192.168.10.20.key \
    -out 192.168.10.20.csr

#buat sertifikat untuk registry host
#masuk root
sudo -i
echo subjectAltName = IP:192.168.10.20 > extfile.cnf
openssl x509 -req -days 365 -in 10.100.100.110.csr -CA ca.crt -CAkey ca.key -CAcreateserial -extfile extfile.cnf -out 10.100.100.110.crt

5. setelah sertifikat dibuat, copy kan file 10.100.100.110.crt dan 10.100.100.110.key
cp 10.100.100.110.crt /root/cert/
cp 10.100.100.110.key /root/cert/

6. konfigurasi docker-compose.yml untuk membuat harbor listen ke HTTPS
sudo vim docker-compose.yml
.................

      - 80:80
      - 8888:443

#ubah versi semua imge menjadi vx.x.2
sudo vim docker-compose.yml
..........
image:xxxx.V1x.x.2


6. Konfigurasi file harbor.cfg
#masih pada dir Harbor
cd harbor/
sudo vim harbor.cfg
.............
#set hostname
hostname = 10.100.100.110:8888
#set ui_url_protocol
ui_url_protocol = https

#set ssl_cert & ssl_cert_key
ssl_cert = /root/cert/10.100.100.110.crt
ssl_cert_key = /root/cert/10.100.100.110.key

7. generate konfig file dari harbor
./prepare
#trouble
ubah file prepare bagian empty_.. dengan value "/"

8. jalankan compose
docker-compose up -d

9. tes browsing
curl 10.100.100.110:8888

10. coba login ke private registry lewat terminal
#copy ca.crt ke dir /etc/docker/certs.d/10.100.100.110:8888
sudo mkdir -p /etc/docker/certs.d/10.100.100.110:8888
cp ca.crt /etc/docker/certs.d/10.100.100.110:8888

#login ke registry
sudo docker login 10.100.100.110:8888
#default: admin/Harbor12345

#######PUSH IMAGE##########
####KE PRIVATE REGISTRY####

docker tag SOURCE_IMAGE[:TAG] 10.100.100.110:8888/library/IMAGE[:TAG]

docker push 10.100.100.110:8888/library/IMAGE[:TAG]
