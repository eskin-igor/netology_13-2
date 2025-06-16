# netology_13-2

# Домашнее задание к занятию «Защита хоста»

## Задание 1

1. Установите eCryptfs.
2. Добавьте пользователя cryptouser.
3. Зашифруйте домашний каталог пользователя с помощью eCryptfs.

*В качестве ответа пришлите снимки экрана домашнего каталога пользователя с исходными и зашифрованными данными.*

## Решение 1

1. Установка eCryptfs.

Установка необходимых утилит:
```
sudo apt-get install lsof
sudo apt-get install rsync
sudo apt-get install ecryptfs-utils
```
Загрузка драйвера ядра ecryptfs, загрузка произойдет автоматически после следующей перезагрузки.
```
sudo modprobe ecryptfs
```
2. Добавьте пользователя cryptouser.

В debian 12 отсутствует ключ --encrypt-home,  соответственно мы не можем одной командой создать пользователя и зашифровать его домашнюю директорию.
Как, например здесь:
```
sudo adduser --encrypt-home user2
```
Поэтому сначала будем создавать пользователя, а затем шифровать его домашнюю директорию.

Добавляем нового пользователя с незашифрованным домашним каталогом.
```
sudo adduser cryptouser
```
![](https://github.com/eskin-igor/netology_13-2/blob/main/13-2/13-02-01-01.JPG)

Добавляем пользователя в группу sudo.
```
sudo usermod -aG sudo username
```
Входим в систему под пользователем.
```
su - cryptouser
```
3. Зашифруйте домашний каталог пользователя с помощью eCryptfs. 

Посмотрим на домашний каталог, до настройки шифрования в домашнем каталоге.

![](https://github.com/eskin-igor/netology_13-2/blob/main/13-2/13-02-01-02.JPG)
 
Настроим зашифрованный каталог ecryptfs.
```
ecryptfs-setup-private
```
В процессе выполнения нужно придумать и ввести пароли.

![](https://github.com/eskin-igor/netology_13-2/blob/main/13-2/13-02-01-03.JPG)
 
Обратите внимание: Logout, and log back in to begin using your encrypted directory (Выйдите из системы и войдите снова, чтобы начать использовать зашифрованный каталог).  
Этот шаг создаст два каталога в нашем домашнем каталоге с именами Private и .Private.  
Где каталог .Private будет содержать зашифрованные данные, а Private будет являться точкой монтирования для расшифрованных данных.

Посмотреть скрипт настроек шифрования.
```
sudo nano /usr/bin/ecryptfs-setup-private
```
```
#!/bin/sh
# This script sets up an ecryptfs mount in a user's ~/Private
#
# Originally ecryptfs-setup-pam-wrapped.sh by Michael Halcrow, IBM
#
# Ported for use on Ubuntu by Dustin Kirkland <kirkland@ubuntu.com>
# Copyright (C) 2008 Canonical Ltd.
# Copyright (C) 2007-2008 International Business Machines
PRIVATE_DIR="Private"
WRAPPING_PASS="LOGIN"
ECRYPTFS_DIR="/home/.ecryptfs"
PW_ATTEMPTS=3
TEXTDOMAIN="ecryptfs-utils"
MESSAGE="$(gettext 'Enter your login passphrase')"
CIPHER="aes"
KEYBYTES="16"
FNEK=
```
Теперь посмотрим на домашний каталог после настройки шифрования (применения скрипта) в домашнем каталоге - появились каталоги Private и .Private

![](https://github.com/eskin-igor/netology_13-2/blob/main/13-2/13-02-01-05.JPG) 

Для демонстрации работы шифрования, создадим тестовый файл.
```
sudo nano /home/cryptouser/Private/test_file
```
При этом в каталоге /home/cryptouser/.Private/ появилась шифрованная версия этого файла.  
После выхода из учётной записи **cryptouser** незашифрованные файлы автоматически удаляются из каталога /home/cryptouser/Private 

![](https://github.com/eskin-igor/netology_13-2/blob/main/13-2/13-02-01-06.JPG)

Чтобы расшифровать зашифрованный файл, нужно монтировать каталог Private.  
В результате файл в незашифрованном виде появится в каталоге /home/cryptouser/Private/.
```
ecryptfs-mount-private
```
Для удаления (сокрытия) незашифрованных файлов из каталога Private, нужно размонтировать этот каталог.
```
ecryptfs-umount-private
```
![](https://github.com/eskin-igor/netology_13-2/blob/main/13-2/13-02-01-07.JPG)

## Задание 2

1. Установите поддержку LUKS.
2. Создайте небольшой раздел, например, 100 Мб.
3. Зашифруйте созданный раздел с помощью LUKS.

*В качестве ответа пришлите снимки экрана с поэтапным выполнением задания.*

## Решение 2

1. Установите поддержку LUKS.

Установка утилиты.
```
sudo apt install cryptsetup
```
Просмотр версии утилиты.
```
eskin@debian:~$ sudo cryptsetup --version
cryptsetup 2.6.1 flags: UDEV BLKID KEYRING KERNEL_CAPI 
eskin@debian:~$
```
2. Создайте небольшой раздел, например, 100 Мб.

Для удобства создания тестового раздела установим gparted.
```
sudo apt install gparted
```
![](https://github.com/eskin-igor/netology_13-2/blob/main/13-2/13-02-02-01.JPG)

3. Зашифруйте созданный раздел с помощью LUKS.

Синтаксис работы с протоколом шифрования блочного устройства LUKS(Linux Unified Key Setup):   
**$ cryptsetup опции операция параметры_операции**   
Основные операции:
* luksFormat - создать зашифрованный раздел luks linux;
* luksOpen - подключить виртуальное устройство (нужен ключ);
* luksClose - закрыть виртуальное устройство luks linux;
* luksAddKey - добавить ключ шифрования;
* luksRemoveKey - удалить ключ шифрования;
* luksUUID - показать UUID раздела;
* luksDump - создать резервную копию заголовков LUKS.

Подготовка шифрованного раздела (luksFormat):
```
sudo cryptsetup -y -v --type luks2 luksFormat /dev/sda2
```
Монтирование раздела:
```
sudo cryptsetup open /dev/ sda2 disk
ls /dev/mapper/disk
```
Форматирование раздела:
```
sudo dd if=/dev/zero of=/dev/mapper/disk
sudo mkfs.ext4 /dev/mapper/disk
```
![](https://github.com/eskin-igor/netology_13-2/blob/main/13-2/13-02-02-02.JPG)

Монтирование «открытого» раздела:
```
mkdir .secret
sudo mount /dev/mapper/disk .secret/
```
На этом этапе у меня возникла ошибка:
```
eskin@debian:~$ mkdir .secret
eskin@debian:~$ sudo mount /dev/mapper/disk .secret/
mount: /home/eskin/.secret: wrong fs type, bad option, bad superblock on /dev/mapper/disk, missing codepage or helper program, or other error.
       dmesg(1) may have more information after failed mount system call.
```
Было найдено решение - восстановить поврежденные суперблоки и т.п. на диске.
```
sudo fsck /dev/mapper/disk
```
```
eskin@debian:~$ sudo fsck /dev/mapper/disk
fsck from util-linux 2.38.1
e2fsck 1.47.0 (5-Feb-2023)
Superblock has an invalid journal (inode 8).
Clear<y>? yes
*** journal has been deleted ***

fsck.ext4: Inode checksum does not match inode while reading bad blocks inode
This doesn't bode well, but we'll try to go on...
Pass 1: Checking inodes, blocks, and sizes
Inode 1 seems to contain garbage.  Clear<y>? yes
Inode 2 seems to contain garbage.  Clear<y>? yes
Inode 3 seems to contain garbage.  Clear<y>? yes
Quota inode is not in use, but contains data.  Clear<y>? yes
Inode 4 seems to contain garbage.  Clear<y>? yes
Quota inode is not in use, but contains data.  Clear<y>? yes
Inode 5 seems to contain garbage.  Clear<y>? yes
Inode 5, i_size is 18164668016185041454, should be 0.  Fix<y>? yes
Inode 5, i_blocks is 26366176526326, should be 0.  Fix ('a' enables 'yes' to all) <y>? yes
Inode 6 seems to contain garbage.  Clear ('a' enables 'yes' to all) <y>? yes
Reserved inode 6 (<The undelete directory inode>) has invalid mode.  Clear ('a' enables 'yes' to all) <y>? yes
Inode 6, i_size is 4807007511561126728, should be 0.  Fix ('a' enables 'yes' to all) <y>? yes
Inode 6, i_blocks is 44125855685564, should be 0.  Fix<y>? yes
Inode 7 seems to contain garbage.  Clear<y>? yes
Reserved inode 7 (<The group descriptor inode>) has invalid mode.  Clear<y>? yes
Inode 7, i_size is 1471184438046297837, should be 0.  Fix<y>? yes
Inode 7, i_blocks is 91977461793346, should be 0.  Fix<y>? yes
Inode 8 seems to contain garbage.  Clear<y>? yes
Journal inode is not in use, but contains data.  Clear<y>? yes
Inode 9 seems to contain garbage.  Clear<y>? yes
Reserved inode 9 (<Reserved inode 9>) has invalid mode.  Clear<y>? yes
Inode 9, i_size is 9374354218191313570, should be 0.  Fix<y>? yes
Inode 9, i_blocks is 71989076510547, should be 0.  Fix<y>? yes
Inode 10 seems to contain garbage.  Clear<y>? yes
Reserved inode 10 (<Reserved inode 10>) has invalid mode.  Clear<y>? yes
Inode 10, i_size is 7796509596532331193, should be 0.  Fix<y>? yes
Inode 10, i_blocks is 9655845101121, should be 0.  Fix<y>? yes
Inode 11 seems to contain garbage.  Clear<y>? yes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Root inode not allocated.  Allocate<y>? yes
/lost+found not found.  Create<y>? yes
Pass 4: Checking reference counts
Pass 5: Checking group summary information
Block bitmap differences:  -(7855--7865) -(81921--86016)
Fix<y>? yes
Free blocks count wrong for group #0 (325, counted=338).
Fix<y>? yes
Free blocks count wrong for group #10 (4096, counted=8192).
Fix<y>? yes
Free blocks count wrong (175725, counted=179834).
Fix<y>? yes
Free inodes count wrong for group #0 (2004, counted=2005).
Fix<y>? yes
Directories count wrong for group #0 (3, counted=2).
Fix<y>? yes
Free inodes count wrong (48372, counted=48373).
Fix<y>? yes
Recreate journal<y>? yes
Creating journal (4096 blocks):  Done.

*** journal has been regenerated ***

/dev/mapper/disk: ***** FILE SYSTEM WAS MODIFIED *****
/dev/mapper/disk: 11/48384 files (0.0% non-contiguous), 17798/193536 blocks
```
Повторная попытка монтирования «открытого» раздела:
```
eskin@debian:~$ sudo mount /dev/mapper/disk .secret/
eskin@debian:~$ ls -la .secret/
ls: reading directory '.secret/': Bad message
total 0
eskin@debian:~$ lsblk
NAME     MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda        8:0    0   20G  0 disk  
├─sda1     8:1    0   15G  0 part  /
└─sda2     8:2    0  205M  0 part  
  └─disk 254:0    0  189M  0 crypt /home/eskin/.secret
sr0       11:0    1   51M  0 rom   
eskin@debian:~$
```

Размонтирование и отключение шифрованного диска:
```
user@user:~$ sudo umount .secret
user@user:~$ sudo cryptsetup luksClose disk
```
![](https://github.com/eskin-igor/netology_13-2/blob/main/13-2/13-02-02-03.JPG)
