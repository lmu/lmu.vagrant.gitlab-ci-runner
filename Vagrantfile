# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  # Use SSH-Key of host machine
  config.ssh.forward_agent = true

  # Set Base Box and URL
  config.vm.box = "debian/buster64"

  # Disable SharedFolder by default.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # General Provider Settings
  config.vm.provider "virtualbox" do |vb|
    # Disable GUI --> No GUI Window will pop up on start, could be opened on virtual Box app via double click on machine.
    vb.gui = false

    # special VirtualBox Customizations:
    vb.customize ["modifyvm", :id,
                  "--nicpromisc1", "allow-all",
                  "--nicpromisc2", "allow-all",
                  "--cpuexecutioncap", "50",
                  # Make a Structure within Virtualbox to group Machines
                  "--groups", "/Vagrant/LMU/GitLab"
                 ]
  end

  config.vm.define "gitlab-ci-01.zuv.uni-muenchen.de", primary: true, autostart: true do |node|
    node.vm.box = "debian/buster64"
    node.vm.hostname = "gitlab-ci-01"

    node.vm.provider "virtualbox" do |vb|
      vb.name = "GitLab-CI-Runner 01"
      vb.memory = 4096
      vb.cpus = 4
    end
    node.vm.network :public_network, ip: "192.168.6.100", netmask: "255.255.255.0"
  end

  config.vm.define "gitlab-ci-02.zuv.uni-muenchen.de", primary: true, autostart: true do |node|
    node.vm.box = "centos/8"
    node.vm.hostname = "gitlab-ci-02"

    node.vm.provider "virtualbox" do |vb|
      vb.name = "GitLab-CI-Runner 02"
      vb.memory = 4096
      vb.cpus = 4
    end
    node.vm.network :public_network, ip: "192.168.6.100", netmask: "255.255.255.0"
  end

  (3..9).each do |i|
    config.vm.define "gitlab-ci-0#{i}.zuv.uni-muenchen.de", primary: false, autostart: false do |node|
      node.vm.box = "debian/buster64"
      node.vm.hostname = "gitlab-ci-0#{i}"

      node.vm.provider "virtualbox" do |vb|
        vb.name = "GitLab-CI-Runner 0#{i}"
        vb.memory = 2048
        vb.cpus = 2
      end
      node.vm.network "private_network", ip: "192.168.6.#{110 + i}", netmask: "255.255.255.0"
    end
  end

  config.vm.provision "bootstrap", type: "shell" do |s|
    s.inline = "echo Bootstrap Machine"
  end
  config.vm.provision "ssh-key", type: "shell", privileged: false do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = <<-SHELL
      echo #{ssh_pub_key} >> ~/.ssh/authorized_keys
    SHELL
  end
  config.vm.provision "base-preseed", type: "ansible" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "ansible/base-preseed.yml"
    #ansible.verbose = "vvv"
  end
  config.vm.provision "application", type: "ansible" do |ansible| # run: "never"
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "ansible/base-preseed.yml"
    #ansible.playbook = "ansible/gitlab-ci-runner.yml"
    ansible.groups = {
      "gitlab-ci-runner" => ["gitlab-ci-01.zuv.uni-muenchen.de",
                             "gitlab-ci-02.zuv.uni-muenchen.de",
                             "gitlab-ci-03.zuv.uni-muenchen.de",
                             "gitlab-ci-04.zuv.uni-muenchen.de",
                             "gitlab-ci-05.zuv.uni-muenchen.de",
                             "gitlab-ci-06.zuv.uni-muenchen.de",
                             "gitlab-ci-07.zuv.uni-muenchen.de",
                             "gitlab-ci-08.zuv.uni-muenchen.de",
                             "gitlab-ci-09.zuv.uni-muenchen.de",
                            ],
    }
    #ansible.verbose = "vvvv"
    #ansible.verbose = "vvvv"
    #ansible.verbose = "vv"
    #ansible.verbose = "v"
    #ansible.verbose = ""
    #ansible.start_at_task = ""
    #ansible.stop_at_task = ""
    ansible.limit = "all"
    #ansible.tags = ["setup", "configuration", "update"]
    #ansible.skip_tags = ["update"]
    ansible.extra_vars = {
      ansible_connection: 'ssh',
      ansible_ssh_args: '-o ForwardAgent=yes',
#      ansible_ssh_private_key_file: ['~/.ssh/id_rsa'],
      ansible_python_interpreter: "python3"
    }
  end

end
