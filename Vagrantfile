# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = '2'

MGRNO   = 1
MONNO   = 1
OSDNO   = 3
SUBNET  = '192.168.101'

ansible_provision = proc do |ansible|
  ansible.playbook = 'site.yml'
  ansible.groups = {
    'admin' => "admin",
    'mgrs' => (0..MGRNO - 1).map { |j| "mgr#{j}" },
    'mons' => (0..MONNO - 1).map { |j| "mon#{j}" },
    'osds' => (0..OSDNO - 1).map { |j| "osd#{j}" },
  }

  ansible.extra_vars = {
    fsid: '10c95f01-2dd2-4863-affa-60c4eafcd8d2',
    monitor_secret: 'AQBNTxZRWId7JxAA/Ac4ToR7ZfNdOGDSToGHpA=='
  }
  ansible.limit = 'all'
end

def create_vmdk(name, size)
  dir = Pathname.new(__FILE__).expand_path.dirname
  path = File.join(dir, '.vagrant', name + '.vmdk')
  `vmware-vdiskmanager -c -s #{size} -t 0 -a scsi #{path} \
   2>&1 > /dev/null` unless File.exist?(path)
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = 'centos/7'
  config.ssh.pty = true
  config.ssh.insert_key = false # workaround for https://github.com/mitchellh/vagrant/issues/5048

  # copy ssh keys
  config.vm.provision "file", source: "keys/id_rsa", destination: "/home/vagrant/.ssh/id_rsa"
  public_key = File.read("keys/id_rsa.pub")
  config.vm.provision :shell, :inline =>"
      echo 'Copying ansible-vm public SSH Keys to the VM'
      mkdir -p /home/vagrant/.ssh
      chmod 700 /home/vagrant/.ssh
      echo '#{public_key}' >> /home/vagrant/.ssh/authorized_keys
      chmod -R 600 /home/vagrant/.ssh/authorized_keys
      echo 'Host 192.168.*.*' >> /home/vagrant/.ssh/config
      echo 'StrictHostKeyChecking no' >> /home/vagrant/.ssh/config
      echo 'UserKnownHostsFile /dev/null' >> /home/vagrant/.ssh/config
      chmod -R 600 /home/vagrant/.ssh/config
      chmod 600 /home/vagrant/.ssh/id_rsa
      ", privileged: false

  config.vm.define "admin" do |admin|
    admin.vm.network :private_network, ip: "#{SUBNET}.30"
    admin.vm.provision :hosts, :sync_hosts => true
    admin.vm.provider :virtualbox do |vb|
      vb.customize ['modifyvm', :id, '--memory', '192']
    end
    admin.vm.provider :vmware_fusion do |v|
      v.vmx['memsize'] = '192'
    end
  end

  (0..MONNO - 1).each do |i|
    config.vm.define "mon#{i}" do |mon|
      mon.vm.network :private_network, ip: "#{SUBNET}.1#{i}"
      mon.vm.provision :hosts, :sync_hosts => true
      mon.vm.provider :virtualbox do |vb|
        vb.customize ['modifyvm', :id, '--memory', '192']
      end
      mon.vm.provider :vmware_fusion do |v|
        v.vmx['memsize'] = '192'
      end
    end
  end
  
  (0..MGRNO - 1).each do |i|
      config.vm.define "mgr#{i}" do |mgr|
      mgr.vm.network :private_network, ip: "#{SUBNET}.2#{i}"
      mgr.vm.provision :hosts, :sync_hosts => true
      mgr.vm.provider :virtualbox do |vb|
        vb.customize ['modifyvm', :id, '--memory', '192']
      end
        mgr.vm.provider :vmware_fusion do |v|
          v.vmx['memsize'] = '192'
      end
    end
  end

  (0..OSDNO - 1).each do |i|
    config.vm.define "osd#{i}" do |osd|
      osd.vm.network :private_network, ip: "#{SUBNET}.10#{i}"
      osd.vm.network :private_network, ip: "#{SUBNET}.20#{i}"
      osd.vm.provision :hosts, :sync_hosts => true
      osd.vm.provider :virtualbox do |vb|
        vb.customize ['storagectl', :id, '--name', "SATA-#{i}", '--add', 'sata', '--portcount', 1]
        (0..1).each do |d|
          unless File.exist?(File.expand_path("disk-#{i}-#{d}.vdi"))
            vb.customize ['createhd', '--filename', "disk-#{i}-#{d}", '--size', '11000']
            vb.customize ['storageattach', :id,
                          '--storagectl', "SATA-#{i}",
                          '--port',  d,
                          '--device', 0,
                          '--type', 'hdd',
                          '--medium', "disk-#{i}-#{d}.vdi"]
            end 
          end
        vb.customize ['modifyvm', :id, '--memory', '512']
      end

      osd.vm.provider :vmware_fusion do |v|
        (0..1).each do |d|
          v.vmx["scsi0:#{d + 1}.present"] = 'TRUE'
          v.vmx["scsi0:#{d + 1}.fileName"] =
            create_vmdk("disk-#{i}-#{d}", '11000MB')
        end
        v.vmx['memsize'] = '192'
      end

      osd.vm.provision 'ansible', &ansible_provision if i == (OSDNO - 1)
    end
  end
end
