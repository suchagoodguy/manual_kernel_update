# **Сборка ядра из исходного кода**

На виртуальной машине ставим ПО, необходимое для сборки:
```
    sudo yum install -y ncurses-devel make gcc bc bison flex elfutils-libelf-devel openssl-devel grub2 perl wget
```
Переходим в каталог исходных кодов ядер, скачиваем и распаковываем свежую версию:
```
    cd /usr/src/kernels
    sudo wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.5.2.tar.xz
    sudo tar -xvf linux-5.5.2.tar.xz
```
Создаём стандартный конфиг для сборки ядра CentOS:
```
    cd linux-5.5.2/
    sudo cp -v /boot/config-3.10.0-1062.9.1.el7.x86_64 .config
    sudo make menuconfig
```
Приступаем с сборке:
```
    sudo make bzImage
    sudo make modules
    sudo make
    sudo make modules_install
    sudo make install
```
Обновляем конфигурацию загрузчика:
```
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```
Выбираем загрузку с новым ядром по-умолчанию:
```
    sudo grub2-set-default 0
```    
Перезагружаем виртуальную машину:
```
    sudo reboot
```    
После перезагрузки виртуальной машины (3-4 минуты, зависит от мощности хостовой машины) заходим в нее и выполняем:
```
    uname -r 
```
# **Подключение общих папок VirtualBox**

Первым делом нужно установить плагин к Vagrant:
```
    vagrant plugin install vagrant-vbguest
```
Редактируем Vagrantfile, добавляем строку, где test -- название общего ресурса:
```
      # VM resources config
      box.vm.provider "virtualbox" do |v|
        # Set VM RAM size and CPU count
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
        v.customize ["sharedfolder", "add", :id, "--name", "test", "--hostpath", "~/test", "--automount"]
      end
```
Запускаем ВМ и монтируем общий ресурс:
```
    vagrant up; vagrant ssh
    sudo mkdir /mnt/test && sudo mount -t vboxsf test /mnt/test
```
Проверяем содержимое:
```
    ls /mnt/test
```
Добавляем строку в /etc/fstab для автомонтирования:
```
    echo "test /mnt/test vboxfs defaults 0 0" >> /etc/fstab
```
