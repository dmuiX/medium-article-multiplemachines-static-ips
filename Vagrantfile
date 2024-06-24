Vagrant.configure("2") do |config|
  config.vm.box = "fedora/40-cloud-base" # no static ip with bridge!!
  config.vm.box_version = "40.20240414.0"
  
  config.vm.provision "shell", inline: "echo 'ssh-ed25519 ' >> /home/vagrant/.ssh/authorized_keys"
    
  nodes = [
    { :name => "haproxy", :ip => "192.168.1.120"},
    { :name => "control-node-1", :ip => "192.168.1.111"},
    { :name => "control-node-2", :ip => "192.168.1.112"},
    { :name => "control-node-3", :ip => "192.168.1.113"},
    { :name => "worker-node-1", :ip => "192.168.1.114"},
    { :name => "worker-node-2", :ip => "192.168.1.115"}
  ]

  nodes.each do |node|
    if node[:name].include? "haproxy"
      config.vm.provider :libvirt do |libvirt|
        libvirt.driver = "kvm"
        libvirt.cpus = 2
        libvirt.memory = 1024
        libvirt.nested = true
      end
    end
    config.vm.provider :libvirt do |libvirt|
      libvirt.driver = "kvm"
      libvirt.cpus = 4
      libvirt.memory = 2048
      libvirt.nested = true
    end

    config.vm.define node[:name] do |config|
      config.vm.hostname = node[:name]
      config.vm.network :public_network,
        :dev => "br0",
        :type => "bridge"
    end

    config.vm.provision "shell" do |s|
      s.env = {"IP" => node[:ip]}
      s.inline = <<-SHELL
        nmcli con modify 'Wired connection 2' ifname eth1 ipv4.method manual ipv4.addresses $IP/24 gw4 192.168.1.1
      SHELL
      s.reboot = true
    end

    config.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "test.yml"
    end
  end
end
