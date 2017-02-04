Vagrant.configure("2") do |config|
  # Set the base box
  config.vm.box = "centos/7"

  # Create a private network, which allows host-only access to the machine using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # config.vm.synced_folder "../data", "/vagrant_data"

  # Set the memory
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end

  # Inject your public SSH key
  id_rsa_key_pub = File.read(File.join(Dir.home, ".ssh", "id_rsa.pub"))

  config.vm.provision :shell,
        :inline => "echo 'appending SSH public key to ~vagrant/.ssh/authorized_keys' && echo '#{id_rsa_key_pub }' >> /home/vagrant/.ssh/authorized_keys && chmod 600 /home/vagrant/.ssh/authorized_keys"

  config.ssh.insert_key = false

  #config.vm.provision "ansible" do |ansible|
    #ansible.playbook = "site.yml"
  #end
end
