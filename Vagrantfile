
# -*- mode: ruby -*-
# vi: set ft=ruby :

##
#/opt/vagrant/embedded/gems/2.2.6/gems/vagrant-2.2.6/plugins/providers/virtualbox/action/
##

class VagrantPlugins::ProviderVirtualBox::Action::Boot
        alias_method :original_call, :call
        def call(env)
          machine = env[:machine]
          driver = machine.provider.driver
          uuid = driver.instance_eval { @uuid }
          ui = env[:ui]
          controller_name="SATAController "
	  volume1="part1_volumegroup.vdi"
		  
          vm_info = driver.execute("showvminfo", uuid)
          name = vm_info.match(".*Name:.*").to_s.scan(/elk-node-+\w/)[0].to_s
	  ui.info vm_info
          ui.info "-------------------------------------"
	  ui.info name 
          ui.info uuid	
          ui.info "-------------------------------------"

          has_this_controller = vm_info.match("Storage Controller Name.*#{controller_name}")
	  has_first_volume = vm_info.match("#{controller_name}.*#{volume1}")


	  current_dir    = File.dirname(File.expand_path(__FILE__))     
	  volume1prefix ="part1_volumegroup-#{name}.vdi"		
	  ui.info "creating disks if needed"		
	  disk1 = "%s/disk/%s" % [current_dir,volume1prefix] 

	  if  File.exist?(disk1)
		ui.info "already has the #{disk1} disk"
	  else
		ui.info "creating  #{disk1} disk"
		driver.execute('createmedium',"disk",'--filename',
		disk1,'--format', 'VDI','--size', "102400")	
	  end

		
          if has_this_controller
            	ui.info "already has the #{controller_name} hdd controller"
          else
            ui.info "creating #{controller_name} controller #{controller_name}"
            driver.execute('storagectl', uuid,
              '--name', "#{controller_name}",
              '--add', 'sata',
              '--controller', 'IntelAhci')
          end
		  
	  if (has_first_volume)
		ui.info "already has the #{volume1}  volume in controller"
	  else
            	ui.info "attaching storage #{volume1} to #{controller_name}"
          	driver.execute('storageattach', uuid,
              '--storagectl', "#{controller_name}",
              '--port', '1',
              '--device', '0',
              '--type', 'hdd',
              '--medium', disk1)			 			  
		end
		
		
          original_call(env)
        end
end
#max N =2
Vagrant.configure("2") do |config|
N = 1
(1..N).each do |machine_id|
  config.vm.define "maquina#{machine_id}" do |machine|
    	machine.vm.box = "centos/7"
        machine.vm.hostname = "lvm-node-#{machine_id}"
    	machine.vm.network "private_network", ip: "192.168.40.#{20+machine_id}"
    	machine.ssh.forward_agent = true
	machine.vm.box_check_update = false
	machine.ssh.username = 'vagrant'
	#machine.ssh.password = 'vagrant'
	machine.ssh.insert_key = 'true'
	machine.vbguest.auto_update = false
	machine.vm.synced_folder ".", "/vagrant", disabled: true

	#puts("enter once")	
    	machine.vm.provider "virtualbox" do |vb|
        #total cpu in my pc = 8 
        if machine_id == 1
        vb.cpus = 1
        else
        vb.cpus = 1
	end
        vb.memory = 2048
        vb.gui = false
        vb.name = "lvm-node-#{machine_id}"
    	
        #https://www.virtualbox.org/manual/ch08.html#vboxmanage-modifyvm
        #vb.customize "pre-boot",["modifyvm", :id, "--nictype1", "Am79C973"]
        #vb.customize "pre-boot",["modifyvm", :id, "--cableconnected1", "on"]
		#puts("enter twice")	

    end
	
		  
    # Only execute once the Ansible provisioner,
    # when all the machines are up and ready.
    #if machine_id == N
    #  machine.vm.provision :ansible do |ansible|
    #    ansible.limit = "all"
    #    ansible.playbook = "playbook.yml"
    #    ansible.verbose = true
    #	ansible.host_key_checking = false
    #  end
    #end 
	
  end
end
end

