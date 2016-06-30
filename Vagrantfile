# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

if Vagrant::VERSION == "1.7.2"
  synced_folder_files = Dir.glob(File.expand_path("../.vagrant/machines/**/synced_folders", __FILE__))
  synced_folder_files.map do |synced_folder_file|
    File.unlink(synced_folder_file) if File.exists?(synced_folder_file)
  end
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "dummy"
  config.omnibus.chef_version = :latest

  config.vm.synced_folder ".", "/vagrant", type: "rsync",
    rsync__exclude: [
      '.vagrant/',
      '.git/',
      'tmp/',
      'packer_cache/'
  ]

  ## AWS
  config.vm.provider :aws do |aws, override|
    aws.access_key_id = ENV['AWS_ACCESS_KEY']
    aws.secret_access_key = ENV['AWS_SECRET_ACCESS_KEY']
    aws.keypair_name = ENV['AWS_EC2_KEYPAIR']
    aws.user_data = "#!/bin/bash\nsed -i -e 's/^Defaults.*requiretty/# Defaults requiretty/g' /etc/sudoers"

    aws.region = ENV['AWS_REGION']
    aws.security_groups = ['elasticsearch']
    aws.instance_type = 'c3.large'
    case ENV['AWS_REGION']
    when 'ap-northeast-1'
      aws.ami = 'ami-f80e0596' # Amazon Linux AMI 2016.03.0 (HVM), SSD Volume Type
    when 'us-east-1'
      aws.ami = 'ami-08111162' # Amazon Linux AMI 2016.03.0 (HVM), SSD Volume Type
    else
      raise "Unsupported region #{ENV['AWS_REGION']}"
    end

    aws.tags = {
      'Name' => "Docker & Elasticsearch #{ENV['PRODUCT_VERSION']} (Developed by #{ENV['USER']})"
    }
    override.ssh.username = "ec2-user"
    override.ssh.private_key_path = ENV['AWS_EC2_KEYPASS']
  end

  config.ssh.pty = true

  ## Sction Provisioning
  config.vm.provision "shell", inline: <<-SHELL
    sudo rpm --import http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-6
    sudo yum -y install http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    sudo yum -y install docker-io
    sudo yum -y install jq
    sudo service docker start
    export DOCKER_HOST=unix:///var/run/docker.sock
    sudo docker pull elasticsearch
    sudo docker run -d -p 9200:9200 -p 9300:9200 -v "$PWD/esdata":/usr/share/elasticsearch/data elasticsearch
  SHELL

end
