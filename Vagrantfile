#
# Vagrant based OpenStack Grizzly       hosts           instances
# eth0 - vagrant/nat                    10.0.2.0/24
# eth1 - instance fixed_ip network      10.16.0.0/22    10.16.1.0/22
# eth2 - instance floating_ip network   172.16.0.0/22   172.16.1.0/22
#

nodes = {
  # run the salt-master first, so that it can be used as an apt-proxy
  # and also try to catch the keys of minions as the come up
  'head0001' => { :third_octet => 0, 'fixed' => 201, 'floating' => 201},
  'head0002' => { :third_octet => 0, 'fixed' => 202, 'floating' => 202},
  'head0003' => { :third_octet => 0, 'fixed' => 203, 'floating' => 203},

  'cpu0001'  => { :third_octet => 0, 'fixed' => 101, 'floating' => 101},
  'cpu0002'  => { :third_octet => 0, 'fixed' => 102, 'floating' => 102},
}

# truncate the current inventory file
File.open('myInventory' ,'w') do |f|
  f.write "[default]\n"
end

Vagrant.configure("2") do |config|
  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
  config.ssh.insert_key = false

  config.vm.provider :virtualbox do |vbox|
    vbox.customize ["modifyvm", :id, "--memory", 1536]

    # allow promiscuous mode
    vbox.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    vbox.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
  end

  nodes.each do |hostname, details|
    config.vm.define "#{hostname}" do |node|
      # node.vm.hostname = "#{hostname}"

      File.open('myInventory' , 'a') do |f|
        #f.write "#{hostname} ansible_ssh_host=10.16.#{details[:third_octet]}.#{details['fixed']} ansible_ssh_user=vagrant ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key\n"
        f.write "#{hostname} ansible_ssh_host=10.16.#{details[:third_octet]}.#{details['fixed']} ansible_ssh_user=vagrant\n"
      end

      # eth1 - fixed/private network
      node.vm.network :private_network, :ip => "172.16.#{details[:third_octet]}.#{details['floating']}", :netmask => "255.255.252.0"
      # eth2 - floating/public network
      node.vm.network :private_network, :ip => "10.16.#{details[:third_octet]}.#{details['fixed']}", :netmask => "255.255.252.0"

    end
  end

  # Ansible provisioning
  config.vm.provision "ansible" do |ansible|
    ansible.inventory_path = "myInventory"
    ansible.playbook = "demo.yml"
    ansible.sudo = true
  end
end
