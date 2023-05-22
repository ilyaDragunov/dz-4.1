Домашнее задание 4.1

Для конфигурации сервера (установки и настройки ZFS) необходимо написать отдельный Bash-скрипт и добавить 
его в Vagrantfile. Пример добавления скрипта в Vagrantfile:


К готовому файлу вагранта добавляем несколько строк, которые создадут при разворачивании машины 4 зеркала с zfs, 
так же на разных дисках зараненее установим различный вид сжатия, конфиг приведен ниже 

```

# -*- mode: ruby -*- vim: set ft=ruby :
disk_controller = 'IDE' # MacOS. This setting is OS dependent. Details https://github.com/hashicorp/vagrant/issues/8105


MACHINES = {
  :zfs => {
        :box_name => "centos/7",
        :box_version => "2004.01",
    :disks => {
        :sata1 => {
            :dfile => './sata1.vdi',
            :size => 512,
            :port => 1
        },
        :sata2 => {
            :dfile => './sata2.vdi',
            :size => 512, # Megabytes
            :port => 2
        },
        :sata3 => {
            :dfile => './sata3.vdi',
            :size => 512,
            :port => 3
        },
        :sata4 => {
            :dfile => './sata4.vdi',
            :size => 512, 
            :port => 4
        },
        :sata5 => {
            :dfile => './sata5.vdi',
            :size => 512,
            :port => 5
        },
        :sata6 => {
            :dfile => './sata6.vdi',
            :size => 512,
            :port => 6
        },
        :sata7 => {
            :dfile => './sata7.vdi',
            :size => 512, 
            :port => 7
        },
        :sata8 => {
            :dfile => './sata8.vdi',
            :size => 512, 
            :port => 8
        },
    }
        
  },
}


Vagrant.configure("2") do |config|


  MACHINES.each do |boxname, boxconfig|

	config.vm.synced_folder '.', '/vagrant', disabled: true
      config.vm.define boxname do |box|


        box.vm.box = boxconfig[:box_name]
        box.vm.box_version = boxconfig[:box_version]


        box.vm.host_name = "zfs"


        box.vm.provider :virtualbox do |vb|
              vb.customize ["modifyvm", :id, "--memory", "1024"]
              needsController = false
        boxconfig[:disks].each do |dname, dconf|
              unless File.exist?(dconf[:dfile])
              vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
         needsController =  true
         end
        end
            if needsController == true
                vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                boxconfig[:disks].each do |dname, dconf|
                vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                end
             end
          end
        box.vm.provision "shell", inline: <<-SHELL
          #install zfs repo
          yum install -y http://download.zfsonlinux.org/epel/zfs-release.el7_8.noarch.rpm
          #import gpg key 
          rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux
          #install DKMS style packages for correct work ZFS
          yum install -y epel-release kernel-devel zfs
          #change ZFS repo
          yum-config-manager --disable zfs
          yum-config-manager --enable zfs-kmod
          yum install -y zfs
          #Add kernel module zfs
          modprobe zfs
          #install wget
          yum install -y wget
	zpool create otus1 mirror /dev/sdb /dev/sdc
	zpool create otus2 mirror /dev/sdd /dev/sde
	zpool create otus3 mirror /dev/sdf /dev/sdg
	zpool create otus4 mirror /dev/sdh /dev/sdi
	zfs set compression=lzjb otus1
	zfs set compression=lz4 otus2
	zfs set compression=gzip-9 otus3
	zfs set compression=zle otus4
      SHELL


    end
  end
end

```


Листинг запуска вагрант машины приведен ниже


