MACHINES = {
  :"backup" => {
              :box_name => "generic/debian12",
              :box_version => "4.3.12",
              :cpus => 1,
              :memory => 1024,
              :disks => {
                :sata1 => {
                  :dfile => './sata1.vdi',
                  :size => 1000,
                  :port => 1
                },
              },
            },
          }

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder ".", "/vagrant", disabled: false
    config.vm.define "backup" do |back|
      back.vm.box = boxconfig[:box_name]
      back.vm.box_version = boxconfig[:box_version]
      back.vm.host_name = "backup"
      back.vm.network "private_network", ip: "192.168.56.10", virtualbox__intnet: "net1"
      back.vm.provider "virtualbox" do |vb|
        vb.memory = boxconfig[:memory]
        vb.cpus = boxconfig[:cpus]
        boxconfig[:disks].each do |dname, dconf|
          unless File.exist?(dconf[:dfile])
            vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
          end
            vb.customize ['storageattach', :id,  '--storagectl', "SATA Controller", '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
        end
      end
      back.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh
            cp ~vagrant/.ssh/auth* ~root/.ssh
            echo -e "n\np\n1\n\n\nw\n" | fdisk /dev/sdb
            mkfs.ext4 /dev/sdb1
            mkdir /var/backup
            mount /dev/sdb1 /var/backup
            rm -rf /var/backup/*
            echo `blkid /dev/sdb1 | awk '{print $2}'` /var/backup ext4 defauls 0 0 >> /etc/fstab
         SHELL

    end
    config.vm.define "client" do |client|
      client.vm.box = boxconfig[:box_name]
      client.vm.box_version = boxconfig[:box_version]
      client.vm.host_name = "client"
      client.vm.network "private_network", ip: "192.168.56.15", virtualbox__intnet: "net1"
      client.vm.provider "virtualbox" do |vb|
        vb.memory = boxconfig[:memory]
        vb.cpus = boxconfig[:cpus]
      end
      client.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh
            cp ~vagrant/.ssh/auth* ~root/.ssh
         SHELL
      client.vm.provision "ansible" do |ansible|
        ansible.playbook = "ansible/playbook.yml"
        #ansible.inventory_path = "ansible/hosts"
        ansible.host_key_checking = "false"
        ansible.limit = "all"
      end
    end
  end
end
