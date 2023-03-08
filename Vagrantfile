# -*- mode: ruby -*-

# No need in a single VM installation, though
ENV["VAGRANT_NO_PARALLEL"] = "yes"


Vagrant.configure("2") do |config|
  # Make sure that version of the vagrant-registration plugin is 1.3.4, weird issues when it's not
  config.vagrant.plugins = [ "vagrant-registration" => { :version => "1.3.4" } ]

  if Vagrant.has_plugin?('vagrant-registration')
    config.registration.unregister_on_halt = false
    config.registration.auto_attach = true
  end
  
  config.vm.box = "generic/rhel9"
  config.ssh.connect_timeout = 90

  config.vm.provider :libvirt do |libvirt|
    libvirt.qemu_use_session = false
  end

# Want to keep installation in a single VM, so we won't install HUB
#  # Can't use FQDN that contains - or _ https://access.redhat.com/solutions/6960629
#  config.vm.define "hublocalhost" do |v|
#    if Vagrant.has_plugin?("vagrant-registration")
#      v.registration.username = ENV["SPONGE_RHSM_USERNAME"]
#      v.registration.password = ENV["SPONGE_RHSM_PASSWORD"]
#      v.registration.pools = [ ENV["SPONGE_RHSM_POOL_BASE"], ENV["SPONGE_RHSM_POOL_AAP"] ]
#      v.registration.name = "hublocalhost"
#    end
#    v.vm.hostname = "hub"
#    v.vm.network :forwarded_port, guest: 443, host: 62443, host_ip: "0.0.0.0"
#    v.vm.provider "libvirt" do |h|
#      h.cpus = 1
#      h.memory = 8320
#    end
#  end

  config.vm.define "aaplocalhost" do |v|
    if Vagrant.has_plugin?("vagrant-registration")
      v.registration.username = ENV["SPONGE_RHSM_USERNAME"]
      v.registration.password = ENV["SPONGE_RHSM_PASSWORD"]
      v.registration.pools = [ ENV["SPONGE_RHSM_POOL_BASE"] ]
      v.registration.name = "aaplocalhost"
    end
    config.vm.provider "libvirt" do |h|
      h.cpus = 4
      h.memory = 8320
    end
    v.vm.hostname = "aaplocalhost"
    v.vm.network :forwarded_port, guest: 443, host: 63443, host_ip: "0.0.0.0"
    v.vm.provision "ansible" do |ansible|
      ansible.become = true
      ansible.compatibility_mode = "2.0"
      ansible.galaxy_role_file = "requirements.yml"
      ansible.groups = {
        "all:vars" => {
          # Single node install https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.2/html/red_hat_ansible_automation_platform_installation_guide/single-machine-scenario
          :bundle_install => "False",
          :create_preload_data => "False",
          # Default is False
          :upgrade_ansible_with_tower => "False",
          # FROM group_vars/all
          :tower_package_name => "automation-controller",
          :tower_package_version => "4.2.1",
          :minimum_ansible_version => "2.12",
          # FROM inventory
          "admin_password" => ENV["SPONGE_AAP_CONTROLLER_ADMIN_PASSWORD"],
          "pg_host" => "aaplocalhost",
          "pg_port" => "5432",
          "pg_database" => "awx",
          "pg_username" => "awx",
          "pg_password" => ENV["SPONGE_AAP_CONTROLLER_PG_PASSWORD"],
          "pg_sslmode" => "prefer",
          "registry_verify_ssl" => false,
          "registry_url" => "registry.redhat.io",
          "registry_username" => ENV["SPONGE_AAP_REGISTRY_USERNAME"],
          "registry_password" => ENV["SPONGE_AAP_REGISTRY_PASSWORD"],
          "receptor_listener_port" => 27199,
          # No HUB
          #"automationhub_admin_password" => ENV["SPONGE_AAP_HUB_ADMIN_PASSWORD"],
          #"automationhub_pg_host" => "aaplocalhost",
          #"automationhub_pg_port" => "5432",
          #"automationhub_pg_database" => "automationhub",
          #"automationhub_pg_username" => "automationhub",
          #"automationhub_pg_password" => ENV["SPONGE_AAP_HUB_PG_PASSWORD"],
          #"automationhub_pg_sslmode" => "prefer",
          #"automationhub_upgrade" => "True",
          #"sso_console_admin_password" => ENV["SPONGE_AAP_SSO_ADMIN_PASSWORD"],
          #"sso_keystore_password" => ENV["SPONGE_AAP_SSO_KEYSTORE_PASSWORD"]
        },
        "automationcontroller" => ["aaplocalhost"],
        #"automationhub" => ["hublocalhost"],
        "database" => ["aaplocalhost"],
        # No SSO
        #"sso" => ["sso"],
        #"sso:vars" => {
        #}
      }
      ansible.host_vars
      ansible.limit = "all,localhost"
      ansible.playbook = "playbook.yml"
    end
  end
end
