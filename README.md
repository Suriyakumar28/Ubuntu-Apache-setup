# Ubuntu-Apache-setup
Full Apache setup file for ubuntu
Step 1 – Install JAVA
1) First, install OpenJDK 11. It is an implementation of the Java Platform. To update the packages index and install the OpenJDK 11 JDK package, run the following commands:

sudo apt update
sudo apt install openjdk-11-jdk -y
2) After installation, verify it by checking the version of JAVA.

java -version
3) You'll get an output like this:

openjdk version "11.0.7" 2020-04-14
OpenJDK Runtime Environment (build 11.0.7+10-post-Ubuntu-3ubuntu1)
OpenJDK 64-Bit Server VM (build 11.0.7+10-post-Ubuntu-3ubuntu1, mixed mode, sharing)
Step 2 – Create Tomcat User and Group
1) Create a new system user and group with a home directory of /opt/tomcat to run the Tomcat service. Enter the following command:

sudo useradd -m -U -d /opt/tomcat -s /bin/false tomcat
Step 3 – Download Tomcat
1) Check the Tomcat download page to see if a newer version is available.

2) After that, use wget to download the Tomcat zip file to the /tmp directory:

VERSION=9.0.97
wget https://downloads.apache.org/tomcat/tomcat-9/v${VERSION}/bin/apache-tomcat-${VERSION}.tar.gz
3) After downloading, extract the tar file to /opt/tomcat directory.

sudo tar -xf apache-tomcat-${VERSION}.tar.gz -C /opt/tomcat/
4) Tomcat is typically updated regularly for security and to introduce new features. For more control, create a symbolic link called “latest” that points to the Tomcat installation directory.

sudo ln -s /opt/tomcat/apache-tomcat-${VERSION} /opt/tomcat/latest
5) Unpack the latest version, then update the symlink to point to it. Additionally, change the directory ownership to the user and group tomcat using:

sudo chown -R tomcat: /opt/tomcat
6) After that, the shell scripts in Tomcat's bin directory need to be made executable.

sudo sh -c 'chmod +x /opt/tomcat/latest/bin/*.sh'
These scripts are useful for starting, stopping, and managing Tomcat instances.

Step 4 – Create the SystemD Unit File
1) Open the text editor and create a tomcat.service unit file in the /etc/systemd/system/ directory.

sudo nano /etc/systemd/system/tomcat.service
2) After that, paste the following configuration in /etc/systemd/system/tomcat.service:

/etc/systemd/system/tomcat.service
[Unit]
Description=Tomcat 9 servlet container
After=network.target
[Service]
Type=forking
User=tomcat
Group=tomcat
Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom -Djava.awt.headless=true"
Environment="CATALINA_BASE=/opt/tomcat/latest"
Environment="CATALINA_HOME=/opt/tomcat/latest"
Environment="CATALINA_PID=/opt/tomcat/latest/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
ExecStart=/opt/tomcat/latest/bin/startup.sh
ExecStop=/opt/tomcat/latest/bin/shutdown.sh
[Install]
WantedBy=multi-user.target
You can modify the JAVA_HOME variable, if the path to Java installation is different.
3) Then, save and close the file. Notify systemd that a new file exists.

sudo systemctl daemon-reload
4) Once done, enable the Tomcat service and start it.

sudo systemctl enable --now tomcat
5) You will need to check the service status.

sudo systemctl status tomcat
6) The result indicates that the Tomcat server is enabled and operational.

Output

tomcat.service - Tomcat 9 servlet container
Loaded: loaded (/etc/systemd/system/tomcat.service; enabled; vendor preset: enabled)
Active: active (running) since Tue 2020-05-12 17:58:37 UTC; 4s ago
Process: 5342 ExecStart=/opt/tomcat/latest/bin/startup.sh (code=exited, status=0/SUCCESS)
Main PID: 5362 (java)
7) Now you can start, stop, even restart Tomcat as any other systemd service. Use the following command:

sudo systemctl start tomcat
sudo systemctl stop tomcat
sudo systemctl restart tomcat
Step 5 – Configure the Firewall
If you wish to access Tomcat from outside the local network, you must open port 8080. To do so, use the following command:

sudo ufw allow 8080/tcp
Installing Guacamole
lsb_release -cd  ; getconf LONG_BIT ; hostname ; hostname -Is
apt install -y gcc g++ libcairo2-dev libjpeg-turbo8-dev libpng-dev libtool-bin libossp-uuid-dev libavcodec-dev libavutil-dev libswscale-dev freerdp2-dev libpango1.0-dev libssh2-1-dev libvncserver-dev libtelnet-dev libssl-dev libvorbis-dev libwebp-dev build-essential net-tools curl git software-properties-common
wget https://downloads.apache.org/guacamole/1.5.5/source/guacamole-server-1.5.5.tar.gz
tar xzf guacamole-server-1.5.5.tar.gz ; cd guacamole-server-1.5.5 ;
apt install build-essential
apt install libpng-dev libjpeg-dev libcairo2-dev -y
./configure --with-init-dir=/etc/init.d
make
make install
ldconfig
systemctl enable guacd ; systemctl start guacd ; systemctl status guacd
mkdir /etc/guacamole
wget https://downloads.apache.org/guacamole/1.5.5/binary/guacamole-1.5.5.war -O /etc/guacamole/guacamole.war
ln -s /etc/guacamole/guacamole.war /opt/tomcat/latest/webapps/
systemctl restart tomcat ; systemctl restart guacd
mkdir /etc/guacamole/{extensions,lib} ; echo "GUACAMOLE_HOME=/etc/guacamole" >> /etc/default/tomcat
nano /etc/guacamole/guacamole.properties
guacd-hostname: localhost
guacd-port:     4822
user-mapping:   /etc/guacamole/user-mapping.xml
auth-provider:  net.sourceforge.guacamole.net.basic.BasicFileAuthenticationProvider
ln -s /etc/guacamole /opt/tomcat/latest/.guacamole
echo -n Secret55! | openssl md5
nano /etc/guacamole/guacd.conf
[server]
#bind_host = localhost
bind_host = 127.0.0.1
bind_port = 4822
nano /etc/guacamole/user-mapping.xml
<user-mapping>
    <authorize
            username="guacadmin"
            password="3abb497c4bb92d58709013408367be3f"
            encoding="md5">

        <!-- First authorized Remote connection -->
        <connection name="Windows Server RDP">
            <protocol>rdp</protocol>
            <param name="hostname">172.00.00.151</param>
            <param name="port">3389</param>
            <param name="username">administrator</param>
            <param name="password">Secret55!</param>
            <param name="ignore-cert">true</param>
        </connection>

        <!-- Second authorized remote connection -->
        <connection name="Windows 10 RDP">
            <protocol>rdp</protocol>
            <param name="hostname">172.00.00.150</param>
            <param name="port">3389</param>
            <param name="username">Win-Desktop</param>
            <param name="password">Secret55!</param>
            <param name="ignore-cert">true</param>
        </connection>
    </authorize>
</user-mapping>
systemctl restart tomcat guacd
http://172.00.00.152:8080/guacamole/.
apt -y install openssh-server
nano /etc/ssh/sshd_config  [ PermitRootLogin no ]
systemctl restart ssh
Example-
User - guacadmin
Password - Secret55!
[Ubuntu [172 00 00 152] 12fc2319f4868046ab5dd60135dc81e5.html](https://github.com/user-attachments/files/22142735/Ubuntu.172.00.00.152.12fc2319f4868046ab5dd60135dc81e5.html)
