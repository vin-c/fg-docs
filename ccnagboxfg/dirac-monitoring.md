# DIRAC monitoring installation

## Binaries

Get the scripts from https://github.com/FranceGrilles/monitoring-dirac

`git clone https://github.com/FranceGrilles/monitoring-dirac.git /usr/lib/monitoring-dirac`

Don't forget to configure it ! See README.md file inside the repository.

## Nagios configuration

Update nagios configuration with the following.

Note : values like `<exemple>` must be modified according to your environment...

```
define command{
        command_name    check_dirac_submit
        command_line    $USER4$/check_dirac_simple.sh --submit
}

define command{
        command_name    check_dirac_jobs
        command_line    $USER4$/check_dirac_simple.sh --check
}

define hostgroup{
        hostgroup_name  Dirac-sites
        alias           Dirac Sites
        members         <member1, member2>
}

# Host template definition
define host{
        name                            dirac-template
        use                             host-template
        icon_image                      dirac.png
        statusmap_image                 dirac.gd2
        register                        0
        }
```

Update `/etc/nagios/private/resource.cfg` with
```
$USER4$=/usr/lib/monitoring-dirac/plugins
```

Create some hosts
```
define host{
        use                     dirac-site
        alias                   FGDirac <SITE>
        host_name               dirac-conf.in2p3.fr
        address                 dirac-conf.in2p3.fr
}

define service{
        use                             generic-service
        host_name                       dirac-conf.in2p3.fr
        service_description             Dirac API
        check_command                   check_tcp!9135
        check_interval                  30
        retry_interval                  15
        }

define service{
        use                             generic-service
        host_name                       dirac-conf.in2p3.fr
        service_description             Dirac job submission
        check_command                   check_dirac_submit
        check_interval                  60
        retry_interval                  30
        }

define service{
        use                             generic-service
        host_name                       dirac-conf.in2p3.fr
        service_description             Dirac jobs workflow check
        check_command                   check_dirac_jobs
        check_interval                  60
        retry_interval                  30
        }
```
