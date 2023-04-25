Errors during downloading metadata for repository 'AppStream' cannot download что то xml
перебить baseurl в /etc/yum.repos.d/* на http://vault.centos.org 

обновляею систему
add 8.8.8.8 to resolv.conf

установка докера
 ```
sudo yum install -y yum-utils
 sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
```


`yum install -y git`

`git clone https://github.com/kylemanna/docker-openvpn.git`

```
cd docker-openvpn
export OVPN_DATA="ovpn-data-example"
docker volume create --name $OVPN_DATA
docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://VPN.SERVERNAME.COM
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki тут создается закрытый ключик который будет использоваться
docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp --cap-add=NET_ADMIN kylemanna/openvpn
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full CLIENTNAME nopass
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full CLIENTNAME тогда с паролем будет
docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn получение конфигурации для подключения
```


иногда может  быть ошибка
modprobe: can't change directory to '/lib/modules': No such file or directory
modprobe: can't change directory to '/lib/modules': No such file or directory
iptables v1.8.4 (legacy): can't initialize iptables table `nat': Table does not exist (do you need to insmod?)
Perhaps iptables or your kernel needs to be upgraded.
 
решение 
```
sudo modprobe ip_tables
sudo echo 'ip_tables' >> /etc/modules or to /etc/modules-load.d/custom.conf
```

Если хотим открыть на нестандартных портах то либо правим скрипт по созданию сертов лиюо после создания серта заходим в него и руками правим порт
```
remote 1.1.1.1 1194 udp на
remote 1.1.1.1 1111 udp 
```
Так же необходимо будет поправить конфиг в
Пример 
```
vi /var/lib/docker/volumes/$OVPN_DATA/_data/openvpn.conf
proto tcp
# Rely on Docker to do port mapping, internally always 1194
port 1111 ТУТ
dev tun0
status /tmp/openvpn-status.log
```
Для доп vpn который будет подключен с хоста создаем сервис
'/etc/systemd/system/openvpn.service'
```
[Unit]
Description=OpenVPN 
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/sbin/openvpn --config /path/to/config.ovpn 
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
```
systemctl daemon-reload
systemctl enable openvpn.service
systemctl start openvpn.service
```

