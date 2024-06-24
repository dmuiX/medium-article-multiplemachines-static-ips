nodes = [
  { :name => "haproxy", :ip => "192.168.1.20"},
  { :name => "control-node-1", :ip => "192.168.1.21"}
  { :name => "control-node-2", :ip => "192.168.1.22"},
  { :name => "control-node-3", :ip => "192.168.1.23"},
  { :name => "worker-node-1", :ip => "192.168.1.24"},
  { :name => "worker-node-2", :ip => "192.168.1.25"}
]

Vagrant.configure("2") do |config|
  nodes.each do |node|
    config.vm.define node[:name] do |nodeConfig|
      nodeConfig.vm.box = "fedora/40-cloud-base" # no static ip with bridge!!
      nodeConfig.vm.box_version = "40.20240414.0"
      nodeConfig.vm.hostname = node[:name]
      nodeConfig.vm.network :public_network, :dev => "br0", :type => "bridge"
      if node[:name] == "haproxy"
        nodeConfig.vm.provider :libvirt do |libvirt|
          libvirt.driver = "kvm"
          libvirt.cpus = 2
          libvirt.memory = 1024
          libvirt.nested = true
        end
      else
        nodeConfig.vm.provider :libvirt do |libvirt|
          libvirt.driver = "kvm"
          libvirt.cpus = 4
          libvirt.memory = 2048
          libvirt.nested = true
        end
      end
      nodeConfig.vm.provision "shell", inline: "echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICW36PfNNkMOmj0Ph9W92Ni7DRTmWn0cjxk7mM/GEr84' >> /home/vagrant/.ssh/authorized_keys"
      nodeConfig.vm.provision "shell", inline: "echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEjcB8a6z8NkRsFx8rQrLhelNIWylswX9f/xY+1nw81x' >> /home/vagrant/.ssh/authorized_keys"
      nodeConfig.vm.provision "shell" do |s|
        s.env = {"IP" => node[:ip]}
        s.inline = <<-SHELL
          echo $IP
          nmcli con modify 'Wired connection 2' ifname eth1 ipv4.method manual ipv4.addresses $IP/24 gw4 192.168.1.1
          nmcli con up 'Wired connection 2'
        SHELL
      end
      nodeConfig.vm.provision "ansible" do |ansible|
        ansible.compatibility_mode = "2.0"
        ansible.playbook = "ansible-playbooks/test.yml"
      end
    end
  end
end
