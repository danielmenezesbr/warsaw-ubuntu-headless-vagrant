# -*- mode: ruby -*-
# vi: set ft=ruby :

# TODO:
#   teste_caixa.py
#       verificar melhor local (dentro ou fora do container)
#   documentação:
#       fazer vídeo demonstrativo.
#       create readme.md

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/xenial64"

  config.vm.network "forwarded_port", guest: 5900, host: 5900
  config.vm.network "forwarded_port", guest: 4444, host: 4444

  config.vbguest.auto_update = false
  # Prevent TTY Errors. By default this is "bash -l"
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  config.vm.provider "virtualbox" do |vb|
     vb.gui = false
     vb.memory = "1024"
  end

  config.vm.synced_folder ".", "/vagrant", disabled: true

  # faz o provisiomento da VM com as configurações e softwares necessários para ter acesso ao CEF e BB
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
     sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
     sudo apt-add-repository 'deb https://apt.dockerproject.org/repo ubuntu-xenial main'
     sudo apt-get update
     sudo apt-get install make python-pip docker-engine docker-compose -y --no-install-recommends --allow-unauthenticated
     sudo systemctl status docker
     git clone https://github.com/danielmenezesbr/docker-selenium
     cd docker-selenium
     git checkout warsaw
     sudo make build
     sudo docker run -d -p 4444:4444 -p 5900:5900 selenium/standalone-firefox-debug:3.4.0-dysprosium
     sudo pip install selenium
     cat >~/teste_caixa.py <<EOF
import urllib2
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
import time
import urllib2

achou = False
while (not achou):
    try:
        conteudo = urllib2.urlopen("http://localhost:4444/wd/hub").read()
        if "<title>WebDriver Hub</title>" in conteudo:
            achou = True
            print "Selenium pronto. Conecte via VNC: localhost:5900"
            print "Senha: secret"
    except:
        print "Aguardando selenium..."
        time.sleep(30)
        pass

driver = webdriver.Remote(
   command_executor='http://localhost:4444/wd/hub',
   desired_capabilities=DesiredCapabilities.FIREFOX)
driver.get("http://www.caixa.gov.br")
EOF
    cd ~
    python teste_caixa.py
  SHELL

end
