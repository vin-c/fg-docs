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
# Add EGI-Trustanchors repository
wget -O /etc/yum.repos.d/EGI-trustanchors.repo http://repository.egi.eu/sw/production/cas/1/current/repo-files/EGI-trustanchors.repo

# Add a nagios user (not done when installing package)
useradd -r -d /var/spool/nagios -s /sbin/nologin nagios

# Update and Install
yum update && yum install -y mod_ssl nagios nagios-plugins ca-policy-egi-core
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

Note : A git-repository is created/synced at `https://gitlab.in2p3.fr/vinc/nagboxfg-config` to track configuration changes

```
# Configure nagioadmin password
htpasswd -c /etc/nagios/passwd nagiosadmin

# Add GRID-FR users
echo "/O=GRID-FR/C=FR/O=CNRS/OU=IPHC/CN=Jerome Pansanel:xxj31ZMTZzkVA\n/O=GRID-FR/C=FR/O=CNRS/OU=IdG/CN=Vincent Gatignol-Jamon:xxj31ZMTZzkVA" >> /etc/nagios/passwd
```
Note : the password for x509 users must be "xxj31ZMTZzkVA"

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
## Bug double slash 
See https://bugzilla.redhat.com/show_bug.cgi?id=1036331
Update the file `/usr/share/nagios/html/config.inc.php`
```
--- config.inc.php.orig 2016-06-13 14:14:59.026000000 +0200
+++ config.inc.php      2016-06-13 14:13:25.012000000 +0200
@@ -4,7 +4,7 @@
 
 $cfg['cgi_config_file']='/etc/nagios/cgi.cfg';  // location of the CGI config file
 
-$cfg['cgi_base_url']='/nagios/cgi-bin/';
+$cfg['cgi_base_url']='/nagios/cgi-bin';
 
 
 // FILE LOCATION DEFAULTS
```
### PerfData

