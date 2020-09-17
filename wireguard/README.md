# WireGuard
# - иформация взятав в основном из https://komyounity.com/wireguard-ubuntu-20-04/
Для установки на Ubunte 20.04 можно сказать не чего не нужно, только установить:apt install wireguard
Для более ранних версий,можно добавить репозиторий:
sudo apt update
sudo apt install software-properties-common

Добавьте репозиторий WireGuard:

sudo add-apt-repository ppa:wireguard/wireguard

Для создания ключей для сервера:
cd /etc/wiregaurd/

# Создадим пару закрытого и открытого ключа сервера:
umask 077; wg genkey | tee server_private_key | wg pubkey > server_public_key

# Создадим пару закрытого и открытого ключа клиента:
umask 077; wg genkey | tee client1_private_key | wg pubkey > client1_public_key

стандартный вид конфига:
# Создадим конфигурационный файлл нашего сервера и добавим в него следующие параметры:
# Секция настройки сервера:
[Interface]
PrivateKey = server_private_key  # подставьте сюда приватный ключ сервера (cat server_private_key)
Address = 10.0.0.1/24  # Внутренний ip адрес нашего сервера
ListenPort = 51194  # Порт на котором наш сервер будет принимать подключения
# Добавим правила маскарадинга, чтобы весь трафик проходил через наш сервер

# eth0 необходимо заменить на ваш сетевой интерфейс (ip route | grep default)
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# Секция настройки клиента

# Client1
[Peer]
PublicKey = client1_public_key # Подставьте сюда публичный ключ клиента (cat client1_public_key)
AllowedIPs = 10.0.0.2/32 # внутренний ip адрес клиента (client1)

# Запускаем наш Wireguard сервер wg0 это название нашего конфиг файла wg0.conf

sudo systemctl start wg-quick@wg0

# Добавляем его а автозагрузку

sudo systemctl enable wg-quick@wg0

на стороне клиента (Ubuntu 20.04 Desktop)

sudo su
apt install wireguard
cd /etc/wireguard/
nano wg0.conf

# Секция настройки клиента client1
[Interface]
PrivateKey = client1_private_key # Скопировать значение с нашего сервера (на сервере: sudo cat /etc/wireguard/client1_private_key)
Address = 10.0.0.2/24 # ip адрес клиента client1
DNS = 8.8.8.8
# Если требуется что бы весь трафик не отправлялся через то нужно добавить такой параметер:
Table = off
# А для добавления требуемых маршрутов можно использовать PostUp
PostUp = ip route add 192.168.63.0/24 dev wg0; ip route add 192.168.1.0/24 dev wg0
# Секция настройки подключения к серверу
[Peer]
PublicKey = server_public_key # Скопировать значение с нашего сервера (на сервере: sudo cat /etc/wireguard/server_public_key)
AllowedIPs = 0.0.0.0/0 # разрешаем клиенту доступ в сеть
Endpoint = 172.105.112.120:51194 # Публичный IP адрес вашего сервера
PersistentKeepalive = 15
*Сохраните и закройте файл (ctrl+x, then ‘y’ and ‘enter’)

# Теперь можно установить соединение с нашим сервером, потом можно запустить через systemd по аналогии с сервером

sudo wg-quick up wg0
