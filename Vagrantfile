Vagrant.configure("2") do |config|

  config.vm.box = "debian/bullseye64"

  # ======================
  # DNS
  # ======================
  config.vm.define "dns" do |dns|
    dns.vm.hostname = "dns.gustavo.local"
    dns.vm.network "private_network", ip: "192.168.57.10"
  
    dns.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "/vagrant/ansible/dns.yml"
      ansible.install = true
    end
  end

  # ======================
  # MAIL (Postfix + Dovecot)
  # ======================
  config.vm.define "postdove" do |mail|
    mail.vm.hostname = "mail.gustavo.local"
    mail.vm.network "private_network", ip: "192.168.57.20"
  end

end