```

user@ubuntu-vm:~/DZ-OTUS/DZ-4.1$ vagrant up
Bringing machine 'zfs' up with 'virtualbox' provider...
==> zfs: Importing base box 'centos/7'...
==> zfs: Matching MAC address for NAT networking...
==> zfs: Checking if box 'centos/7' version '2004.01' is up to date...
==> zfs: Setting the name of the VM: DZ-41_zfs_1684790682162_69376
==> zfs: Clearing any previously set network interfaces...
==> zfs: Preparing network interfaces based on configuration...
    zfs: Adapter 1: nat
==> zfs: Forwarding ports...
    zfs: 22 (guest) => 2222 (host) (adapter 1)
==> zfs: Running 'pre-boot' VM customizations...
==> zfs: Booting VM...
==> zfs: Waiting for machine to boot. This may take a few minutes...
    zfs: SSH address: 127.0.0.1:2222
    zfs: SSH username: vagrant
    zfs: SSH auth method: private key
    zfs:
    zfs: Vagrant insecure key detected. Vagrant will automatically replace
    zfs: this with a newly generated keypair for better security.
    zfs:
    zfs: Inserting generated public key within guest...
    zfs: Removing insecure key from the guest if it's present...
    zfs: Key inserted! Disconnecting and reconnecting using new SSH key...
==> zfs: Machine booted and ready!
==> zfs: Checking for guest additions in VM...
    zfs: No guest additions were detected on the base box for this VM! Guest
    zfs: additions are required for forwarded ports, shared folders, host only
    zfs: networking, and more. If SSH fails on this machine, please install
    zfs: the guest additions and repackage the box to continue.
    zfs:
    zfs: This is not an error message; everything may continue to work properly,
    zfs: in which case you may ignore this message.
==> zfs: Setting hostname...
==> zfs: Running provisioner: shell...
    zfs: Running: inline script
    zfs: Loaded plugins: fastestmirror
    zfs: Examining /var/tmp/yum-root-MfBUQS/zfs-release.el7_8.noarch.rpm: zfs-release-1-7.8.noarch
    zfs: Marking /var/tmp/yum-root-MfBUQS/zfs-release.el7_8.noarch.rpm to be installed
    zfs: Resolving Dependencies
    zfs: --> Running transaction check
    zfs: ---> Package zfs-release.noarch 0:1-7.8 will be installed
    zfs: --> Finished Dependency Resolution
    zfs:
    zfs: Dependencies Resolved
    zfs:
    zfs: ================================================================================
    zfs:  Package          Arch        Version      Repository                      Size
    zfs: ================================================================================
    zfs: Installing:
    zfs:  zfs-release      noarch      1-7.8        /zfs-release.el7_8.noarch      2.9 k
    zfs:
    zfs: Transaction Summary
    zfs: ================================================================================
    zfs: Install  1 Package
    zfs:
    zfs: Total size: 2.9 k
    zfs: Installed size: 2.9 k
    zfs: Downloading packages:
    zfs: Running transaction check
    zfs: Running transaction test
    zfs: Transaction test succeeded
    zfs: Running transaction
    zfs:   Installing : zfs-release-1-7.8.noarch                                     1/1
    zfs:   Verifying  : zfs-release-1-7.8.noarch                                     1/1
    zfs:
    zfs: Installed:
    zfs:   zfs-release.noarch 0:1-7.8
    zfs:
    zfs: Complete!
    zfs: Loaded plugins: fastestmirror
    zfs: Determining fastest mirrors
    zfs:  * base: mirror.23m.com
    zfs:  * extras: centos.schlundtech.de
    zfs:  * updates: mirror.23m.com
    zfs: Resolving Dependencies
    zfs: --> Running transaction check
    zfs: ---> Package epel-release.noarch 0:7-11 will be installed
    zfs: ---> Package kernel-devel.x86_64 0:3.10.0-1160.90.1.el7 will be installed
    zfs: --> Processing Dependency: perl for package: kernel-devel-3.10.0-1160.90.1.el7.x86_64
    zfs: ---> Package zfs.x86_64 0:0.8.5-1.el7 will be installed
    zfs: --> Processing Dependency: zfs-kmod = 0.8.5 for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libzpool2 = 0.8.5 for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libzfs2 = 0.8.5 for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libuutil1 = 0.8.5 for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libnvpair1 = 0.8.5 for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: sysstat for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libzpool.so.2()(64bit) for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libzfs_core.so.1()(64bit) for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libzfs.so.2()(64bit) for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libuutil.so.1()(64bit) for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libnvpair.so.1()(64bit) for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Running transaction check
    zfs: ---> Package libnvpair1.x86_64 0:0.8.5-1.el7 will be installed
    zfs: ---> Package libuutil1.x86_64 0:0.8.5-1.el7 will be installed
    zfs: ---> Package libzfs2.x86_64 0:0.8.5-1.el7 will be installed
    zfs: ---> Package libzpool2.x86_64 0:0.8.5-1.el7 will be installed
    zfs: ---> Package perl.x86_64 4:5.16.3-299.el7_9 will be installed
    zfs: --> Processing Dependency: perl-libs = 4:5.16.3-299.el7_9 for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Socket) >= 1.3 for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Scalar::Util) >= 1.10 for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl-macros for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl-libs for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(threads::shared) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(threads) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(constant) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Time::Local) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Time::HiRes) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Storable) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Socket) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Scalar::Util) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Pod::Simple::XHTML) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Pod::Simple::Search) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Getopt::Long) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Filter::Util::Call) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(File::Temp) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(File::Spec::Unix) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(File::Spec::Functions) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(File::Spec) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(File::Path) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Exporter) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Cwd) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: perl(Carp) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: --> Processing Dependency: libperl.so()(64bit) for package: 4:perl-5.16.3-299.el7_9.x86_64
    zfs: ---> Package sysstat.x86_64 0:10.1.5-20.el7_9 will be installed
    zfs: --> Processing Dependency: libsensors.so.4()(64bit) for package: sysstat-10.1.5-20.el7_9.x86_64
    zfs: ---> Package zfs-dkms.noarch 0:0.8.5-1.el7 will be installed
    zfs: --> Processing Dependency: dkms >= 2.2.0.3 for package: zfs-dkms-0.8.5-1.el7.noarch
    zfs: --> Processing Dependency: gcc for package: zfs-dkms-0.8.5-1.el7.noarch
    zfs: --> Running transaction check
    zfs: ---> Package gcc.x86_64 0:4.8.5-44.el7 will be installed
    zfs: --> Processing Dependency: libgomp = 4.8.5-44.el7 for package: gcc-4.8.5-44.el7.x86_64
    zfs: --> Processing Dependency: cpp = 4.8.5-44.el7 for package: gcc-4.8.5-44.el7.x86_64
    zfs: --> Processing Dependency: libgcc >= 4.8.5-44.el7 for package: gcc-4.8.5-44.el7.x86_64
    zfs: --> Processing Dependency: glibc-devel >= 2.2.90-12 for package: gcc-4.8.5-44.el7.x86_64
    zfs: --> Processing Dependency: libmpfr.so.4()(64bit) for package: gcc-4.8.5-44.el7.x86_64
    zfs: --> Processing Dependency: libmpc.so.3()(64bit) for package: gcc-4.8.5-44.el7.x86_64
    zfs: ---> Package lm_sensors-libs.x86_64 0:3.4.0-8.20160601gitf9185e5.el7 will be installed
    zfs: ---> Package perl-Carp.noarch 0:1.26-244.el7 will be installed
    zfs: ---> Package perl-Exporter.noarch 0:5.68-3.el7 will be installed
    zfs: ---> Package perl-File-Path.noarch 0:2.09-2.el7 will be installed
    zfs: ---> Package perl-File-Temp.noarch 0:0.23.01-3.el7 will be installed
    zfs: ---> Package perl-Filter.x86_64 0:1.49-3.el7 will be installed
    zfs: ---> Package perl-Getopt-Long.noarch 0:2.40-3.el7 will be installed
    zfs: --> Processing Dependency: perl(Pod::Usage) >= 1.14 for package: perl-Getopt-Long-2.40-3.el7.noarch
    zfs: --> Processing Dependency: perl(Text::ParseWords) for package: perl-Getopt-Long-2.40-3.el7.noarch
    zfs: ---> Package perl-PathTools.x86_64 0:3.40-5.el7 will be installed
    zfs: ---> Package perl-Pod-Simple.noarch 1:3.28-4.el7 will be installed
    zfs: --> Processing Dependency: perl(Pod::Escapes) >= 1.04 for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
    zfs: --> Processing Dependency: perl(Encode) for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
    zfs: ---> Package perl-Scalar-List-Utils.x86_64 0:1.27-248.el7 will be installed
    zfs: ---> Package perl-Socket.x86_64 0:2.010-5.el7 will be installed
    zfs: ---> Package perl-Storable.x86_64 0:2.45-3.el7 will be installed
    zfs: ---> Package perl-Time-HiRes.x86_64 4:1.9725-3.el7 will be installed
    zfs: ---> Package perl-Time-Local.noarch 0:1.2300-2.el7 will be installed
    zfs: ---> Package perl-constant.noarch 0:1.27-2.el7 will be installed
    zfs: ---> Package perl-libs.x86_64 4:5.16.3-299.el7_9 will be installed
    zfs: ---> Package perl-macros.x86_64 4:5.16.3-299.el7_9 will be installed
    zfs: ---> Package perl-threads.x86_64 0:1.87-4.el7 will be installed
    zfs: ---> Package perl-threads-shared.x86_64 0:1.43-6.el7 will be installed
    zfs: ---> Package zfs-dkms.noarch 0:0.8.5-1.el7 will be installed
    zfs: --> Processing Dependency: dkms >= 2.2.0.3 for package: zfs-dkms-0.8.5-1.el7.noarch
    zfs: --> Running transaction check
    zfs: ---> Package cpp.x86_64 0:4.8.5-44.el7 will be installed
    zfs: ---> Package glibc-devel.x86_64 0:2.17-326.el7_9 will be installed
    zfs: --> Processing Dependency: glibc-headers = 2.17-326.el7_9 for package: glibc-devel-2.17-326.el7_9.x86_64
    zfs: --> Processing Dependency: glibc = 2.17-326.el7_9 for package: glibc-devel-2.17-326.el7_9.x86_64
    zfs: --> Processing Dependency: glibc-headers for package: glibc-devel-2.17-326.el7_9.x86_64
    zfs: ---> Package libgcc.x86_64 0:4.8.5-39.el7 will be updated
    zfs: ---> Package libgcc.x86_64 0:4.8.5-44.el7 will be an update
    zfs: ---> Package libgomp.x86_64 0:4.8.5-39.el7 will be updated
    zfs: ---> Package libgomp.x86_64 0:4.8.5-44.el7 will be an update
    zfs: ---> Package libmpc.x86_64 0:1.0.1-3.el7 will be installed
    zfs: ---> Package mpfr.x86_64 0:3.1.1-4.el7 will be installed
    zfs: ---> Package perl-Encode.x86_64 0:2.51-7.el7 will be installed
    zfs: ---> Package perl-Pod-Escapes.noarch 1:1.04-299.el7_9 will be installed
    zfs: ---> Package perl-Pod-Usage.noarch 0:1.63-3.el7 will be installed
    zfs: --> Processing Dependency: perl(Pod::Text) >= 3.15 for package: perl-Pod-Usage-1.63-3.el7.noarch
    zfs: --> Processing Dependency: perl-Pod-Perldoc for package: perl-Pod-Usage-1.63-3.el7.noarch
    zfs: ---> Package perl-Text-ParseWords.noarch 0:3.29-4.el7 will be installed
    zfs: ---> Package zfs-dkms.noarch 0:0.8.5-1.el7 will be installed
    zfs: --> Processing Dependency: dkms >= 2.2.0.3 for package: zfs-dkms-0.8.5-1.el7.noarch
    zfs: --> Running transaction check
    zfs: ---> Package glibc.x86_64 0:2.17-307.el7.1 will be updated
    zfs: --> Processing Dependency: glibc = 2.17-307.el7.1 for package: glibc-common-2.17-307.el7.1.x86_64
    zfs: ---> Package glibc.x86_64 0:2.17-326.el7_9 will be an update
    zfs: ---> Package glibc-headers.x86_64 0:2.17-326.el7_9 will be installed
    zfs: --> Processing Dependency: kernel-headers >= 2.2.1 for package: glibc-headers-2.17-326.el7_9.x86_64
    zfs: --> Processing Dependency: kernel-headers for package: glibc-headers-2.17-326.el7_9.x86_64
    zfs: ---> Package perl-Pod-Perldoc.noarch 0:3.20-4.el7 will be installed
    zfs: --> Processing Dependency: perl(parent) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
    zfs: --> Processing Dependency: perl(HTTP::Tiny) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
    zfs: ---> Package perl-podlators.noarch 0:2.5.1-3.el7 will be installed
    zfs: ---> Package zfs-dkms.noarch 0:0.8.5-1.el7 will be installed
    zfs: --> Processing Dependency: dkms >= 2.2.0.3 for package: zfs-dkms-0.8.5-1.el7.noarch
    zfs: --> Running transaction check
    zfs: ---> Package glibc-common.x86_64 0:2.17-307.el7.1 will be updated
    zfs: ---> Package glibc-common.x86_64 0:2.17-326.el7_9 will be an update
    zfs: ---> Package kernel-headers.x86_64 0:3.10.0-1160.90.1.el7 will be installed
    zfs: ---> Package perl-HTTP-Tiny.noarch 0:0.033-3.el7 will be installed
    zfs: ---> Package perl-parent.noarch 1:0.225-244.el7 will be installed
    zfs: ---> Package zfs-dkms.noarch 0:0.8.5-1.el7 will be installed
    zfs: --> Processing Dependency: dkms >= 2.2.0.3 for package: zfs-dkms-0.8.5-1.el7.noarch
    zfs: --> Finished Dependency Resolution
    zfs: Error: Package: zfs-dkms-0.8.5-1.el7.noarch (zfs)
    zfs:            Requires: dkms >= 2.2.0.3
    zfs:  You could try using --skip-broken to work around the problem
    zfs:  You could try running: rpm -Va --nofiles --nodigest
    zfs: Loaded plugins: fastestmirror
    zfs: ================================== repo: zfs ===================================
    zfs: [zfs]
    zfs: async = True
    zfs: bandwidth = 0
    zfs: base_persistdir = /var/lib/yum/repos/x86_64/7
    zfs: baseurl = http://download.zfsonlinux.org/epel/7.8/x86_64/
    zfs: cache = 0
    zfs: cachedir = /var/cache/yum/x86_64/7/zfs
    zfs: check_config_file_age = True
    zfs: compare_providers_priority = 80
    zfs: cost = 1000
    zfs: deltarpm_metadata_percentage = 100
    zfs: deltarpm_percentage =
    zfs: enabled = 0
    zfs: enablegroups = True
    zfs: exclude =
    zfs: failovermethod = priority
    zfs: ftp_disable_epsv = False
    zfs: gpgcadir = /var/lib/yum/repos/x86_64/7/zfs/gpgcadir
    zfs: gpgcakey =
    zfs: gpgcheck = True
    zfs: gpgdir = /var/lib/yum/repos/x86_64/7/zfs/gpgdir
    zfs: gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux
    zfs: hdrdir = /var/cache/yum/x86_64/7/zfs/headers
    zfs: http_caching = all
    zfs: includepkgs =
    zfs: ip_resolve =
    zfs: keepalive = True
    zfs: keepcache = False
    zfs: mddownloadpolicy = sqlite
    zfs: mdpolicy = group:small
    zfs: mediaid =
    zfs: metadata_expire = 604800
    zfs: metadata_expire_filter = read-only:present
    zfs: metalink =
    zfs: minrate = 0
    zfs: mirrorlist =
    zfs: mirrorlist_expire = 86400
    zfs: name = ZFS on Linux for EL7 - dkms
    zfs: old_base_cache_dir =
    zfs: password =
    zfs: persistdir = /var/lib/yum/repos/x86_64/7/zfs
    zfs: pkgdir = /var/cache/yum/x86_64/7/zfs/packages
    zfs: proxy = False
    zfs: proxy_dict =
    zfs: proxy_password =
    zfs: proxy_username =
    zfs: repo_gpgcheck = False
    zfs: retries = 10
    zfs: skip_if_unavailable = False
    zfs: ssl_check_cert_permissions = True
    zfs: sslcacert =
    zfs: sslclientcert =
    zfs: sslclientkey =
    zfs: sslverify = True
    zfs: throttle = 0
    zfs: timeout = 30.0
    zfs: ui_id = zfs/x86_64
    zfs: ui_repoid_vars = releasever,
    zfs:    basearch
    zfs: username =
    zfs:
    zfs: Loaded plugins: fastestmirror
    zfs: ================================ repo: zfs-kmod ================================
    zfs: [zfs-kmod]
    zfs: async = True
    zfs: bandwidth = 0
    zfs: base_persistdir = /var/lib/yum/repos/x86_64/7
    zfs: baseurl = http://download.zfsonlinux.org/epel/7.8/kmod/x86_64/
    zfs: cache = 0
    zfs: cachedir = /var/cache/yum/x86_64/7/zfs-kmod
    zfs: check_config_file_age = True
    zfs: compare_providers_priority = 80
    zfs: cost = 1000
    zfs: deltarpm_metadata_percentage = 100
    zfs: deltarpm_percentage =
    zfs: enabled = 1
    zfs: enablegroups = True
    zfs: exclude =
    zfs: failovermethod = priority
    zfs: ftp_disable_epsv = False
    zfs: gpgcadir = /var/lib/yum/repos/x86_64/7/zfs-kmod/gpgcadir
    zfs: gpgcakey =
    zfs: gpgcheck = True
    zfs: gpgdir = /var/lib/yum/repos/x86_64/7/zfs-kmod/gpgdir
    zfs: gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux
    zfs: hdrdir = /var/cache/yum/x86_64/7/zfs-kmod/headers
    zfs: http_caching = all
    zfs: includepkgs =
    zfs: ip_resolve =
    zfs: keepalive = True
    zfs: keepcache = False
    zfs: mddownloadpolicy = sqlite
    zfs: mdpolicy = group:small
    zfs: mediaid =
    zfs: metadata_expire = 604800
    zfs: metadata_expire_filter = read-only:present
    zfs: metalink =
    zfs: minrate = 0
    zfs: mirrorlist =
    zfs: mirrorlist_expire = 86400
    zfs: name = ZFS on Linux for EL7 - kmod
    zfs: old_base_cache_dir =
    zfs: password =
    zfs: persistdir = /var/lib/yum/repos/x86_64/7/zfs-kmod
    zfs: pkgdir = /var/cache/yum/x86_64/7/zfs-kmod/packages
    zfs: proxy = False
    zfs: proxy_dict =
    zfs: proxy_password =
    zfs: proxy_username =
    zfs: repo_gpgcheck = False
    zfs: retries = 10
    zfs: skip_if_unavailable = False
    zfs: ssl_check_cert_permissions = True
    zfs: sslcacert =
    zfs: sslclientcert =
    zfs: sslclientkey =
    zfs: sslverify = True
    zfs: throttle = 0
    zfs: timeout = 30.0
    zfs: ui_id = zfs-kmod/x86_64
    zfs: ui_repoid_vars = releasever,
    zfs:    basearch
    zfs: username =
    zfs:
    zfs: Loaded plugins: fastestmirror
    zfs: Loading mirror speeds from cached hostfile
    zfs:  * base: mirror.23m.com
    zfs:  * extras: centos.schlundtech.de
    zfs:  * updates: mirror.23m.com
    zfs: Resolving Dependencies
    zfs: --> Running transaction check
    zfs: ---> Package zfs.x86_64 0:0.8.5-1.el7 will be installed
    zfs: --> Processing Dependency: zfs-kmod = 0.8.5 for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libzpool2 = 0.8.5 for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libzfs2 = 0.8.5 for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libuutil1 = 0.8.5 for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libnvpair1 = 0.8.5 for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: sysstat for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libzpool.so.2()(64bit) for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libzfs_core.so.1()(64bit) for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libzfs.so.2()(64bit) for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libuutil.so.1()(64bit) for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Processing Dependency: libnvpair.so.1()(64bit) for package: zfs-0.8.5-1.el7.x86_64
    zfs: --> Running transaction check
    zfs: ---> Package kmod-zfs.x86_64 0:0.8.5-1.el7 will be installed
    zfs: ---> Package libnvpair1.x86_64 0:0.8.5-1.el7 will be installed
    zfs: ---> Package libuutil1.x86_64 0:0.8.5-1.el7 will be installed
    zfs: ---> Package libzfs2.x86_64 0:0.8.5-1.el7 will be installed
    zfs: ---> Package libzpool2.x86_64 0:0.8.5-1.el7 will be installed
    zfs: ---> Package sysstat.x86_64 0:10.1.5-20.el7_9 will be installed
    zfs: --> Processing Dependency: libsensors.so.4()(64bit) for package: sysstat-10.1.5-20.el7_9.x86_64
    zfs: --> Running transaction check
    zfs: ---> Package lm_sensors-libs.x86_64 0:3.4.0-8.20160601gitf9185e5.el7 will be installed
    zfs: --> Finished Dependency Resolution
    zfs:
    zfs: Dependencies Resolved
    zfs:
    zfs: ================================================================================
    zfs:  Package           Arch     Version                            Repository  Size
    zfs: ================================================================================
    zfs: Installing:
    zfs:  zfs               x86_64   0.8.5-1.el7                        zfs-kmod   486 k
    zfs: Installing for dependencies:
    zfs:  kmod-zfs          x86_64   0.8.5-1.el7                        zfs-kmod   1.1 M
    zfs:  libnvpair1        x86_64   0.8.5-1.el7                        zfs-kmod    32 k
    zfs:  libuutil1         x86_64   0.8.5-1.el7                        zfs-kmod    26 k
    zfs:  libzfs2           x86_64   0.8.5-1.el7                        zfs-kmod   208 k
    zfs:  libzpool2         x86_64   0.8.5-1.el7                        zfs-kmod   861 k
    zfs:  lm_sensors-libs   x86_64   3.4.0-8.20160601gitf9185e5.el7     base        42 k
    zfs:  sysstat           x86_64   10.1.5-20.el7_9                    updates    315 k
    zfs:
    zfs: Transaction Summary
    zfs: ================================================================================
    zfs: Install  1 Package (+7 Dependent packages)
    zfs:
    zfs: Total download size: 3.0 M
    zfs: Installed size: 11 M
    zfs: Downloading packages:
    zfs: Public key for lm_sensors-libs-3.4.0-8.20160601gitf9185e5.el7.x86_64.rpm is not installed
    zfs: warning: /var/cache/yum/x86_64/7/base/packages/lm_sensors-libs-3.4.0-8.20160601gitf9185e5.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
    zfs: Public key for sysstat-10.1.5-20.el7_9.x86_64.rpm is not installed
    zfs: --------------------------------------------------------------------------------
    zfs: Total                                              1.0 MB/s | 3.0 MB  00:02
    zfs: Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    zfs: Importing GPG key 0xF4A80EB5:
    zfs:  Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
    zfs:  Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
    zfs:  Package    : centos-release-7-8.2003.0.el7.centos.x86_64 (@anaconda)
    zfs:  From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    zfs: Running transaction check
    zfs: Running transaction test
    zfs: Transaction test succeeded
    zfs: Running transaction
    zfs:   Installing : libnvpair1-0.8.5-1.el7.x86_64                                1/8
    zfs:   Installing : libuutil1-0.8.5-1.el7.x86_64                                 2/8
    zfs:   Installing : libzfs2-0.8.5-1.el7.x86_64                                   3/8
    zfs:   Installing : libzpool2-0.8.5-1.el7.x86_64                                 4/8
    zfs:   Installing : lm_sensors-libs-3.4.0-8.20160601gitf9185e5.el7.x86_64        5/8
    zfs:   Installing : sysstat-10.1.5-20.el7_9.x86_64                               6/8
    zfs:   Installing : kmod-zfs-0.8.5-1.el7.x86_64                                  7/8
    zfs:   Installing : zfs-0.8.5-1.el7.x86_64                                       8/8
    zfs:   Verifying  : libuutil1-0.8.5-1.el7.x86_64                                 1/8
    zfs:   Verifying  : libzfs2-0.8.5-1.el7.x86_64                                   2/8
    zfs:   Verifying  : zfs-0.8.5-1.el7.x86_64                                       3/8
    zfs:   Verifying  : sysstat-10.1.5-20.el7_9.x86_64                               4/8
    zfs:   Verifying  : kmod-zfs-0.8.5-1.el7.x86_64                                  5/8
    zfs:   Verifying  : libnvpair1-0.8.5-1.el7.x86_64                                6/8
    zfs:   Verifying  : lm_sensors-libs-3.4.0-8.20160601gitf9185e5.el7.x86_64        7/8
    zfs:   Verifying  : libzpool2-0.8.5-1.el7.x86_64                                 8/8
    zfs:
    zfs: Installed:
    zfs:   zfs.x86_64 0:0.8.5-1.el7
    zfs:
    zfs: Dependency Installed:
    zfs:   kmod-zfs.x86_64 0:0.8.5-1.el7
    zfs:   libnvpair1.x86_64 0:0.8.5-1.el7
    zfs:   libuutil1.x86_64 0:0.8.5-1.el7
    zfs:   libzfs2.x86_64 0:0.8.5-1.el7
    zfs:   libzpool2.x86_64 0:0.8.5-1.el7
    zfs:   lm_sensors-libs.x86_64 0:3.4.0-8.20160601gitf9185e5.el7
    zfs:   sysstat.x86_64 0:10.1.5-20.el7_9
    zfs:
    zfs: Complete!
    zfs: Loaded plugins: fastestmirror
    zfs: Loading mirror speeds from cached hostfile
    zfs:  * base: mirror.23m.com
    zfs:  * extras: centos.schlundtech.de
    zfs:  * updates: mirror.23m.com
    zfs: Resolving Dependencies
    zfs: --> Running transaction check
    zfs: ---> Package wget.x86_64 0:1.14-18.el7_6.1 will be installed
    zfs: --> Finished Dependency Resolution
    zfs:
    zfs: Dependencies Resolved
    zfs:
    zfs: ================================================================================
    zfs:  Package        Arch             Version                   Repository      Size
    zfs: ================================================================================
    zfs: Installing:
    zfs:  wget           x86_64           1.14-18.el7_6.1           base           547 k
    zfs:
    zfs: Transaction Summary
    zfs: ================================================================================
    zfs: Install  1 Package
    zfs:
    zfs: Total download size: 547 k
    zfs: Installed size: 2.0 M
    zfs: Downloading packages:
    zfs: Running transaction check
    zfs: Running transaction test
    zfs: Transaction test succeeded
    zfs: Running transaction
    zfs:   Installing : wget-1.14-18.el7_6.1.x86_64                                  1/1
    zfs:   Verifying  : wget-1.14-18.el7_6.1.x86_64                                  1/1
    zfs:
    zfs: Installed:
    zfs:   wget.x86_64 0:1.14-18.el7_6.1
    zfs:
    zfs: Complete!
	
```

