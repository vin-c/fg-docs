## Installation guide of ccnagboxfg

OS : CentOS7.2 x86_64
 
CPU : 4 cores

RAM : 4 Gib

HDD : 40 Gib

```
  --- Physical volume ---
  PV Name               /dev/vda2
  VG Name               rootvg
  PV Size               39,80 GiB / not usable 2,00 MiB
  Allocatable           yes 
  PE Size               4,00 MiB
  Total PE              10189
  Free PE               8439
  Allocated PE          1750
  PV UUID               u0oCBJ-PAcE-HBjk-FYiK-ccQX-DU1z-dD02NM

  --- Volume group ---
  VG Name               rootvg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  8
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                4
  Open LV               4
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               39,80 GiB
  PE Size               4,00 MiB
  Total PE              10189
  Alloc PE / Size       1750 / 6,84 GiB
  Free  PE / Size       8439 / 32,96 GiB
  VG UUID               mySHNr-idY6-xrLI-esJE-cfGU-MzrY-43odHu

  --- Logical volume ---
  LV Path                /dev/rootvg/tmp
  LV Name                tmp
  VG Name                rootvg
  LV UUID                lNi8Bx-8o8S-uDJR-aqrQ-afvw-r61X-Zlyjqq
  LV Write Access        read/write
  LV Creation host, time cctest30.in2p3.fr, 2016-03-29 06:08:35 +0200
  LV Status              available
  # open                 1
  LV Size                500,00 MiB
  Current LE             125
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2

  --- Logical volume ---
  LV Path                /dev/rootvg/usr
  LV Name                usr
  VG Name                rootvg
  LV UUID                tEiJNW-UTxe-IC1i-GWI5-Ebax-l4vx-EckcZX
  LV Write Access        read/write
  LV Creation host, time cctest30.in2p3.fr, 2016-03-29 06:08:38 +0200
  LV Status              available
  # open                 1
  LV Size                3,42 GiB
  Current LE             875
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/rootvg/var
  LV Name                var
  VG Name                rootvg
  LV UUID                2XUdYP-u9aH-hjvH-BP5d-95lY-nno7-BBnNOy
  LV Write Access        read/write
  LV Creation host, time cctest30.in2p3.fr, 2016-03-29 06:08:45 +0200
  LV Status              available
  # open                 1
  LV Size                1,95 GiB
  Current LE             500
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
   
  --- Logical volume ---
  LV Path                /dev/rootvg/root
  LV Name                root
  VG Name                rootvg
  LV UUID                XBrdeT-lkGw-qefX-KqXQ-Ssmi-JzsV-lYgfGg
  LV Write Access        read/write
  LV Creation host, time cctest30.in2p3.fr, 2016-03-29 06:08:51 +0200
  LV Status              available
  # open                 1
  LV Size                1000,00 MiB
  Current LE             250
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
```

### Basic Security

```
systemctl enable firewalld
systemctl start firewalld
firewall-cmd --permanent --add-service ssh
firewall-cmd --permanent --add-service https
firewall-cmd --permanent --add-service http
systemctl restart firewalld
```

### Packages installation

```
# Get iRODS binaries
mkdir /root/irods-rpm
cd /root/irods-rpm
wget http://www.grand-est.fr/yum/irods/sl7/x86_64/irods-client-gsi-3.3-3.el7.centos.x86_64.rpm
wget http://www.grand-est.fr/yum/irods/sl7/x86_64/irods-3.3-3.el7.centos.x86_64.rpm

# Add EGI-Trustanchors repository
wget -O /etc/yum.repos.d/EGI-trustanchors.repo http://repository.egi.eu/sw/production/cas/1/current/repo-files/EGI-trustanchors.repo

# Add a nagios user (not done when installing package)
useradd -r nagios

# Update and Install
yum update && yum install -y mod_ssl nagios nagios-plugins ca-policy-egi-core
yum localinstall -y irods-3.3-3.el7.centos.x86_64.rpm irods-client-gsi-3.3-3.el7.centos.x86_64.rpm
```

### Certificate request

Follow the instructions here : https://igc.services.cnrs.fr/servercert/?CA=GRID2-FR

Get the files `hostcert.key` and `hostcert.pem` and put them in `/etc/grid-security/`.

Set the right ACL on the files : `chmod 0400 *.key && chmod 0644 *.pem`

You can make symlinks from the original files...

