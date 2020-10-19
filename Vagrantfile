IMAGE_NAME = "bento/ubuntu-20.04"

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 4096
        v.cpus = 2
    end

    config.vm.define "maas" do |maas|    
        maas.vm.box = IMAGE_NAME
        maas.vm.network "private_network", ip: "172.16.1.10"
        maas.vm.hostname = "maas"
        maas.vm.network "forwarded_port", guest: 5240, host: 5240
	maas.vm.provision "ansible" do |ansible|
            ansible.playbook = "maas/playbook_maas.yml"
            ansible.extra_vars = {
                node_ip: "172.16.1.10",
            }
        end
    end

    config.vm.define "juju" do |juju|    
        juju.vm.box = IMAGE_NAME
        juju.vm.network "private_network", ip: "172.16.1.20"
        juju.vm.hostname = "juju"
	juju.vm.provision "ansible" do |ansible|
            ansible.playbook = "juju_ansible/playbook_juju.yml"
            ansible.extra_vars = {
                node_ip: "172.16.1.20",
            }
        end
    end
end
