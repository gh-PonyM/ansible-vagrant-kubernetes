PROVIDER='libvirt'
ENV['VAGRANT_DEFAULT_PROVIDER'] = PROVIDER
IMAGE_NAME = "generic/ubuntu2110"
VAGRANT_API_VERSION = "2"
PLAYBOOK = 'k8s.yml'

machines=[
    {
    :hostname => "controller",
    :ram => 4096,
    :cpu => 3,
    :playbook => "k8s.yml",
    :groups => ["k8s_cluster", "k8s_controller"],
    :ip => "192.168.56.10"
  },
  {
    :hostname => "worker1",
    :ram => 2048,
    :cpu => 3,
    :playbook => "k8s.yml",
    :groups => ["k8s_cluster", "k8s_worker"],
    :ip => "192.168.56.11"
  }
]

Vagrant.configure(VAGRANT_API_VERSION) do |config|

  ansible_groups = Hash.new
  machines.each do |machine|
    if machine.has_key?(:groups)
        machine[:groups].each do |g|
            if ansible_groups.has_key?(g)
                ansible_groups[g].push(machine[:hostname])
            else
                ansible_groups[g] = [machine[:hostname]]
            end
        end
    end
  end

  machines.each do |machine|
    config.vm.define machine[:hostname] do |node|
      node.vm.box = IMAGE_NAME
      node.vm.hostname = machine[:hostname]
      if machine.has_key?(:ip)
        node.vm.network "private_network", ip: machine[:ip]
      end

      # Prevents generating a ssh key for every host individually
      node.ssh.insert_key = false

      node.vm.provider PROVIDER do |vb|
        vb.memory =  machine[:ram]
        vb.cpus = machine[:cpu]
      end
    end
  end

  # After bringing up all machines, provision the first, but select all hosts
  config.vm.define machines[0][:hostname] do |node|
    node.vm.provision "ansible" do |ansible|
      ansible.playbook = PLAYBOOK

      # We set that here since it's vagrant specific
      # Avoid using the same hostnames for vagrant vm's than in your normal inventory
      # because of the ansible facts cache
      ansible.extra_vars = {
         "k8s_interface": "eth1"
      }
      ansible.groups = ansible_groups

      # This is crucial in order to run the playbook on all hosts in parallel
      ansible.limit = 'all'
      ansible.raw_arguments = ["-t", "cluster_join"]
    end
  end
end
