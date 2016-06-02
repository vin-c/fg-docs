# OpenStack Cloud monitoring installation

## Binaries

Get the scripts from https://github.com/FranceGrilles/monitoring-cloud

`git clone https://github.com/FranceGrilles/monitoring-cloud.git /usr/lib/monitoring-cloud`

Don't forget to configure it ! See README.md file inside the repository.

## Nagios configuration

Update nagios configuration with the following.

Note : values like `<exemple>` must be modified according to your environment...

```
# site-config-file without full path
# File must be in /usr/lib/monitoring-cloud/config

define command{
        command_name    check_openstack
        command_line    $USER2$/check_openstack.sh -c $ARG1$ -- $ARG2$
        ; $ARG1$ = site-config-file
        ; $ARG2$ = tempest.test.to.run
}

define command{
        command_name    check_isolation
        command_line    $USER2$/check_isolation.sh -a $ARG1$ -b $ARG2$
        ; $ARG1$ = site-config-file-a
        ; $ARG2$ = site-config-file-b
}

define command{
        command_name    check_openstack_regex
        command_line    $USER2$/check_openstack.sh -c $ARG1$ -e $ARG2$
        ; $ARG1$ = site-config-file
        ; $ARG2$ = '(^tempest\.regex\.test_basic_(scenario|values))'
}

define hostgroup{
        hostgroup_name  openstack-sites
        alias           Openstack Sites
        members         <member1, member2>
}

# Host template definition
define host{
        name                            openstack-site  ; The name of this host template
        notifications_enabled           1               ; Host notifications are enabled
        event_handler_enabled           1               ; Host event handler is enabled
        flap_detection_enabled          1               ; Flap detection is enabled
        process_perf_data               1               ; Process performance data
        retain_status_information       1               ; Retain status information across program restarts
        retain_nonstatus_information    1               ; Retain non-status information across program restarts
        notification_period             24x7            ; Send host notifications at any time
        register                        0               ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL HOST, JUST A TEMPLATE!
        check_period                    24x7            ; By default, Linux hosts are checked round the clock
        check_interval                  5               ; Actively check the host every 5 minutes
        retry_interval                  1               ; Schedule host check retries at 1 minute intervals
        max_check_attempts              10              ; Check each Linux host 10 times (max)
        check_command                   check-host-alive ; Default command to check Linux hosts
        notification_interval           120             ; Resend notifications every 2 hours
        notification_options            d,u,r           ; Only send notifications for specific host states
        contact_groups                  admins          ; Notifications get sent to the admins by default
        register                        0               ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL HOST, JUST A TEMPLATE!
        action_url                      /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=_HOST_' class='tips' rel='/pnp4nagios/index.php/popup?host=$HOSTNAME$&srv=_HOST_
        }
```

Update `/etc/nagios/private/resource.cfg` with
```
$USER2$=/usr/lib/monitoring-cloud
```

Create some hosts
```
define host{
        use                     openstack-site
        alias                   FGCloud <SITE>
        host_name               <site>
        address                 <srv.openstack.in2p3.fr>
}

define service{
        use                             generic-service
        host_name                       <site>
        service_description             Openstack API
        check_command                   check_tcp!5000
        check_interval                  30
        retry_interval                  15
        }

define service{
        use                             generic-service
        host_name                       <site>
        service_description             Openstack Scenario
        check_command                   check_openstack!<tempest-site.conf>!tempest.api.fgcloud.test_basic_scenario
        check_interval                  1440
        retry_interval                  30
        }

define service{
        use                             generic-service
        host_name                       <site>
        service_description             Openstack Isolation Test
        check_command                   check_isolation!<tempest-site1.conf>!<tempest-site2.conf>
        check_interval                  10080
        retry_interval                  1440
        }
```
