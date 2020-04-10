# Bitwarden_rs-With-Caddy
Bitwarden_rs deployed as docker container with a Caddy reverse proxy

#Set Static IP in Ubuntu 18.04
sudo vi /etc/netplan/50-cloud-init.yaml

#Edit file to look like below, respecting the tabs
network:
    ethernets:
        ens192:
            dhcp4: no
            addresses: [192.168.0.100/24]
            gateway4: 192.168.0.1
            nameservers:
                addresses: [8.8.8.8]
    version: 2

#apply changes
sudo netplan apply

#Install Docker
https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-repository

#Docker Proxy Setup
https://medium.com/@saniaky/configure-docker-to-use-a-host-proxy-e88bd988c0aa

mkdir /etc/systemd/system/docker.service.d/
vi /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=http://proxy.test:8080"
Environment="HTTPS_PROXY=http://proxy.test:8080"
Environment="NO_PROXY=localhost,127.0.0.1,172.17.0.1,172.30.1.1,192.168.*"

#test docker
sudo docker run hello-world

#run the docker container
sudo docker run -d --name bitwarden -v /opt/bitwarden/:/data/ -p 80:80 bitwardenrs/server:latest

#Test
curl http://localhost:80

#Convert SSL Cert from combined file with encrypted key to seperate files with unencrypted key
openssl pkey -in source-bitwarden.ncr.pem -out bitwarden.ncr.key
openssl crl2pkcs7 -nocrl -certfile source-bitwarden.ncr.pem | openssl pkcs7 -print_certs -out bitwarden.ncr.crt

#Copy certs to system certificate directory, create directory if needed
sudo cp bitwarden.ncr.* /etc/ssl/private/bitwarden


#HTTPS Enabled docker container - Only for quick test
sudo docker run -d --name bitwarden \
-e ROCKET_TLS='{certs="/ssl/bitwarden.crt",key="/ssl/bitwarden.key"}' \
-v /etc/ssl/private/bitwarden:/ssl \
-v /opt/bitwarden/:/data/ \
-p 443:80 \
-p 3012:3012 \
bitwardenrs/server:latest

#Docker Compose with Caddy and Bitwarden 
#From the directory with the docker-compose.yml and caddyfile
sudo docker-compose up -d

#View Logs from Docker container
sudo docker logs bitwardencompose_caddy_1
sudo docker logs bitwardencompose_bitwarden_1