```
# ls -lah /etc/grid-security/
total 80K
drwxr-xr-x    3 root root 4,0K 19 mai   10:52 ./
drwxr-xr-x. 106 root root  12K 19 mai   10:06 ../
-r--------    1 root root 1,7K 19 mai   10:51 ccnagboxfg.in2p3.fr.key
-rw-r--r--    1 root root 5,3K 19 mai   10:51 ccnagboxfg.in2p3.fr.pem
drwxr-xr-x    2 root root  44K 19 mai   10:04 certificates/
-rw-r--r--    1 root root 1,8K 28 janv. 00:08 gsi.conf
lrwxrwxrwx    1 root root   23 19 mai   10:52 hostcert.key -> ccnagboxfg.in2p3.fr.key
lrwxrwxrwx    1 root root   23 19 mai   10:52 hostcert.pem -> ccnagboxfg.in2p3.fr.pem

# openssl x509 -in hostcert.pem -noout -enddate -subject
notAfter=May 19 08:32:09 2017 GMT
subject= /O=GRID-FR/C=FR/O=CNRS/OU=IdG/CN=ccnagboxfg.in2p3.fr

```

### Nagios

```
# Configure nagioadmin password
htpasswd -c /etc/nagios/passwd nagiosadmin

# Add GRID-FR users
echo "/O=GRID-FR/C=FR/O=CNRS/OU=IPHC/CN=Jerome Pansanel:xxj31ZMTZzkVA\n/O=GRID-FR/C=FR/O=CNRS/OU=IdG/CN=Vincent Gatignol-Jamon:xxj31ZMTZzkVA" >> /etc/nagios/passwd
```

Some config tweaks `/etc/nagios/nagios.cfg`
```
# comment contacts or other objects that will be defined elswhere
#cfg_file=/etc/nagios/objects/contacts.cfg

# Update timeout of service checks as the testing scenario last ~90 to 120s
service_check_timeout=180
```

Configure HTTPS

`/etc/httpd/conf.d/ssl.conf`

```
SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5
SSLCertificateFile /etc/grid-security/ccnagboxfg.in2p3.fr.pem
SSLCertificateKeyFile /etc/grid-security/ccnagboxfg.in2p3.fr.key
SSLCACertificatePath    /etc/grid-security/certificates
SSLCARevocationPath     /etc/grid-security/certificates
SSLVerifyClient require
SSLVerifyDepth  5
SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire
```

Redirect rules

`/etc/httpd/conf.d/nagios.conf`

```
ServerName ccnagboxfg.in2p3.fr
RedirectMatch ^/$ /nagios/
<VirtualHost *:80>
   ServerName ccnagboxfg.in2p3.fr
   Redirect "/" "https://ccnagboxfg.in2p3.fr/"
</VirtualHost>

```

### PerfData
```
yum install perl-CGI rrdtool-perl perl-GD
wget -O nagios-graph-1.5.2.tgz "https://sourceforge.net/projects/nagiosgraph/files/nagiosgraph/1.5.2/nagiosgraph-1.5.2.tar.gz/download"
tar -xvzf nagios-graph-1.5.2.tgz && cd nagiosgraph-1.5.2
./install.pl
```

-> Default answers are ok

-> Let the script update nagios and httpd configuration files

`/etc/httpd/conf.d/nagiosgraph.conf`
Add this snippet to the two <Directory> statements to enable x509 auth
```
   AuthName "Nagios Access"
   AuthType Basic
   AuthUserFile /etc/nagios/passwd
   <IfModule mod_authz_core.c>
      # Apache 2.4
      <RequireAll>
         Require all granted
         # Require local
         Require valid-user
      </RequireAll>
   </IfModule>
   <IfModule !mod_authz_core.c>
      # Apache 2.2
      Order allow,deny
      Allow from all
      #  Order deny,allow
      #  Deny from all
      #  Allow from 127.0.0.1
      Require valid-user
   </IfModule>
```

Create the file `/usr/share/nagios/html/ssi/common-header.ssi`
```
<script type="text/javascript" src="/nagiosgraph/nagiosgraph.js"></script>
```

Update the nagios templates to include perfdata

Add `use graph` to the generic-service template
```
define service{
        name                            generic-service
+++     use                             graph
        (...)
        register                        0
}
```
Add `action_url` to the host-template
```
define host{
+++     action_url                      /nagiosgraph/cgi-bin/showhost.cgi?host=$HOSTNAME$' onMouseOver='showGraphPopup(this)' onMouseOut='hideGraphPopup()' rel='/nagiosgraph/cgi-bin/showgraph.cgi?host=$HOSTNAME$
        (...)
        register                        0
}
```
Add a graph-service-template
```
+++define service{
+++        name                            graph
+++        action_url                      /nagiosgraph/cgi-bin/show.cgi?host=$HOSTNAME$&service=$SERVICEDESC$' onMouseOver='showGraphPopup(this)' onMouseOut='hideGraphPopup()' rel='/nagiosgraph/cgi-bin/showgraph.cgi?host=$HOSTNAME$&service=$SERVICEDESC$
+++        register                        0
+++        }
```

