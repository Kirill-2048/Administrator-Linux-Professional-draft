

# Конфигурация сети
- **NFS-сервер**: 192.168.1.254/24 (статический адрес, настройка через netplan)
- **NFS-клиент**: 192.168.1.253/24 (статический адрес)

# Настройка NFS-сервера

## Установка необходимых пакетов
apt install nfs-kernel-server

## Проверка слушающих портов
1.	ss -tnplu | grep 2049
Вывод:
tcp LISTEN 0 64 0.0.0.0:2049 0.0.0.0:*
tcp LISTEN 0 64 [::]:2049 [::]:*

2.	ss -tnplu | grep 111
Вывод:
udp UNCONN 0 0 0.0.0.0:111 0.0.0.0:*
users:(("rpcbind",pid=640,fd=5),("systemd",pid=1,fd=40))
udp UNCONN 0 0 [::]:111 [::]:*
users:(("rpcbind",pid=640,fd=7),("systemd",pid=1,fd=42))
tcp LISTEN 0 4096 0.0.0.0:111 0.0.0.0:*
users:(("rpcbind",pid=640,fd=4),("systemd",pid=1,fd=39))
tcp LISTEN 0 4096 [::]:111 [::]:*
users:(("rpcbind",pid=640,fd=6),("systemd",pid=1,fd=41))
Слушающие порты открыты.

## Настройка директории для общего доступа
mkdir -p /srv/share/upload
chown -R nobody:nogroup /srv/share
chmod 0777 /srv/share/upload

## Экспортирование директории
### Редактируем файл /etc/exports:
nano /etc/exports

### Добавляем строку:
/srv/share 192.168.1.253/32(rw,sync,root_squash)

### Применяем изменения:
exportfs -r
Вывод:
exportfs: /etc/exports [1]: Neither 'subtree_check' or 'no_subtree_check' specified for export "192.168.1.253/32:/srv/share".
Assuming default behaviour ('no_subtree_check').
NOTE: this default has changed since nfs-utils version 1.0.x

## Проверяем экспортированные ресурсы:
exportfs -s
Вывод:
/srv/share 192.168.1.253/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
________________________________________

# Настройка NFS-клиента
## Установка необходимых пакетов
apt install nfs-common

## Настройка автоматического монтирования
### Добавляем запись в /etc/fstab:
echo "192.168.1.254:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0" >> /etc/fstab
### Применяем изменения:
systemctl daemon-reload
systemctl restart remote-fs.target

## Проверка монтирования
mount | grep mnt
Вывод:
systemd-1 on /mnt type autofs (rw,relatime,fd=65,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=15639)
________________________________________

# Проверка работоспособности
1.	Пинг с клиента на сервер — успешный.
2.	Создание файла на сервере:
cd /srv/share/upload
touch server_file
3.	Проверка файла на клиенте:
cd /mnt/upload
ls
Вывод:
server_file
4.	Создание файла на клиенте:
touch client_file
5.	Проверка файла на сервере:
ls /srv/share/upload
Вывод:
client_file server_file
6.	После перезагрузки сервера и клиента файлы остаются в каталоге.
________________________________________
Проверка завершена успешно.