We will be using pnp4nagios with its "Bulk mode with NPCD" configuration (https://docs.pnp4nagios.org/pnp-0.6/modes#bulk_mode_with_npcd)
```
yum install pnp4nagios
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
+++     action_url                      /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=_HOST_' class='tips' rel='/pnp4nagios/index.php/popup?host=$HOSTNAME$&srv=_HOST_
        (...)
        register                        0
}
```
Add a graph-service-template
```
+++define service{
+++        name                            graph
+++        action_url                      /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=$SERVICEDESC$' class='tips' rel='/pnp4nagios/index.php/popup?host=$HOSTNAME$&srv=$SERVICEDESC$
+++        register                        0
+++        }
```
Add pnp4nagios commands
```
# pnp4nagios
#
# Bulk with NPCD mode
#
 define command {
       command_name    process-service-perfdata-file
       command_line    /bin/mv /var/log/pnp4nagios/service-perfdata /var/spool/pnp4nagios/service-perfdata.$TIMET$
 }

define command {
       command_name    process-host-perfdata-file
       command_line    /bin/mv /var/log/pnp4nagios/host-perfdata /var/spool/pnp4nagios/host-perfdata.$TIMET$
}
```
Create/Link the file from `/etc/nagios/pnp4nagios/common-header.ssi` to `/usr/share/nagios/html/ssi/common-header.ssi`
```
<script src="/pnp4nagios/media/js/jquery-min.js" type="text/javascript"></script>
<script src="/pnp4nagios/media/js/jquery.cluetip.js" type="text/javascript"></script>
<script type="text/javascript">
jQuery.noConflict();
jQuery(document).ready(function() {
  jQuery('a.tips').cluetip({ajaxCache: false, dropShadow: false,showTitle: false });
});
</script>
```
Update `/etc/nagios/nagios.cfg`
```
# pnp4nagios
process_performance_data=1

#
# Bulk / NPCD mode
# 

# *** the template definition differs from the one in the original nagios.cfg
#
service_perfdata_file=/var/log/pnp4nagios/service-perfdata
service_perfdata_file_template=DATATYPE::SERVICEPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tSERVICEDESC::$SERVICEDESC$\tSERVICEPERFDATA::$SERVICEPERFDATA$\tSERVICECHECKCOMMAND::$SERVICECHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tSERVICESTATE::$SERVICESTATE$\tSERVICESTATETYPE::$SERVICESTATETYPE$
service_perfdata_file_mode=a
service_perfdata_file_processing_interval=15
service_perfdata_file_processing_command=process-service-perfdata-file

# *** the template definition differs from the one in the original nagios.cfg
#
host_perfdata_file=/var/log/pnp4nagios/host-perfdata
host_perfdata_file_template=DATATYPE::HOSTPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tHOSTPERFDATA::$HOSTPERFDATA$\tHOSTCHECKCOMMAND::$HOSTCHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$
host_perfdata_file_mode=a
host_perfdata_file_processing_interval=15
host_perfdata_file_processing_command=process-host-perfdata-file
```
Activate Bulk/NPCD service
```
chkconfig --add npcd
chkconfig npcd on
systemctl start npcd
```
Create/Link the templates from `/etc/nagios/pnp4nagios/check_openstack.php` 
```
# ls -l /usr/share/nagios/html/pnp4nagios/templates/    
total 0
lrwxrwxrwx 1 root root   42  1 juin  16:42 check_isolation.php -> /etc/nagios/pnp4nagios/check_openstack.php
lrwxrwxrwx 1 root root   42  1 juin  16:42 check_openstack.php -> /etc/nagios/pnp4nagios/check_openstack.php
lrwxrwxrwx 1 root root   42  1 juin  16:52 check_openstack_regex.php -> /etc/nagios/pnp4nagios/check_openstack.php
```
```
<?php
##
$ds_name[1] = "Execution Time";
$opt[1] = "--vertical-label \"Time ($UNIT[1])\" --title \"Execution Time for $hostname / $servicedesc\" ";
#
$def[1] =  "DEF:exec_time=$RRDFILE[1]:$DS[1]:AVERAGE " ;
$def[1] .= "AREA:exec_time#0000FF:\"Execution Time \" " ;
$def[1] .= "LINE1:exec_time#000000 " ;
$def[1] .= rrd::gprint("exec_time", array("LAST", "MAX", "AVERAGE"), "%3.0lf $UNIT[1]") ;
##
$ds_name[2] = "Number of tests";
$opt[2] = "--vertical-label \"Count\" --title \"Number of tests for $hostname / $servicedesc\" ";
#
$def[2] =  "DEF:nb_tests=$RRDFILE[1]:$DS[2]:AVERAGE " ;
$def[2] .= "AREA:nb_tests#AABBCC:\"Total   \" " ;
$def[2] .= "LINE1:nb_tests#AABBCC " ;
$def[2] .= rrd::gprint("nb_tests", array("LAST", "MAX", "AVERAGE"), "%3.0lf $UNIT[2]") ;
#
$def[2] .= "DEF:nb_tests_ok=$RRDFILE[1]:$DS[3]:AVERAGE " ;
$def[2] .= "AREA:nb_tests_ok#00FF00:\"OK      \" " ;
$def[2] .= "LINE1:nb_tests_ok#00FF00 " ;
$def[2] .= rrd::gprint("nb_tests_ok", array("LAST", "MAX", "AVERAGE"), "%3.0lf $UNIT[2]") ;
#
$def[2] .= "DEF:nb_tests_ko=$RRDFILE[1]:$DS[4]:AVERAGE " ;
$def[2] .= "AREA:nb_tests_ko#FF0000:\"NOK     \" " ;
$def[2] .= "LINE1:nb_tests_ko#FF0000 " ;
$def[2] .= rrd::gprint("nb_tests_ko", array("LAST", "MAX", "AVERAGE"), "%3.0lf $UNIT[2]") ;
#
$def[2] .= "DEF:nb_skipped=$RRDFILE[1]:$DS[5]:AVERAGE " ;
$def[2] .= "AREA:nb_skipped#0000FF:\"Skipped \" " ;
$def[2] .= "LINE1:nb_skipped#0000FF " ;
$def[2] .= rrd::gprint("nb_skipped", array("LAST", "MAX", "AVERAGE"), "%3.0lf $UNIT[2]") ;
?>
```
Create/Link the configuration from `/etc/nagios/pnp4nagios/check_openstack.cfg`
```
# ls -l /etc/pnp4nagios/check_commands
lrwxrwxrwx 1 root root   42 13 juin  10:49 check_isolation.cfg -> /etc/nagios/pnp4nagios/check_openstack.cfg
lrwxrwxrwx 1 root root   42 13 juin  10:49 check_openstack.cfg -> /etc/nagios/pnp4nagios/check_openstack.cfg
lrwxrwxrwx 1 root root   42 13 juin  10:53 check_openstack_regex.cfg -> /etc/nagios/pnp4nagios/check_openstack.cfg
```
```
# RRD Heartbeat in seconds
# This Option is only used while creating new RRD Databases
# Existing RRDs can be changed by "rrdtool tune"
# More on http://oss.oetiker.ch/rrdtool/doc/rrdtune.en.html
#
# This value defaults to 8640
RRD_HEARTBEAT = 129600
```
### (Re)start services
```
systemctl restart httpd nagios npcd
```
