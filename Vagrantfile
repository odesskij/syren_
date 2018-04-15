# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos/7"

  [1, 2].each do |i|
    config.vm.define "apache_#{i}" do |apache|
      apache.vm.provision "shell", inline: <<-SHELL
         mkdir -p /var/www/html
         yum install -y httpd
         cp /vagrant/vhost.conf /etc/httpd/conf.d/vhost.conf
         echo I\\\'m server #{i} > /var/www/html/index.html
         systemctl enable httpd
         systemctl start httpd
      SHELL

      apache.vm.network "private_network", ip: "192.168.33.1#{i}"
      apache.vm.hostname = "apache#{i}"
    end
  end

  config.vm.define :haproxy do |haproxy|
    haproxy.vm.provision "shell", inline: <<-SHELL
      yum -y update
      yum -y install haproxy
      cp /vagrant/haproxy.cfg /etc/haproxy/haproxy.cfg
      systemctl enable haproxy
      systemctl start haproxy
    SHELL

    haproxy.vm.network "private_network", ip: "192.168.33.13"
    haproxy.vm.network "forwarded_port", guest: 80, host: 8080
    haproxy.vm.hostname = "haproxy"
  end

  config.vm.define :jenkins do |jenkins|

    jenkins.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end

    jenkins.vm.provision "shell", inline: <<-SHELL
      curl -o /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
      rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key

      yum -y update
      yum -y install jenkins java-1.8.0-openjdk

      echo 2.117 > /var/lib/jenkins/jenkins.install.UpgradeWizard.state
      echo 2.117 > /var/lib/jenkins/jenkins.install.InstallUtil.lastExecVersion
      mkdir -p /var/lib/jenkins/jobs/haproxy
      cp /vagrant/jenkins/job_config.xml /var/lib/jenkins/jobs/haproxy/config.xml

      curl -L https://raw.githubusercontent.com/hgomez/devops-incubator/master/forge-tricks/batch-install-jenkins-plugins.sh -o batch-install-jenkins-plugins.sh
      echo workflow-aggregator > plugins.txt
      bash ./batch-install-jenkins-plugins.sh --plugins plugins.txt --plugindir /var/lib/jenkins/plugins

      chown -R jenkins:jenkins /var/lib/jenkins

      systemctl enable jenkins
      systemctl start jenkins
    SHELL

    jenkins.vm.network "private_network", ip: "192.168.33.14"
    jenkins.vm.network "forwarded_port", guest: 8080, host: 8081
    jenkins.vm.hostname = "jenkins"
  end

end
