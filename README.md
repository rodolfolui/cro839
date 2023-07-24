# Resilency Orchestration 8.3.9 container

[![License](https://img.shields.io/badge/License-Apache2-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0) [![Slack](https://img.shields.io/badge/Join-Slack-blue)](https://callforcode.org/slack) [![Website](https://img.shields.io/badge/View-Website-blue)](https://code-and-response.github.io/Project-Sample/)

*Read this in other languages: [English](README.md), [Español](README.es_co.md).*

## Introduction to Resilency Orchestration 8.3.9

[![Watch the video](https://higherlogicdownload.s3.amazonaws.com/IMWUC/DeveloperWorksImages_blog-storageneers/ScreenShot2018-10-03at4.32.30PM.png)](https://youtu.be/YRKI2deypuI)


## Prepare CRO training environment:

### Prepare CRO environment with docker/podman

Prepare a Virtual Machine with RHEL 8.x/9.x (Linux Red Hat Enterprise) or Ubuntu with Docker or Podman

    1. On Apple Mac use docker for Mac and command line
    2. On windows use docker for windows and select linux containers enablinbg hyper-v virtualization. It is easier to use VirtualBox and start Ubuntu VM with docker
    3. On RHEL 8.x podman is installed by default and is fully command compatible with docker commands.  For scenarios where need docker-compose you can use podman-compose

### Pre-requisite preparation

    1. Import and run RHEL 8 UBI (minimal) base image
        $ docker run -it --name cro -h crort -p 8080:8080 registry.access.redhat.com/ubi8/ubi bash

        or RHEL 9 UBI

        $ docker run -it --name cro -h crort -p 8080:8080 registry.access.redhat.com/ubi9/ubi bash
        
    2. Install library
       [root@crort /]# dnf -y install hostname libXtst net-tools iputils procps-ng rsync sudo unzip wget
       
    3. Download Tomcat 9.0.54 and install on /opt
       [root@crort /]# wget -qO- https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.54/bin/apache-tomcat-9.0.54.tar.gz | tar -xzf - -C /opt
       
    4. Download MariaDB 10.5.9 installation rpm's
       [root@crort /]# wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-shared-10.5.9-1.el8.x86_64.rpm
       [root@crort /]# wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-common-10.5.9-1.el8.x86_64.rpm
       [root@crort /]# wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-server-10.5.9-1.el8.x86_64.rpm
       [root@crort /]# wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-client-10.5.9-1.el8.x86_64.rpm
       [root@crort /]# wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/galera-4-26.4.7-1.el8.x86_64.rpm

    5. Download socat and boost-program-options (required for galera clustering)
       [root@crort /]# wget http://mirror.centos.org/centos/8-stream/AppStream/x86_64/os/Packages/socat-1.7.4.1-1.el8.x86_64.rpm
       [root@crort /]# wget https://vault.centos.org/centos/8/AppStream/x86_64/os/Packages/boost-program-options-1.66.0-10.el8.x86_64.rpm

    6. Install MariaDB, start container and change root password
       [root@crort /]# rpm -Uvh boost-program-options-1.66.0-10.el8.x86_64.rpm
       [root@crort /]# dnf --disableplugin=subscription-manager install -y MariaDB-server galera MariaDB-client MariaDB-shared MariaDB-backup MariaDB-common
       [root@crort /]# /etc/init.d/mysql start
       [root@crort /]# mysqladmin -f -u root password '<new_password>'

### Resiliency Orchestration Install

    1. On other linux shell (Outside container), copy product installation binaries (Server.tar.gz) inside the container:
       $ docker copy Server.tar.gz crort:/tmp
       
    2. Execute RO installation steps starting on page 77 in console mode:
       [root@crort /]# cd tmp
       [root@crort /]# tar xvzf Server.tar.gz
       [root@crort /]# sed 's/LOCAL_HOST_SERVER=/LOCAL_HOST_SERVER=10.0.0.2/
       > s/DATABASE_PASSWORD=/DATABASE_PASSWORD=Passw0rd/
       > s/LICENSE_ACCEPTED=/LICENSE_ACCEPTED=TRUE/
       > s/SUPPORT_USER_PASSWORD=/SUPPORT_USER_PASSWORD=Orchestration123./
       > s/SANOVI_USER_PASSWORD=/SANOVI_USER_PASSWORD=Orchestration123./
       > s/MODIFY_SYSTEM_FILES=0/MODIFY_SYSTEM_FILES=1/
       > s/TOMCAT_HOME=.*/TOMCAT_HOME=\/opt\/apache-tomcat-9.0.54/' /tmp/PanacesServerInstaller.properties > /tmp/PanacesServerCro.properties
       [root@crort /]# /tmp/install.bin -f /tmp/PanacesServerCro.properties
       
    3. Execute post-installation steps on RO Install guide on page 85 (numeral 5.6) console mode

### Start RO services

    1. Start RO services in container:
       export EAMSROOT=/opt/panaces
       /etc/init.d/mysql status || /etc/init.d/mysql start
       cd /opt/panaces/bin
       ./panaces restart
       
    2. On other linux shell (Outside container), save running container image to persist installation (containers are not persistent by nature)
       $ docker commit crort cro:8.3.9
    
And that's up! already have a base image of RO we can use for training or test
    $ docker images

REPOSITORY                           TAG             IMAGE ID      CREATED        SIZE

localhost/cro                        8.3.9         3d5162fe511f  19 hours ago   3.98 GB


### Execution of automated scripts

There are 4 scripts to execute previous steps in a quick way on a RHEL system with internet access:

- comandos839        Installation shell script (requires a local folder named 839 to copy installers)
- Dockerfile.cro839  Dockerfile to build RO 8.3.9 images
- Dockerfile.mariadb Dockerfile to build MariaDB 10.5.9 image
- instcro            RO installation script

       [root@crort /]# ./comandos839
       cro839
       EULA.txt
       install.bin
       PanacesServerInstaller.properties
       base1
       Error: no container with name or ID "base1" found: no such container
       STEP 1/17: FROM registry.access.redhat.com/ubi8/ubi
       STEP 2/17: ENV HOME=/var/lib/mysql     SUMMARY="MariaDB 10.5.9 Tomcat 9.0.54 Panaces 8.3.9"     DESCRIPTION="Resilency Orchestration 8.3.9"
       --> c94d5c9735f
       STEP 2/19: ENV HOME=/var/lib/mysql     SUMMARY="MariaDB 10.5.9 Tomcat 9.0.54 Panaces 8.3.9"     DESCRIPTION="Resilency Orchestration 8.3.9"
       --> Using cache c94d5c9735fc39773070cd2f740d5dcb549c0e212e946f91aa179c94251c88c8
       --> c94d5c9735f
       STEP 3/19: LABEL summary="$SUMMARY"       description="$DESCRIPTION"       io.k8s.description="$DESCRIPTION"       io.k8s.display-name="CRO 8.3.9"       io.openshift.expose-services="3306:mysql,8080:tomcat"       io.openshift.tags="database,mysql,mariadb,mariadb105,mariadb-105"       com.redhat.component="mariadb-105-container"       name="cro829"       version="1"       usage="docker run -it -e DISPLAY=$DISPLAY --network host -v /tmp/.X11-unix:/tmp/.X11-unix -p 8080:8080 -p 3306:3306 -h crort --name crort cro80 bash"       maintainer="Kyndryl Colombia <rvasquez@kyndryl.com>"
       --> Using cache 00b571fcae3f93db60a572b9cc81fb56e20cd32308336562d5e4bd4173909e11
       --> 00b571fcae3
       STEP 4/19: RUN yum --disableplugin=subscription-manager install -y yum-utils &&     INSTALL_PKGS="hostname libXtst net-tools bind-utils iputils iproute lsof perl-DBI libaio libsepol lsof openssl procps-ng rsync sudo unzip wget" &&     yum --disableplugin=subscription-manager install -y --setopt=tsflags=nodocs $INSTALL_PKGS &&     rpm -V $INSTALL_PKGS &&     yum clean all
       --> Using cache 60a8e971f53e1464ae0ee11337105c19251c0835e597bd97d6cb48d01fe34bdf
       --> 60a8e971f53
       STEP 5/19: VOLUME ["/install"]
       --> Using cache e4af9cbf0f8031b570217fbe53f7a7838866a583e73dda06bd867c8ce6b747ac
       --> e4af9cbf0f8
       STEP 6/19: RUN wget -qO- https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.54/bin/apache-tomcat-9.0.54.tar.gz | tar -xzf - -C /opt 
       --> Using cache d50df6ac80363d31cf4cd11f7b1a1986c4629be3541dcb9c005701a7f8482d5b
       --> d50df6ac803
       STEP 7/19: RUN wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-shared-10.5.9-1.el8.x86_64.rpm
       --> Using cache 862ebe0e9b2f38432a4b1cd169b58c0dbfb856b727afe39e7d66dac4660ad328
       --> 862ebe0e9b2
       STEP 8/19: RUN wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-common-10.5.9-1.el8.x86_64.rpm
       --> Using cache 3c70b0f0ad0d202e5bfa2689866029a174e0f51b8f00a4ce80b6cf2467059b30
       --> 3c70b0f0ad0
       STEP 9/19: RUN wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-server-10.5.9-1.el8.x86_64.rpm
       --> Using cache 111f0191241c2b655c81f2b04da318922e5bacf690af24750b128129590327c2
       --> 111f0191241
       STEP 10/19: RUN wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-client-10.5.9-1.el8.x86_64.rpm
       --> Using cache 179233c9c1853008e95e412d91afdeae67496d5cc436d7953181e1ced79ca69a
       --> 179233c9c18
       STEP 11/19: RUN wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/galera-4-26.4.7-1.el8.x86_64.rpm
       --> Using cache 734d1bbec81ecdbd653cf19466a7234051a57ac5ee532d16727e3dfdb2a5a2c7
       --> 734d1bbec81
       STEP 12/19: RUN wget http://mirror.centos.org/centos/8-stream/AppStream/x86_64/os/Packages/socat-1.7.4.1-1.el8.x86_64.rpm
       --> Using cache 8cdb3db3a4dd80155a2cc459ed342e3ec2ae43a10845f34952c3552c97a8a3c4
       --> 8cdb3db3a4d
       STEP 13/19: RUN wget https://vault.centos.org/centos/8/AppStream/x86_64/os/Packages/boost-program-options-1.66.0-10.el8.x86_64.rpm
       --> Using cache aa1a60cf153f3ab63607c220af841223eeba921efe9cfd0c0317f051d86a8a93
       --> aa1a60cf153
       STEP 14/19: RUN yum localinstall -y socat* boost* galera-4* MariaDB-* && rm *.rpm
       --> Using cache 6ac8e8f3cef773c73c7851900e29e0e76bd82d998397bc778ef4cfad1d9ba4a6
       --> 6ac8e8f3cef
       STEP 15/19: COPY instcro /
       --> Using cache 283dad31c14d9c3c8f01bb12aa8b1ed97de79b2cd42869fee464beabac51f0fa
       --> 283dad31c14
       STEP 16/19: RUN echo "SELINUX=permissive" >> /etc/selinux/config
       --> Using cache 57d09330768e51bdff0329845247b3e344c845f203655bf99fd130d5654b89ce
       --> 57d09330768
       STEP 17/19: EXPOSE 3306/tcp
       --> badcfa333df
       STEP 18/19: EXPOSE 8080/tcp
       --> 306766a64ab
       STEP 19/19: CMD ["mysqld"]
       47904c383b3378ef2f5855f2f97c628910368d6994b9ad72f782929e4dfc31bf
       Configurando parametros de RHEL
       Definiendo propiedades de instalacion
       Configurando clave de mariadb a Passw0rd
       mariadbencryption/
       mariadbencryption/server-key.pem
       mariadbencryption/keys.txt
       mariadbencryption/client-key.pem
       mariadbencryption/server-cert.pem
       mariadbencryption/startMariaDB_with_Vault.sh
       mariadbencryption/tablesToBeEncrypted
       mariadbencryption/ca-cert.pem
       mariadbencryption/truststore.jks
       mariadbencryption/client-cert.pem
       Enter password:
       
       **Enter mariadb password (Passw0rd)**

       Instalando Resilency Orchestrator
       220511 12:34:31 mysqld_safe Logging to '/var/lib/mysql/64359045dc63.err'.
       220511 12:34:31 mysqld_safe Starting mariadbd daemon with databases from /var/lib/mysql
       Preparing to install

       **Follow RO installation steps**
