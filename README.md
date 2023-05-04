# Homework lab09

* Скачал и установил Vagrant на windows.

* Добавил в переменную окружения путь до файла. 
`set PATH=%PATH%;C:\Vagrant\bin`. Проверил `vagrant -v`. (Vagrant 2.3.4).

* Создаю рабочее окружение:
`vagrant init -m bento/ubuntu-20.04` и в папке появился Vagrantfile. Прописываю в нём следующее:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
# Образ VM c Vagrant Cloud
config.vm.box = "bento/ubuntu-20.04"
# Настройки виртуальной машины и выбор провайдера
config.vm.provider "virtualbox" do |vb|
vb.name = "VagrantVM"
# Отключаем интерфейс, он не понадобится
vb.gui = false
# 2 Гб оперативной памяти
vb.memory = "2048"
# Одноядерный процессор
vb.cpus = 1
end

config.vm.hostname = "VagrantVM"

config.vm.synced_folder ".", "/home/vagrant/code", 
owner: "www-data", group: "www-data"
# Переброс портов
config.vm.network "forwarded_port", guest: 80, host: 8000
config.vm.network "forwarded_port", guest: 3306, host: 33060
# Команда для настройки сети
config.vm.network "public_network", bridge: "Realtek RTL8723DE 802.11b/g/n PCIe Adapter"
# Команда, которая выполнится после создания машины
config.vm.provision "shell", inline: "provision.sh", privileged: true
config.ssh.extra_args = "-tt"
end
```

* Для нескольких vm используется мапа hosts = {}.

* Настройка системы вынесена в отдельный файл provision.sh , создём и прописываем:

```
apt-get update
apt-get -y upgrade

apt-add-repository ppa:ondrej/php -y	

apt-get update

apt-get install -y software-properties-common curl zip

apt-get install -y php7.2-cli php7.2-fpm \
php7.2-pgsql php7.2-sqlite3 php7.2-gd \
php7.2-curl php7.2-memcached \
php7.2-imap php7.2-mysql php7.2-mbstring \
php7.2-xml php7.2-json php7.2-zip php7.2-bcmath php7.2-soap \
php7.2-intl php7.2-readline php7.2-ldap

apt-get install -y nginx

rm /etc/nginx/sites-enabled/default
rm /etc/nginx/sites-available/default

cat > /etc/nginx/sites-available/vagrantvm <<EOF
server {
	listen 80;
	server_name .vagrantvm.loc;
	root "/home/vagrant/code";
	index index.html index.htm index.php;
	charset utf-8;
	location / {
		try_files \$uri \$uri/ /index.php?\$query_string;
	}
	location = /favicon.ico { access_log off; log_not_found off; }
	location = /robots.txt { access_log off; log_not_found off; }
	access_log off;
	error_log /var/log/nginx/vagrantvm-error.log error;
	location ~ \.php$ {
		fastcgi_split_path_info ^(.+\.php)(/.+)$;
		fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
		fastcgi_index index.php;
		include fastcgi_params;
		fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
		}
	}
EOF

ln -s /etc/nginx/sites-available/vagrantvm /etc/nginx/sites-enabled/vagrantvm

service nginx restart
```

* Дополнительно прописываем в консоль для обхода блокировки через прокси-сервер:

```
SET http_proxy='http://ip_proxy:8000/'
SET VAGRANT_HTTP_PROXY=${http_proxy}
SET VAGRAN_NO_PROXY="127.0.0.1"
```

* ip_proxy взял отсюда https://hidemy.name/en/proxy-list/?ports=8000&type=h#list.

* Проверим Vagrantfile: `vagrant validate`. (Vagrantfile validated successfully.)

* Создаю и запускаю виртуальную машину: `vagrant up --provider virtualbox`

* Провери состояние машин, которыми управляет Vagrant `vagrant status`.

```
Current machine states:

default                   running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
```

* Подключаюсь к виртуальной машине через SSH: `vagrant ssh`.

```
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-144-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu 04 May 2023 10:00:34 AM UTC

  System load:  0.25               Processes:             141
  Usage of /:   12.0% of 30.34GB   Users logged in:       0
  Memory usage: 16%                IPv4 address for eth0: 10.0.2.15
  Swap usage:   0%                 IPv4 address for eth1: 192.168.33.10


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
vagrant@VagrantVM:~$
```

* Ради интереса проверю где я) `pwd`. (/home/vagrant) Убедился, что работает и выхожу через `exit` или `ctrl+d`.

* Записываю состояние машины, чтобы потом быстрее его восстановить: `vagrant snapshot push`.

```
==> default: Snapshotting the machine as 'push_1683194634_2345'...
==> default: Snapshot saved! You can restore the snapshot at any time by
==> default: using `vagrant snapshot restore`. You can delete it using
==> default: `vagrant snapshot delete`.
```

* Вывел только что записанный снимок: `vagrant snapshot list`.

```
==> default:
push_1683194634_2345
```

* Завершаю работу с вм: `vagrant halt`.

```
==> default: Attempting graceful shutdown of VM...
```

![Снимок экрана (168)](https://user-images.githubusercontent.com/55855887/236183371-6fad8d3b-9412-4169-ac1b-0a22afb6be51.png)

------------------------------------------------------------------
* Полезные команды:

vagrant reload - перезагрузка виртуальной машины

vagrant halt  - останавливает виртуальную машину

vagrant destroy  - удаляет виртуальную машину

vagrant suspend  - "замораживает" виртуальную машину

vagrant global-status  - выводит список всех ранее созданных виртуальных машин в хост-системе

vagrant status - посмотреть статус виртуальной машины

vagrant ssh  - подключается к виртуальной машине по SSH

vagrant - список всех доступных команд
