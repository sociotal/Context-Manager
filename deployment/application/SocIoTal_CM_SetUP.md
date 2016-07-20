#Installation guide for SocIoTal Context Manager

This guide is intended to provide an easy way to have your own working instance of the SocIoTal Context Manager (CM), ready to be linked to other SocIoTal components, such the SocIoTal Communities Manager (ComM) and the SocIoTal Identity Manager (IdM). The master piece of the SocIoTal CM is distributed as a WAR file, ready to be deployed on top of a Web Application Server. Distributes version has been tested on Tomcat (Tomcat7) and JBoss (JBoss AS 7.1) but other Web App servers supporting WAR files could be used. This guide will make use of Tomcat (Tomcat version 7).

SocIoTal CM requires a Context Broker (CB) supporting OMA NGSI to manage context sources and context information. Distributed version of CM has been built on top of the FIWARE Orion CB \[[*http://catalogue.fiware.org/enablers/publishsubscribe-context-broker-orion-context-broker*](http://catalogue.fiware.org/enablers/publishsubscribe-context-broker-orion-context-broker)\] (versions from 0.9.0 to 1.2.1 has been tested). The guide includes also the required steps to install and link this component. Other CBs could be tested.

Current installation guide has been run on an Ubuntu Server 14.04 LTS 64 bits (Trusty Tahr). This tutorial has been tested last time on 18/07/2016. Since then, some links might be changed.

Operating System: UBUNTU 14.04 LTE SERVER 64 bits + Base components
===================================================================

This section prepares a basic (and clean) Ubuntu Server 14.04 64 bits existing installation to get the required software to run SocIoTal CM. In particular, this section will install JAVA OpenSDK 1.7 and Tomcat 7. The firs action is to update the system, in order to get access to latest versions of components:

Install JAVA
------------
```
\# sudo apt-get update
```
In a terminal, use the following command to install OpenJDK Java Development Kit:

\# sudo apt-get install default-jdk

If everything went fine, checking the installed version should report:

\# java -version

java version "1.7.0\_101"

OpenJDK Runtime Environment (IcedTea 2.6.6) (7u101-2.6.6-0ubuntu0.14.04.1)

OpenJDK 64-Bit Server VM (build 24.95-b01, mixed mode)

OpenJDK 1.7 version is enough to support SocIoTal CM. Later versions of JDK should be also supported, but have not been tested.

Install Apache Tomcat (v7)
--------------------------

This tutorial, and current version of the SocIoTal Cloud Instance, runs with Tomcat version 7. No further tests have been done with later versions of this Web Application Server. SocIoTal CM has been also tested on JBoss AS 7.1.

To install Tomcat7 on our Ubuntu:

\# sudo apt-get install tomcat7

To control (upload wars) remotely:

\# sudo apt install tomcat7-docs tomcat7-admin

Access to the manager application is protected by default: you need to define a user with the role "manager-gui" in **/etc/tomcat7/tomcat-users.xml** before you can access it.

\# sudo vi /etc/tomcat7/tomcat-users.xml

Then write this and save the file

&lt;role rolename="manager-gui"/&gt;

&lt;role rolename="admin-gui"/&gt;

&lt;role rolename="admin"/&gt;

&lt;user username="username" password="password" roles="admin,admin-gui,manager-gui"/&gt;

Finally, restart tomcat service:

\#sudo service tomcat7 restart

### Configure Tomcat7 to support https connections

If you want to configure your Tomcat7 to support https connections (and so the SocIoTal CM), you can find detailed documentation through this link \[[*https://tomcat.apache.org/tomcat-7.0-doc/ssl-howto.html*](https://tomcat.apache.org/tomcat-7.0-doc/ssl-howto.html)\]. A sort direct howto is shown below:

First, get a valid PKCS12 certificate. If you still don’t get one, you can create an auto-signed certificate, useful for testing or trustworthy deployments using OpenSSL \[[*https://help.ubuntu.com/12.04/serverguide/certificates-and-security.html\#creating-a-self-signed-certificate*](https://help.ubuntu.com/12.04/serverguide/certificates-and-security.html%23creating-a-self-signed-certificate)\]

Once you have your .p12 file plus you password, you can configure Tomcat to enable HTTPS using your certificate. For this example, we’re using 8443 port, but other could be used. To do so, edit the tomcat 7 server.xml file:

\#sudo vi /etc/tomcat7/server.xml

Then, write this and save the file

&lt;Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"

maxThreads="200" scheme="https" secure="true"

keystoreType="PKCS12"

keystoreFile="/etc/tomcat7/yourcertfile.p12" keystorePass="yourpasswd"

clientAuth="false" sslProtocol="TLS"/&gt;

Finally, restart tomcat service:

\#sudo service tomcat7 restart

Install SocIoTal Context Broker
===============================

For this instance of SocIoTal CM we’re using FIWARE Orion Context Broker \[[*http://catalogue.fiware.org/enablers/publishsubscribe-context-broker-orion-context-broker*](http://catalogue.fiware.org/enablers/publishsubscribe-context-broker-orion-context-broker)\] as its NGSI Context Broker. In turn, this component requires a MongoDB instance. For Orion Context Broker 1.2.1, MongoDB 3.2 version has been installed:

Install MongoDB
---------------

**MongoDB** is required running either in the same system where Orion Context Broker is going to be installed or in a different host accessible through the network. Further details about installing mongoDB within Ubuntu can be found through this link \[*https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu*/\]. Below are the steps for a quick setup of mongo 3.2 version in our Ubuntu machine.

\#sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927

\#echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

\#sudo apt-get update

\#sudo apt-get install -y mongodb-org

Last stable mongoDB (3.2.8).

\# mongod --version

db version v3.2.8

git version: ed70e33130c977bda0024c125b56d159573dbaf0

OpenSSL version: OpenSSL 1.0.1f 6 Jan 2014

allocator: tcmalloc

modules: none

build environment:

distmod: ubuntu1404

distarch: x86\_64

target\_arch: x86\_64

If everything went fine, mongodb should be running on port 27017

Port and other features of this mongodb installation can be changed by editing **/etc/mongod.conf**.

Install FIWARE Orion Context Broker
-----------------------------------

There exists an advanced tutorial about how to install Orion Context Broker provided by FIWARE and through its enablers catalogue \[[*https://fiware-orion.readthedocs.io/en/develop/admin/install/index.html*](https://fiware-orion.readthedocs.io/en/develop/admin/install/index.html)\]. This quick tutorial provides just the set of commands required to directly download, install and configure the Orion CB and its dependences, for version 0.9.0 to 1.2.1. Further versions have been not tested.

As the Orion CB is distributed as rpm debian package, we will use “alien” software to easily install this packages and all the corresponding dependences.

\# sudo apt-get install alien

Download and install the list of dependences. More details about why are these required can be found in FIWARE’s tutorial.

**Packages:**

\#wget ftp://rpmfind.net/linux/centos/6.8/os/x86\_64/Packages/boost-thread-1.41.0-28.el6.x86\_64.rpm

\#wget ftp://fr2.rpmfind.net/linux/centos/6.8/os/x86\_64/Packages/boost-system-1.41.0-28.el6.x86\_64.rpm

\#wget [*ftp://fr2.rpmfind.net/linux/centos/6.8/os/x86\_64/Packages/boost-filesystem-1.41.0-28.el6.x86\_64.rpm*](ftp://fr2.rpmfind.net/linux/centos/6.8/os/x86_64/Packages/boost-filesystem-1.41.0-28.el6.x86_64.rpm)

\#wget [*ftp://195.220.108.108/linux/centos/6.8/os/x86\_64/Packages/boost-regex-1.41.0-28.el6.x86\_64.rpm*](ftp://195.220.108.108/linux/centos/6.8/os/x86_64/Packages/boost-regex-1.41.0-28.el6.x86_64.rpm)

\#wget ftp://195.220.108.108/linux/Mandriva/official/2010.0/x86\_64/media/main/release/lib64icu42-4.2.1-1mdv2010.0.x86\_64.rpm

\#sudo alien -i boost-thread-1.41.0-28.el6.x86\_64.rpm

\#sudo alien -i boost-system-1.41.0-28.el6.x86\_64.rpm

\#sudo alien -i boost-filesystem-1.41.0-28.el6.x86\_64.rpm

\#sudo alien -i boost-regex-1.41.0-28.el6.x86\_64.rpm

\#sudo alien -i lib64icu42-4.2.1-1mdv2010.0.x86\_64.rpm

**Libraries:**

\# sudo apt-get install libcurl4-openssl-dev

Not required in CB 1.2.1 (but might be needed in earlier versions or other linux distributions)

\# sudo apt-get install libboost-all-dev

In our case, the libraries were installed in “/usr/lib64” folder. To correct this (and CB is able to locate them)

create a .conf file within “/etc/ld.so.conf.d/”

\#sudo vi /etc/ld.so.conf.d/libboost.conf

add a line (“/usr/lib64”) within and save it

quit and reload your configuration:

\#sudo ldconfig

Download and Install **Orion CB package**

\#wget [*http://repositories.testbed.fiware.org/repo/rpm/6/x86\_64/contextBroker-1.2.1-1.x86\_64.rpm*](http://repositories.testbed.fiware.org/repo/rpm/6/x86_64/contextBroker-1.2.1-1.x86_64.rpm)

\#sudo alien -c -i contextBroker-1.2.1-1.x86\_64.rpm

Now, you should be able to execute Orion CB directly from command line (this example is configured to listen in port 4537, only attached to local host and uses only IPv4. Further commands/configurations can be obtained by using “contextBroker --help”):

\# sudo contextBroker -port 4537 -localIp 127.0.0.1 -ipv4

To configure Orion CB as a service, create file “contextBroker.conf”:

\#sudo vi /etc/init/contextBroker.conf

Write and save in this file:

\# FIWARE ORION CONTEXT BROKER INSTANCE - contextBroker job file

description "Service conf file for the Context Broker service (FIWARE ORION CB)"

author "Ignacio Elicegui Maestro &lt;IEMaestro@tlmat.unican.es&gt;"

start on (local-filesystems and net-device-up IFACE!=lo)

stop on runlevel \[016\]

\# Automatically restart process if crashed

respawn

setuid root

script

\#Change to folder where -initially- CB file is installed

cd /usr/bin

\#run contextBroker

\#set port on 4357

\#set bind IP address to local

contextBroker -port 4357 -localIp 127.0.0.1 -ipv4

end script

Next time you reboot the system, Orion CB should be up and listening how you configured it.

Install SocIoTal Context Manager
================================

SocIoTal Context Manager is presented as a WAR file to be deployed in a Web Application Server. Currently, JBoss AS 7.1 and Tomcat7 have been tested (both successfully) but, initially would be possible to deploy them in any Web AS supporting WARs. The WAR package includes the .jar objects to run all the CM methods documented plus all the required external libraries, in order to ensure its execution in most of the platforms where it could be deployed. The corresponding WAR file can be downloaded from SocIoTal GitHub \[Link\].

Before deploy it in the Tomcat, its configuration file should be created in /server/SocIoTal\_CM:

\#sudo mkdir /server

\#sudo mkdir /server/SocIoTal\_CM

\#sudo vi /server/SocIoTal\_CM/Configfile

Then, write down the configuration you require for your deployment:

\#SocIoTal Context Manager Configuration File

\#\#\#\#\#\#Context Broker Params

\#CB\_IP specifies where the OMA NGSI Context Broker is listening (IP)

\#CP\_PORT specifies where the OMA NGSI Context Broker is listening (PORT)

\# for a local deployment (where CM and CB are installed in the same machine)

\# CB\_IP = 127.0.0.1

\# CB\_PORT = 1026 (if no port has been pointed when CB has started)

CB\_IP = 127.0.0.1

CB\_PORT = 1026

\#\#\#\#\#\#Communities Manager Params

\#Specifies the parameters required to link the CM with the SocIoTal Communities Manager

\#This enables the support for communities

\#Admin Password of the SocIoTal IdM Instance the Community Manager is linked (see Configuration file of the Communities Manager)

Comm\_Admin\_Passwd = KeyRock\_Admin\_Pass

\#IP and Port where the selected Comm Manager is listening (matches the Web App Server

\#configuration)

Comm\_IP = 127.0.0.1

Comm\_Port = 8080

\#URI where the token services of the Communities Manager are listening

Comm\_Token\_Path = /SocIoTal\_COMM\_V1/TOKEN/

\#\#\#\#\#\#\#Capability Manager Params

\#Specifes the configuration parameters to activate Capability-Token support

\#Supporting of Capabilty-Tokens:

\#FALSE: Cap-Token is ignored, either it is present in headers or not

\#TRUE: Cap-Token is REQUIRED for all transactions with SocIoTal CM

\#HALF: for developing/testing purposes, the CM will evaluate the Capability-Token ONLY if \#this header is present in the HTTP/S Request. If not, Cap-Token evaluation is bypassed.

Act\_Cap\_Token\_Eval = HALF

\#Server certificate to evaluate Cap-Token

Server\_cert = /server/certs/server.cer

\#Certification Authority Certificate to evaluate Cap-Tohen

CA\_cert = /server/certs/ca.cer

\#Parameters to link the FIWARE KeyRock instance enabled by SocIoTal IdM

\#KeyRock ADMIN PASSWORD

KeyRock\_ADMIN = KeyRock\_Admin\_Pass

\#KeyRock ADMIN IP + PORT

KeyRock\_IP = 193.144.205.93

KeyRock\_Port = 443

If you want to integrate with SocIoTal Capability Manager, you should get and specify the “server.cer” and the “ca.cer” pointed in the configuration file. These testing files are available in the GitHub server.

Once this is set, you can access Tomcat7 web and upload the SocIoTal CM WAR file and start it. This link shows you how to do it using Tomcat7 \[[*https://tomcat.apache.org/tomcat-7.0-doc/manager-howto.html\#Deploy\_A\_New\_Application\_Archive\_(WAR)\_Remotely*](https://tomcat.apache.org/tomcat-7.0-doc/manager-howto.html%23Deploy_A_New_Application_Archive_(WAR)_Remotely)\].

Now, you can refer to WiKi SocIoTal Context Manager and start using the methods there described.
