#    	<u>HOW TO SETUP TOMCAT ON CENTOS 7</u>



###  **What is tomcat ?**
It is an open-source Java servlet container that implements many Java Enterprise Specs such as the Websites API, Java-Server Pages and last but not least, the Java Servlet.

For tomcat setup we need java installed on our server.



### **1. Install java JRE** 

#### **1.1 To Check latest java version**
```
# sudo yum search openjdk
```
#### **1.2 To install latest java version 11**
```
# sudo yum install java-1.8.0-openjdk*
```
#### **1.3 After installation check java version**

```
# java -version
```

### **2. Download tomcat for centos 7**

#### **2.1 Check tomcat latest version on given site**

https://downloads.apache.org/tomcat/tomcat-9/

Go on above website check latest version of tomcat 9 and find core tar.gz right click on it and copy link address like follows and download it by commands
```
# cd opt
# wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.64/bin/apache-tomcat-9.0.64.tar.gz
```
#### **2.2 Extract tar file**

```
# tar -xf apache-tomcat-9.0.64.tar.gz    
```

#### **2.3 Rename file name apache-tomcat-9.0.27 to tomcat**
```
# mv apache-tomcat-9.0.27 tomcat
```

### 3. Change webapps folder location 

By default `webaaps` folder is in `/opt/tomcat/` . We move it to `/var/www/tomcat/`

```
# cd /var/www/
# mkdir tomcat
# mv webapps /opt/tomcat/ /var/www/tomcat/
```

### 4. Softlink to logs folder 

By default `logs` folder is in `/opt/tomcat/`.  We softlink it to `/var/logs/`.

```
# mkdir /var/log/tomcat
# ln -s /var/log/tomcat/ /opt/tomcat/logs
```

### 5. SSL Certification using openSSL

Following command will generate `key` and `crt` this 2 files
```
# mkdir /opt/tomcat/keys
# cd /opt/tomcat/keys
# openssl req -newkey 2048 -nodes -keyout tomcat.key -x509 -days 365 -out tomcat.crt
```
We have to export those 2 file in `p12` file using folowing command
```
openssl pkcs12 -inkey tomcat.key -in tomcat.crt -export -out tomcat.p12
```
Now add details of certification  in `server.xml` 

### 6. Changes in `server.xml` 
#### **6.1 Change port of tomcat**

By default port is `8080`. We neet to do changes in `server.xml` file to change tomcat port. 
```
# cd /opt/tomcat/conf/
# vi server.xml
```
We have to edit following section. This is orignal code in `server.xml` file with port  `8080` and redirecting port `8443`
```
<Connector port="8080" protocol="HTTP/1.1"
	              connectionTimeout="20000"
	              redirectPort="8443" />
```
This is edited code in `server.xml` file with change port `443` and redirecting port `8787`

```
<Connector port="443" protocol="HTTP/1.1"
                  connectionTimeout="20000"
                  redirectPort="8787" />
```
After changing ports start tomcat agian and checkwebapps process and port using `ps` and `netstat` commands.

#### **6.2 Change webapps folder path**

By default `webapps` folder path is `/opt/tomcat` but we move it to `/var/www/tomcat`

We have to edit following section. This is orignal code in `server.xml` file with appBase  `webapps` 
```
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
```
his is edited code in `server.xml` file with change appBase `/var/www/tomcat`
```
<Host name="localhost"  appBase="/var/www/tomcat/webapps/"
            unpackWARs="true" autoDeploy="true">
```
####    **6.3 Add Certification details**

Now we have 3 files `tomcat.key , tomcat.crt , tomcat.p12` we need to add their enrty in `server.xml`

We have to edit following section. This is orignal code in `server.xml` file
```
<Connector port="443" protocol="HTTP/1.1"
                  connectionTimeout="20000"
                  redirectPort="8787" />
```
This is edited code in `server.xml` file
```
<Connector
           protocol="org.apache.coyote.http11.Http11AprProtocol"
           port="443" maxThreads="200" redirectPort="8787"
           scheme="https" secure="true" SSLEnabled="true"
           SSLCertificateFile="/opt/tomcat/conf/keys/server.crt"
           SSLCertificateKeyFile="/opt/tomcat/conf/keys/server.key"
           SSLVerifyClient="optional" SSLProtocol="TLSv1+TLSv1.1+TLSv1.2"
           certificateKeystoreFile="/opt/tomcat/conf/keys/tomcat1.p12"
      	   certificateKeystorePassword="pass@123"	/>
```

### 7. Create tomcat user 
```
# useradd -m -U -d /opt/tomcat -s /bin/false tomcat
```

### 8. Change ownership root to tomcat
```
# cd /opt/
# chown -R tomcat:tomcat tomcat
# chown -R tomcat:tomcat /var/www/tomcat
# chown -R tomcat:tomcat /var/log/
```

### 9. Check HTTPS is woring or not
```
# curl -k -v https://localhost/
```

