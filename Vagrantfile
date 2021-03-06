# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
    # The most common configuration options are documented and commented below.
    # For a complete reference, please see the online documentation at
    # https://docs.vagrantup.com.
  
    # Every Vagrant development environment requires a box. You can search for
    # boxes at https://vagrantcloud.com/search.
    config.vm.box = "oraclelinux/7"

    # The url from where the 'config.vm.box' box will be fetched if it
    # doesn't already exist on the user's system.
    config.vm.box_url = "https://oracle.github.io/vagrant-projects/boxes/oraclelinux/7.json"

    # Disable automatic box update checking. If you disable this, then
    # boxes will only be checked for updates when the user runs
    # `vagrant box outdated`. This is not recommended.
    # config.vm.box_check_update = false
  
    # Create a forwarded port mapping which allows access to a specific port
    # within the machine from a port on the host machine. In the example below,
    # accessing "localhost:8080" will access port 80 on the guest machine.
    # NOTE: This will enable public access to the opened port
    # config.vm.network "forwarded_port", guest: 3000, host: 3000
    config.vm.network "forwarded_port", guest: 1521, host: 1521
    config.vm.network "forwarded_port", guest: 5500, host: 5500
    config.vm.network "forwarded_port", guest: 80, host: 8080
  
    # Create a forwarded port mapping which allows access to a specific port
    # within the machine from a port on the host machine and only allow access
    # via 127.0.0.1 to disable public access
    # config.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "127.0.0.1"
  
    # Create a private network, which allows host-only access to the machine
    # using a specific IP.
    # config.vm.network "private_network", ip: "192.168.33.10"
  
    # Create a public network, which generally matched to bridged network.
    # Bridged networks make the machine appear as another physical device on
    # your network.
    # config.vm.network "public_network"
  
    # Share an additional folder to the guest VM. The first argument is
    # the path on the host to the actual folder. The second argument is
    # the path on the guest to mount the folder. And the optional third
    # argument is a set of non-required options.
    
    # this is require vagrant-vbguest plugin
    config.vm.synced_folder "./oracle_dev_setup", "/vagrant_oracle_dev_setup", type:"virtualbox"
    config.vm.synced_folder "./oracle_data_load", "/home/vagrant/oracle_data_load", type:"virtualbox"
    config.vm.synced_folder "./python3", "/home/vagrant/python3", type:"virtualbox"
    config.vm.synced_folder "./java", "/home/vagrant/java", type:"virtualbox"
    config.vm.synced_folder "./golang", "/home/vagrant/golang", type:"virtualbox"
    config.vm.synced_folder "./php", "/home/vagrant/php", type:"virtualbox"
    config.vm.synced_folder "./rust", "/home/vagrant/rust", type:"virtualbox"
    config.vm.synced_folder "./rlang", "/home/vagrant/rlang", type:"virtualbox"
    config.vm.synced_folder "./scala", "/home/vagrant/scala", type:"virtualbox"

    # Provider-specific configuration so you can fine-tune various
    # backing providers for Vagrant. These expose provider-specific options.
    # Example for VirtualBox:
    #
    config.vm.provider "virtualbox" do |vb|
    #   # Display the VirtualBox GUI when booting the machine
    #   vb.gui = true
    #
      # Customize the amount of memory on the VM:
      vb.memory = "4096"
    end
    #
    # View the documentation for the provider you are using for more
    # information on available options.
  
    # Enable provisioning with a shell script. Additional provisioners such as
    # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
    # documentation for more information about their specific syntax and use.
    config.vm.provision "shell", inline: <<-SHELL
      yum update -y
      # 日本語ロケールを追加しておく。追加しないとエラー。
      localedef -f UTF-8 -i ja_JP ja_JP

      # 検索の単純化のためmlocateをインストール
      yum install -y mlocate

      # webサーバーインストール
      yum install -y nginx

      # SELinux,firewalldの初期状態の確認
      echo SELinux status is ...
      getenforce
      echo firewalld status is ...
      systemctl enable firewalld
      systemctl start firewalld
      firewall-cmd --list-all

      # hostosからguestosの通信で指定のポートを開けておく。
      firewall-cmd --add-port=1521/tcp --zone=public --permanent
      firewall-cmd --add-port=5500/tcp --zone=public --permanent
      firewall-cmd --add-service=http --zone=public --permanent
      firewall-cmd --add-service=https --zone=public --permanent

      # リバースプロキシの設定方法https://gobuffalo.io/en/docs/deploy/proxy
      # nginxのリバースプロキシを使う場合に必要なselinuxの設定
      setsebool -P httpd_can_network_connect on
      setsebool -P httpd_can_network_relay on

      # test用テーブルデータ放り込み用ライブラリ
      yum install -y python3-pip
      pip3 install names

      # python環境
      pip3 install pipenv

      # gradleインストール
      wget https://services.gradle.org/distributions/gradle-6.6.1-bin.zip
      yum install -y unzip
      unzip gradle-6.6.1-bin.zip
      mv gradle-6.6.1 /usr/local/gradle
      rm gradle-6.6.1-bin.zip
      su - vagrant -c 'echo export PATH=/usr/local/gradle/bin:$PATH >> $HOME/.bash_profile'

      # golang環境
      # 開発者様のレポジトリがあるのでそれを使う
      yum-config-manager --enable ol7_developer_golang112
      yum install -y golang
      su - vagrant -c 'echo export GOPATH=$HOME/.go >> $HOME/.bash_profile'

      # rust環境
      # rhel系はyumだと古いRustが入るのでfedoraのepelレポジトリを使う。近いミラーを選ぶこと。
      # レポジトリの公開鍵をダウンロードしてインストール
      wget https://ftp.jaist.ac.jp/pub/Linux/Fedora/epel/RPM-GPG-KEY-EPEL-7Server
      rpm --import RPM-GPG-KEY-EPEL-7Server
      rm RPM-GPG-KEY-EPEL-7Server
      yum-config-manager --add-repo https://ftp.jaist.ac.jp/pub/Linux/Fedora/epel/7Server/x86_64
      yum install -y rust cargo
      # 競合するため、Rustインストール後は無効化
      yum-config-manager --disable ftp.jaist.ac.jp_pub_Linux_Fedora_epel_7Server_x86_64 > /dev/null

      # rlang環境
      yum-config-manager --enable ol7_developer_EPEL
      yum install -y R

      # scala環境
      curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
      yum install -y sbt

      # php環境
      yum install -y oracle-php-release-el7
      yum install -y php
      # yum -y install php php-oci8-19c としてインストールすると、
      # PHP Warning:  PHP Startup: Unable to load dynamic library 'oci8.so' (tried: /usr/lib64/php/modules/oci8.so (libclntsh.so.19.1: cannot open shared object file: No such file or directory), /usr/lib64/php/modules/oci8.so.so (/usr/lib64/php/modules/oci8.so.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
      # PHP Warning:  PHP Startup: Unable to load dynamic library 'pdo_oci.so' (tried: /usr/lib64/php/modules/pdo_oci.so (libclntsh.so.19.1: cannot open shared object file: No such file or directory), /usr/lib64/php/modules/pdo_oci.so.so (/usr/lib64/php/modules/pdo_oci.so.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
      # libclntsh.so.19.1がないからエラーになるが最新のものをインストールしても、libclntsh.so.18.1のため、利用できず、コンパイルが必要。
      # php oci8コンパイルのためのツールをインストール
      yum install -y php-devel php-pear

      # oracle linuxではOracle databaseインストール前に入っているので不要
      # curl -o oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm
      # yum -y localinstall oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm
    SHELL
  end
  