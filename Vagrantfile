# vi: set ft=ruby :

NUM_DAEMONS = ENV["NUM_DAEMONS"] ? ENV["NUM_DAEMONS"].to_i : 1

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.network "private_network", type: "dhcp"
  config.vm.box = "generic/ubuntu1804" # "centos/7"

  (0..NUM_DAEMONS - 1).each do |i|
    config.vm.define "mon#{i}" do |mon|
      mon.vm.hostname = "mon#{i}"
    end
    config.vm.define "mgr#{i}" do |mgr|
      mgr.vm.hostname = "mgr#{i}"
    end
    config.vm.define "osd#{i}" do |osd|
      osd.vm.hostname = "osd#{i}"
      osd.vm.provider :libvirt do |libvirt|
        libvirt.storage :file, :size => '5G'
        libvirt.storage :file, :size => '5G'
      end
    end
  end

  config.vm.provision "shell" do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = "echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys"
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo sed -i 's,^DNSSEC=.*$,DNSSEC=no,' /etc/systemd/resolved.conf
    sudo systemctl restart systemd-resolved
    curl -L https://shaman.ceph.com/api/repos/ceph/master/latest/ubuntu/$(lsb_release -sc)/repo/ | sudo tee /etc/apt/sources.list.d/shaman.list
    sudo apt update
    sudo apt install -y ceph python-pip
    sudo pip install remoto
  SHELL
end
