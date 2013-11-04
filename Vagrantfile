# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "Ubuntu precise 32 VirtualBox"
  config.vm.box_url = "http://files.vagrantup.com/precise32.box"

  $script = <<EOF
echo "Atualizando repositorios apt..."
sudo apt-get update

echo "Instalando openjdk-6..."
sudo apt-get install openjdk-6-jdk -q -y --fix-missing

echo "Instalando unzip e vim..."
sudo apt-get install -y unzip vim vim-gtk
EOF

  config.vm.provision :shell, :inline => $script

  config.vm.define :buildserver do |buildserver_config|
    buildserver_config.vm.hostname = "d5800sd120.hml.5800bseguros.com.br"
    buildserver_config.vm.network :private_network, ip: "172.168.0.120"

    buildserver_config.vm.synced_folder "~/.m2", "/opt/ips/.m2"
    buildserver_config.vm.synced_folder "sync/apps", "/opt/ips/apps"
    buildserver_config.vm.synced_folder "conf", "/configuration"
    buildserver_config.vm.synced_folder "sync/hudson_home", "/opt/ips/config/build"

    buildserver_config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "3000", "--ioapic", "on"]
    end
    
    $script = <<EOF
if [ ! -e "/usr/lib/jvm/jdk1.6.0_45" ]; then
    echo "Instalando java oracle 1.6..."
    export JDK_URL="http://download.oracle.com/otn-pub/java/jdk/6u45-b06/jdk-6u45-linux-i586.bin"
    if [ ! -e "/tmp/jdk.bin" ]; then
        wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F" "$JDK_URL" -O /tmp/jdk.bin
    fi
    #unzip /tmp/jdk.bin -d /tmp
    cd /tmp
    sh jdk.bin
    sudo mv /tmp/jdk1.6.0_45 /usr/lib/jvm/
    sudo ln -s /usr/lib/jvm/jdk1.6.0_45 /usr/lib/jvm/java-6-oracle
    sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-6-oracle/jre/bin/java 1
    sudo update-alternatives --set java /usr/lib/jvm/java-6-oracle/jre/bin/java
    echo "export JAVA_HOME=/usr/lib/jvm/java-6-oracle" | sudo tee /etc/profile.d/jdk.sh
fi

echo "Configurando maven..."
echo "export MAVEN2_DIR=/opt/ips/maven" | sudo tee /etc/profile.d/maven.sh
echo "export MAVEN3_DIR=/opt/ips/maven3" | sudo tee -a /etc/profile.d/maven.sh
echo "export MAVEN_OPTS=-Xmx1024m" | sudo tee -a /etc/profile.d/maven.sh
echo "export M2_REPO=/opt/ips/.m2/repository" | sudo tee -a /etc/profile.d/maven.sh
echo "export M2_HOME=/opt/ips/maven" | sudo tee -a /etc/profile.d/maven.sh
echo "export M3_HOME=/opt/ips/maven3" | sudo tee -a /etc/profile.d/maven.sh
echo "export MAVEN_HOME=/opt/ips/maven" | sudo tee -a /etc/profile.d/maven.sh
echo "export PATH=\\$PATH:/opt/ips/maven/bin" | sudo tee -a /etc/profile.d/maven.sh


if [ ! -e "/opt/ips/maven" ]; then
    echo "Instalando maven2..."
    MAVEN="apache-maven-2.2.1-bin.tar.gz"
    MAVEN_URL="http://ftp.unicamp.br/pub/apache/maven/maven-2/2.2.1/binaries/$MAVEN"
    wget "$MAVEN_URL" -O /tmp/$MAVEN
    tar zxvf /tmp/$MAVEN -C /tmp
    sudo mv /tmp/apache-maven-2.2.1 /opt/ips/maven
fi

if [ ! -e "/opt/ips/maven3" ]; then
    echo "Instalando maven3..."
    MAVEN="apache-maven-3.0.5-bin.tar.gz"
    MAVEN_URL="http://ftp.unicamp.br/pub/apache/maven/maven-3/3.0.5/binaries/$MAVEN"
    wget "$MAVEN_URL" -O /tmp/$MAVEN
    tar zxvf /tmp/$MAVEN -C /tmp
    sudo mv /tmp/apache-maven-3.0.5 /opt/ips/maven3
