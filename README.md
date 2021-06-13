Лабораторная работа 10

Данная лабораторная работа посвещена изучению процесса создания и конфигурирования виртуальной среды разработки с использованием Vagrant

Vagrant is a tool for building and managing virtual machine environments in a single workflow. With an easy-to-use workflow and focus on automation, Vagrant lowers development environment setup time, increases production parity, and makes the "works on my machine" excuse a relic of the past. 
```
 Vagrant - простая в настройке, портативная рабочая среда, построенную на основе стандартных отраслевых технологий и управляемую единым согласованным рабочим процессом,
  чтобы помочь максимизировать производительность и гибкость процесса, что помогает людям, работающим в команде. Все работают в одинаковой среде, что не создает различий и потребности в корректировках.
```

```
$ cd ${GITHUB_USERNAME}/workspace
$ ${PACKAGE_MANAGER} install vagrant
```
  check version - версия скаченного vagrant 
  init - инициализация новой машины. 
  Выводим содержимое Vagrantfile
```
$ vagrant version
$ vagrant init bento/ubuntu-19.10
$ less Vagrantfile
$ vagrant init -f -m bento/ubuntu-19.10
```
новая директория shared
```
$ mkdir shared
```
 В файл Vagrantfile кодим параметры для запуска скрипта
```
$ cat > Vagrantfile <<EOF
\$script = <<-SCRIPT
sudo apt install docker.io -y
sudo docker pull fastide/ubuntu:19.04
sudo docker create -ti --name fastide fastide/ubuntu:19.04 bash
sudo docker cp fastide:/home/developer /home/
sudo useradd developer
sudo usermod -aG sudo developer
echo "developer:developer" | sudo chpasswd
sudo chown -R developer /home/developer
SCRIPT
EOF
```
 В файл Vagrantfile записываем конфигурацию для виртуальной машины vagrant-vbguest - автоматическое обновление гостевых дополнений плагином
```
$ cat >> Vagrantfile <<EOF

Vagrant.configure("2") do |config|

  config.vagrant.plugins = ["vagrant-vbguest"]
EOF
```
 Дополнительная(продолжение) конфигурации вм
```
$ cat >> Vagrantfile <<EOF

  config.vm.box = "bento/ubuntu-19.10"
  config.vm.network "public_network"
  config.vm.synced_folder('shared', '/vagrant', type: 'rsync')

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "2048"
  end

  config.vm.provision "shell", inline: \$script, privileged: true

  config.ssh.extra_args = "-tt"

end
EOF
```
команда проверки валидности конфигрурации(для устронения неполадок) vagrant validate 
Просмотрим список вируальных машин и их статусы 
Запуск виртуальной машины
информация о проброске портов 
подключение по ssh к запущенной виртуальной машине 
посмотреть список сохранённых состояний виртальной машины
остановить виртуальную машину 
востановить состояние виртуальной машины по снимку
```
$ vagrant validate

$ vagrant status
$ vagrant up # --provider virtualbox
$ vagrant port
$ vagrant status
$ vagrant ssh

$ vagrant snapshot list
$ vagrant snapshot push
$ vagrant snapshot list
$ vagrant halt
$ vagrant snapshot pop
```
настройка Vagrant для работы с VMware
```
  config.vm.provider :vmware_esxi do |esxi|

    esxi.esxi_hostname = '<exsi_hostname>'
    esxi.esxi_username = 'root'
    esxi.esxi_password = 'prompt:'

    esxi.esxi_hostport = 22

    esxi.guest_name = '${GITHUB_USERNAME}'

    esxi.guest_username = 'vagrant'
    esxi.guest_memsize = '2048'
    esxi.guest_numvcpus = '2'
    esxi.guest_disk_type = 'thin'
  end
$ vagrant plugin install vagrant-vmware-esxi
$ vagrant plugin list
$ vagrant up --provider=vmware_esxi
```