Далее проверим что внесенные изменения работают 

```
	
user@ubuntu-vm:~/DZ-OTUS/DZ-4.1$
user@ubuntu-vm:~/DZ-OTUS/DZ-4.1$ vagrant ssh

[vagrant@zfs ~]$ zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M   118K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus2   480M   118K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus3   480M   118K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus4   480M   118K   480M        -         -     0%     0%  1.00x    ONLINE  -

[vagrant@zfs ~]$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk
`-sda1   8:1    0   40G  0 part /
sdb      8:16   0  512M  0 disk
|-sdb1   8:17   0  502M  0 part
`-sdb9   8:25   0    8M  0 part
sdc      8:32   0  512M  0 disk
|-sdc1   8:33   0  502M  0 part
`-sdc9   8:41   0    8M  0 part
sdd      8:48   0  512M  0 disk
|-sdd1   8:49   0  502M  0 part
`-sdd9   8:57   0    8M  0 part
sde      8:64   0  512M  0 disk
|-sde1   8:65   0  502M  0 part
`-sde9   8:73   0    8M  0 part
sdf      8:80   0  512M  0 disk
|-sdf1   8:81   0  502M  0 part
`-sdf9   8:89   0    8M  0 part
sdg      8:96   0  512M  0 disk
|-sdg1   8:97   0  502M  0 part
`-sdg9   8:105  0    8M  0 part
sdh      8:112  0  512M  0 disk
|-sdh1   8:113  0  502M  0 part
`-sdh9   8:121  0    8M  0 part
sdi      8:128  0  512M  0 disk
|-sdi1   8:129  0  502M  0 part
`-sdi9   8:137  0    8M  0 part

[vagrant@zfs ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        489M     0  489M   0% /dev
tmpfs           496M     0  496M   0% /dev/shm
tmpfs           496M  6.8M  489M   2% /run
tmpfs           496M     0  496M   0% /sys/fs/cgroup
/dev/sda1        40G  3.2G   37G   8% /
otus1           352M  128K  352M   1% /otus1
otus2           352M  128K  352M   1% /otus2
otus3           352M  128K  352M   1% /otus3
otus4           352M  128K  352M   1% /otus4
tmpfs           100M     0  100M   0% /run/user/1000

[vagrant@zfs ~]$ zfs get all | grep compression
otus1  compression           lzjb                   local
otus2  compression           lz4                    local
otus3  compression           gzip-9                 local
otus4  compression           zle                    local

```# dz-4.1