fi

if [ ! -e "/opt/ips/csvn" ]; then
    echo "Instalando CSVN..."
    CSVN_URL="http://downloads-guests.open.collab.net/files/documents/61/5491/CollabNetSubversionEdge-2.2.0_linux-x86.tar.gz"
    wget -c "$CSVN_URL" -O /tmp/csvn.tar.gz
    tar zxvf /tmp/csvn.tar.gz -C /tmp
    sudo mv /tmp/csvn /opt/ips/
    echo "export PATH=\\$PATH:/opt/ips/csvn/bin" | sudo tee /etc/profile.d/csvn.sh
fi

# Inicia servico no boot
sudo cp /opt/ips/apps/ips-boot-start.sh /etc/init.d/build
sudo chown root:root /etc/init.d/build
sudo update-rc.d build defaults
sudo update-rc.d build enable

# Configura file descriptors e ip forwarding)
sudo cp /configuration/sysctl.conf /etc/
sudo cp /configuration/limits.conf /etc/security/

# Configura /etc/hosts
sudo cp /configuration/hosts /etc/
EOF

    buildserver_config.vm.provision :shell, :inline => $script

  end


  config.vm.define :ldap do |ldap_config|
    ldap_config.vm.hostname = "ldap-matriz.bradseg.com.br"
    ldap_config.vm.network :private_network, ip: "172.168.0.100"
    ldap_config.vm.synced_folder "conf", "/configuration"
    ldap_config.ssh.forward_x11 = true

    ldap_config.vm.provider :virtualbox do |vb|
      vb.gui = false
      vb.customize ["modifyvm", :id, "--memory", "512", "--ioapic", "on"]
    end

    $script = <<EOF
echo "Instalando OpenLdap Server..."
sleep 2
sudo bash -c "DEBIAN_FRONTEND=noninteractive aptitude install -q -y slapd ldap-utils"
sleep 2
echo "Configurando OpenLdap Server..."
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f /configuration/ldap/olcSuffix.ldif
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f /configuration/ldap/rootDn.ldif
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f /configuration/ldap/olcRootPW.ldif
echo "Criando dominio, organizacoes, grupos e usuarios no OpenLdap Server..."
ldapadd -x -w 123456 -D cn=admin,dc=5800bseguros,dc=com,dc=br -f /configuration/ldap/domain.ldif
ldapadd -x -w 123456 -D cn=admin,dc=5800bseguros,dc=com,dc=br -f /configuration/ldap/organizations.ldif
ldapadd -x -w 123456 -D cn=admin,dc=5800bseguros,dc=com,dc=br -f /configuration/ldap/groups.ldif
ldapadd -x -w 123456 -D cn=admin,dc=5800bseguros,dc=com,dc=br -f /configuration/ldap/users.ldif


if [ ! -e "/opt/ApacheDirectoryStudio" ]; then
    echo "Instalando Apache Directory Studio..."
    ADS_URL="http://ftp.unicamp.br/pub/apache//directory/studio/dist/2.0.0.v20130628/ApacheDirectoryStudio-linux-x86-2.0.0.v20130628.tar.gz"
    cd /tmp
    wget "$ADS_URL" -O /tmp/ads.tar.gz
    tar zxvf ads.tar.gz
    sudo mv /tmp/ApacheDirectoryStudio-linux-x86-2.0.0.v20130628 /opt/ApacheDirectoryStudio
    echo "export GDK_NATIVE_WINDOWS=1" > /opt/ApacheDirectoryStudio/start.sh
    echo "/opt/ApacheDirectoryStudio/ApacheDirectoryStudio" >> /opt/ApacheDirectoryStudio/start.sh
    chmod +x /opt/ApacheDirectoryStudio/start.sh
fi
EOF

    ldap_config.vm.provision :shell, :inline => $script

    # ldap_config.vm.provision :chef_solo do |chef|
    #   chef.add_recipe "openldap::server"
    #   chef.json = {
    #     :basedn => "dc=5800bseguros,dc=com,dc=br",
    #     :rootpw => "{SSHA}f3q/7JM5KDENuvDOyyiw5tVwl0LilpUq",
    #     :manage_ssl => false,
    #     :tls_enabled => false
    #   }
    # end

  end
end
