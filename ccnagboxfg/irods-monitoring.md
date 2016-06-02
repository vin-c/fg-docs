# iRods monitoring installation

## Binaries
```
mkdir /root/irods-rpm
cd /root/irods-rpm
wget http://www.grand-est.fr/yum/irods/sl7/x86_64/irods-client-gsi-3.3-3.el7.centos.x86_64.rpm
wget http://www.grand-est.fr/yum/irods/sl7/x86_64/irods-3.3-3.el7.centos.x86_64.rpm
yum localinstall -y irods-3.3-3.el7.centos.x86_64.rpm irods-client-gsi-3.3-3.el7.centos.x86_64.rpm
```

Get the scripts from https://github.com/FranceGrilles/monitoring-irods

`git clone https://github.com/FranceGrilles/monitoring-irods.git /usr/lib/monitoring-irods`

## Nagios configuration

Update nagios configuration with the following.

Note : values like `<exemple>` must be modified according to your environment...

```
# iRods Commands
define command{
        command_name    check_irods_icat_connections
        command_line    $USER3$/check_irods_icat_connections.sh
}

define command{
        command_name    check_irods_resource_all
        command_line    $USER3$/check_irods_resource_all.sh -H $HOSTNAME$ -R $ARG1$
}

define command{
        command_name    check_irods_passive
        command_line    $USER1$/check_dummy 3 "$ARG1$"
}

# iRods ServiceGroups
define servicegroup{
        servicegroup_name       SERVICE_IRODS3_ICAT     ; The name of the hostgroup
        alias                   SERVICE_IRODS3_ICAT     ; Long name of the group
        }

define servicegroup{
        servicegroup_name       SERVICE_IRODS3_RESOURCE ; The name of the hostgroup
        alias                   SERVICE_IRODS3_RESOURCE ; Long name of the group
        }

# iRods HostGroups
define hostgroup{
        hostgroup_name  fg-irods-resources              ; The name of the hostgroup
        alias           FG-iRODS Resources              ; Long name of the group
        members         <srv.resource1.in2p3.fr,srv.resource2.in2p3.fr,srv.resource3.in2p3.fr>
        }

define hostgroup{
        hostgroup_name  fg-irods-icat                   ; The name of the hostgroup
        alias           FG-iRODS iCAT                   ; Long name of the group
        members         <srv.catalog.in2p3.fr>
        }

# iRods Services
define service{
        use                     generic-service
        hostgroup_name          fg-irods-resources
        service_description     org.irods.irods3.Resource-Iput
        servicegroups           SERVICE_IRODS3_RESOURCE
        check_command           check_irods_passive!This metric is part of the iRODS-All bundle and cannot be executed indepentently
        passive_checks_enabled  1
        active_checks_enabled   0
        }

# Create a service to check iget on each resource
define service{
        use                     generic-service
        hostgroup_name          fg-irods-resources
        service_description     org.irods.irods3.Resource-Iget
        servicegroups           SERVICE_IRODS3_RESOURCE
        check_command           check_irods_passive!This metric is part of the iRODS-All bundle and cannot be executed indepentently
        passive_checks_enabled  1
        active_checks_enabled   0
        }

# Create a service to check irm on each resource
define service{
        use                     generic-service
        hostgroup_name          fg-irods-resources
        service_description     org.irods.irods3.Resource-Irm
        servicegroups           SERVICE_IRODS3_RESOURCE
        check_command           check_irods_passive!This metric is part of the iRODS-All bundle and cannot be executed indepentently
        passive_checks_enabled  1
        active_checks_enabled   0
        }
```

Update `/etc/nagios/private/resource.cfg` with
```
$USER3$=/usr/lib/monitoring-irods/plugins
```

Create a Catalog host
```
define host{
        use             linux-server                    ; Inherit default values from a template
        host_name       <srv.catalog.in2p3.fr>          ; The name we're giving to this resource
        alias           IN2P3-CC iRODS iCAT             ; A longer name associated with the resource
        hostgroups      fg-irods-icat                   ; Host groups this resource is associated with
        }

define service{
        use                     generic-service
        host_name               <srv.catalog.in2p3.fr>
        service_description     org.irods.irods3.Icat-Connection
        servicegroups           SERVICE_IRODS3_ICAT
        check_command           check_irods_icat_connections
        }

```

Create some Resource hosts
```
# iRods Resource Host Exemple
define host{
        use             linux-server                    ; Inherit default values from a template
        host_name       <srv.resource1.in2p3.fr>        ; The name we're giving to this resource
        alias           IN2P3-R1 iRODS Resource         ; A longer name associated with the resource
        address         <134.158.xxx.yyy>               ; IP address of the irods resource
        hostgroups      fg-irods-resources              ; Host groups this resource is associated with
        contact_groups  admins
        }

# iRODS-All check: test for copying and removing files on each resource
define service{
        use                     generic-service
        host_name               <srv.resource1.in2p3.fr>
        service_description     org.irods.irods3.Resource-All
        servicegroups           SERVICE_IRODS3_RESOURCE
        normal_check_interval   60
        check_command           check_irods_resource_all!<name-of-disk-resource>
        }
```
