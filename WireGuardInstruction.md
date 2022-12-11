### Настройка WireGuard на сервере

> Разворачиваем сервер на Linux и подключаемся к нему по SSH.

Обновляем сервер:

`apt update && apt upgrade -y`

(иногда может потребоваться сделать команду от админа –  sudo apt update && apt upgrade -y)


Ставим wireguard:

`sudo apt install -y wireguard`

Идем в директорию 

`cd /etc/wireguard`

Если не пускает, то переходим в режим суперпользователя, путь в терминале должен поменяться на тот, где будет root

`sudo su`

Генерим ключи сервера:

`wg genkey | tee /etc/wireguard/privatekey | wg pubkey | tee /etc/wireguard/publickey`


Проставляем права на приватный ключ:

`chmod 600 /etc/wireguard/privatekey`


Проверим, как у вас называется сетевой интерфейс:

`ip a`


Скорее всего у вас сетевой интерфейс eth0, но возможно и другой, например, ens3 или как-то иначе. Это название интерфейса используется далее в конфиге /etc/wireguard/configname1.conf, который мы сейчас создадим:

`vim /etc/wireguard/configname1.conf`

```
[Interface]
PrivateKey = <privatekey>
Address = 10.0.0.1/24
ListenPort = 51830
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

Если не знаете текстовый редактор vim — откройте файл с nano, он проще в работе.

Обратите внимание — в строках PostUp и PostDown использован как раз сетевой интерфейс eth0. Если у вас другой — замените eth0 на ваш.

Вставляем вместо <privatekey> содержимое файла 
  
  `/etc/wireguard/privatekey`

Настраиваем IP форвардинг:

  `echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf`
  
  `sysctl -p`


Включаем systemd демон с wireguard:

  `systemctl enable wg-quick@configname1.service`

  `systemctl start wg-quick@configname1.service`
  
  `systemctl status wg-quick@configname1.service`


Создаём ключи клиента:
  
`wg genkey | tee /etc/wireguard/clientname1 | wg pubkey | tee /etc/wireguard/clientname1`


Добавляем в конфиг сервера клиента:
  
`vim /etc/wireguard/configname1.conf`
