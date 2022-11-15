# Documentation: NAGIOS for Health

This nagios repository has been sanatized of all private information in the `objects/hosts/live` directory, `all the `objects/hostgroups/host_hostgroups*` files and this README.md file. It now serves as an example of my work.

## Table of Contents

- [Overview](#overview)
- [Templating](#templating)
  - [Overview](#overview-2)
  - [Creating a Template](#creating-a-template)
  - [Using a Template](#using-a-template)
- [Objects Definitions](#objects-definitions)
  - [Command Objects](#command-objects)
  - [Service Objects](#service-objects)
  - [Hostgroup Objects](#hostgroup-objects)
    - [Service Grouping](#service-grouping)
    - [Host Grouping](#host-grouping)
  - [Host Objects](#host-objects)
  - [Contact Objects](#contact-objects)
  - [Contactgroup Objects](#contactgroup-objects)
  - [Timeperiod Objects](#timeperiod-objects)

# Nagios Core

## Overview

Nagios Core, the software used in this monitoring system, is primarily a tool that performs check scheduling, check execution, check processing, event handling and alerting. Performing checks, sending notifications, processing performance data and other tasks are handled by other Nagios projects or external applications.

Nagios is a framework that you build your monitoring solution out of, and so there are several configuration files that you will need to create before anything can be monitored.

## nagios.cfg

The main configuration file, `nagios.cfg`, is stored in the root of the nagios configuration directory. Typically this will be the `/etc/nagios/` directory, making the file `/etc/nagios/nagios.cfg`. This file is used to link to the object definition files stored in the `objects` directory, or `/etc/nagios/objects/`. For the rest of this documentation directories and files will be referred to by their relative path to the root directory for the nagios configuration.

## Resources Files

Resource files can be used to store user-defined macros and variables. The variables in the resource files can be used in the object definitions and can include sensitive configuration information (like passwords) without making them available to the CGIs.

The resource files are stored in the `prviate` directory, the default file `private/resource.cfg` is already there and configured. You can specify one or more optional resource files by using the `resource_file` directive in your main configuration file.

E.g.

    resource_file=/etc/nagios/private/resource.cfg

Be sure to change the permissions of each of the resource files using `chmod 640 /path/to/resource/file` and `chown root:nagios /path/to/resource/file` so the secrets cannot be read by users other than root, or by processes other than nagios. If you list the file using the `ls -lh` command, you should see something like this:

    [root@nec-nagios-ccsrp ~]# ls -lh /etc/nagios/private/resource.cfg
    -rw-r-----. 1 root nagios 1.3K Jul 22 11:01 /etc/nagios/private/resource.cfg

## Object Definitions

Object definition files are used to define the things nagios should be monitoring, how they monitor the things, and how the notify contacts about the things. The objects included in this configuration are hosts, hostgroups, services, contacts, contactgroups, timeperiods and commands. A description of each is available further below.

The object definition files are stored in the `objects` directory and are included in the `nagios.cfg` file by using the `cfg_file` and `cfg_dir` parameters.

E.g.

    cfg_dir=/etc/nagios/objects/commands
    cfg_file=/etc/nagios/objects/somefile.cfg

The `cfg_dir` option includes all `.cfg` files within that directory.

Nagios Core's website has all the objects definitions and all their directives documented [here](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/objectdefinitions.html). The Nagios core objects documentation provides more detail on what each directive is than what is covered here.

# Templating

## Overview

Templating, or object inheritance in nagios-speak, is a way to reduce the size of the configuration and reduce human related data entry errors. The benefit of using the template-based config file format is that you can create object definitions that have some or most of their properties inherited from other object definitions.

Objects definitions of the same type as the template definition can use the template with the `use` directive. Multiple templates can be used on a single object by listing them in the order of inheritance. The templates will be merged in the order they are listed on the `use` line. Local directives will always take priority over directives in a template.


## Creating a Template

Template definitions are stored in the `objects/templates/*.cfg` files. Each is named for the type of template definition, e.g. `objects/templates/contacts.cfg`.

Create a new file, or edit an exiting template. The contents of the template definition can be any directive that would normally be applied to the type of definition. There are three special directives to be aware of:

 The first directive is `name`. Its just a "template" name that can be referenced in other object definitions so they can inherit the objects properties/variables. Template names must be unique amongst objects of the same type, so you can't have two or more host definitions that have "hosttemplate" as their template name.

The second directive is `use`. This is where you specify the name of the template object that you want to inherit properties/variables from. The name you specify for this variable must be defined as another object's template named (using the name variable). This allows you to chain templates together.

The third directive is `register`. This variable is used to indicate whether or not the object definition should be "registered" with Nagios. By default, all object definitions are registered. If you are using a partial object definition as a template, you would want to prevent it from being registered. Values are as follows:
 - 0 = do NOT register object definition.
 - 1 = register object definition (this is the default).

 This directive is NOT inherited; every (partial) object definition used as a template must explicitly set the register directive to be 0. This prevents the need to override an inherited register directive with a value of 1 for every object that should be registered.

## Using a Template

Using a template in a normal object definition is done by specifying one or more templates of the same type as the object in the `use` directive within the object. Templates are merged into the object definition in the order they are listed. If there were two templates applied to an object, `first` and `second`, and both templates change the same directive, `second` will overwrite the directive set by `first`.

For example, to include the `host-template1` and `ntg-css-hosts` templates in that order in a `host` object, the following can be done. See the `use` line?


    define host{
    use                     host-template1,ntg-ccs-hosts
    host_name               dhdrhbasp078.sanatized_domain.tld
    alias                   dhdrhbasp078.sanatized_domain.tld
    address                 203.0.113.111
    }

To include the `contact_by_email` and `contact_24x7` templates in a `contact` object, the following can be done. See the `use` line?

    define contact{
    contact_name        admin
    alias               Default Admin Contact
    email               admin-user@example.com
    use                 contact_by_email,contact_24x7
    }

In both of the above examples, each of the templates named are of the same type. See below, the contents of the templates has been removed in this example.

The `host` templates are defined as a `host` object.

    define host {
    name                            host-template1
    ...
    register                        0
    }


    define host {
    name                            ntg-ccs-hosts
    ...
    register                        0
    }

The `contact` templates are defined as a `contact` object.

    define contact{
    name                            contact_by_email
    ...
    register                        0
    }


    define contact{
    name                            contact_24x7
    ...
    register                        0
    }

# Objects Definitions

## Command Objects

A command definition is just that, it defines a command. The command definitions can be found in the `objects/commands/*.cfg` files. The different command types have been separated into different files to make it easier to find where each is, and to provide some guidance on how to expand this configuration should more commands be added in future.

- HTTP and HTTPS checking command definitions are in the `objects/commands/http_commands.cfg` file.
- TCP port checking command definitions are in the `objects/commands/tcp_commands.cfg` file.
- SSH availability command definitions are in the `objects/commands/ssh_commands.cfg` file.

Each of the command definitions has a comment above it to describe its usage as each have one or more arguments that must be supplied when calling the command in the service object.

##### Example

The below command checks a https server is responding and has options to alert with a warning or error message based on the delay in the response. The `$ARG1$` and `$ARG2$` are defined in the service object (example below). The `$HOSTADDRESS$` is automatically provided by Nagios using the `address` field of the `host` object that the service is being applied to. The `$USER1$` variable is defined in the `private/resource.cfg` file and is the path to where the nagios plug-ins are stored.

    # Check a https server responds.  Warn if responds in longer than $ARG1$ seconds but less than $ARG2$, critical if more than $ARG2$ seconds.
    define command{
    command_name        check_https
    command_line        $USER1$/check_http --ssl -H $HOSTADDRESS$ -w $ARG1$ -c $ARG2$
    }

The `$ARG1$` and `$ARG2$` in the above command object are defined in the below service object in the `check_command` field. The `check_https!5!10` line means: Call the `check_https` command definition with `5` being the first argument and `10` being the second argument. The arguments are separated by `!` marks.

    # Check HIPS Web Front-End response
    define service{
    hostgroup_name                  hips_web_fe
    service_description             HIPS Web Front-End
    check_command                   check_https!5!10
    use                             generic-service,hips_service
    notification_interval           0 ; set > 0 if you want to be renotified
    }

Using the information of both the above examples, what will this service check do?

If you answered 'it produces a warning alert if a https response takes longer than 5 seconds and produces a critical alert if it takes longer than 10', you would be correct.


## Service Objects

A service definition is used to identify a "service" that runs on a host. The term "service" is used very loosely. It can mean an actual service that runs on the host (POP, SMTP, HTTP, etc.) or some other type of metric associated with the host (response to a ping, number of logged in users, free disk space, etc.).

Each service object definition includes a `hostgroup_name`, `service_description`, `check_command` and `use` directive. From the earlier sections in this document, you should already understand what the `check_command` (See: [Command Objects](#command-objects)) and `use` (See: [Templating](#using-a-template)) directives are used for.

The `service_description` directive is used to define the description of the service, which may contain spaces, dashes, and colons (semicolons, apostrophes, and quotation marks should be avoided). No two services associated with the same host can have the same description. Services are uniquely identified with their `hostgroup_name` and `service_description` directives.

The `hostgroup_name` directive is used to list the short-names of of the hostgroups that the service runs on. Multiple hostgroups can be specified in a comma separated list.

Service object definitions have been split into different configuration files to keep them from growing too large. HTTP and HTTPS service checks are in the `objects/services/http_services.cfg`. The TCP and SSH service checks are in their own similarly named files in the same `objects/services/` directory.

## Hostgroup Objects

A hostgroup definition is used to group one or more hosts together for simplifying configuration.

The hostgroups object definitions are stored in the `objects/hostgroups/host_hostgroups.cfg` and `objects/hostgroups/service_hostgroups.cfg` files.

 - `host_hostgroups.cfg` stores hostgroups where individual servers are members.
 - `service_hostgroups.cfg` stores hostgroups that are targets of service objects.

The hostgroups from `host_hostgroups.cfg` are added as `hostgroup_members` to the `service_hostgroups.cfg` hostgroups so a group of hosts, or multiple groups of hosts can be checked for a specific service, or a group of services. This is possible thanks to the ability to nest hostgroups within each other.

### Service Grouping

As we learned in the previous section, service objects can be applied to a hostgroup and hostgroups can be nested. In this configuration, all services have their own hostgroup and where applicable, several service hostgroup's are nested inside another hostgroup.

E.g.

An entry from the `objects/hostgroups/service_hostgroups.cfg` file that checks each of the ports for CCIS. See the `hostgroup_name`? It will be referenced in the next example.

    # CCIS Application Ports
    define hostgroup {
    hostgroup_name      ccis_application_port_all
    alias               CCIS Application Ports
    hostgroup_members   ccis_hosts
    }

All of the specific port checks have their own hostgroup defined in `object/hostgroups/service_hostgroups_ccis.cfg`. Take note of the `hostgroup_members` in the below examples.


    # CCIS Application Port 6011
    define hostgroup {
    hostgroup_name      ccis_application_port_6011
    alias               CCIS Application Port 6011
    hostgroup_members   ccis_application_port_all
    }

    # CCIS Application Port 6012
    define hostgroup {
    hostgroup_name      ccis_application_port_6012
    alias               CCIS Application Port 6012
    hostgroup_members   ccis_application_port_all
    }

    ...

    # CCIS Application Port 6083
    define hostgroup {
    hostgroup_name      ccis_application_port_6083
    alias               CCIS Application Port 6083
    hostgroup_members   ccis_application_port_all
    }

Other services are also split up in this way. `CCIS`, `PCIS`, `HIPS` and `Citrix` service checks are nested in this way.

### Host Grouping

Hosts are bundled into groups such as HIPS `hips_hosts`, PCIS `pcis_hosts`, CCIS `ccis_hosts` and Citrix `citrix_services_hosts`. These hostgroups do not have services directly tied to them through the service objects `hostgroup_name` directive, instead they are used as members via the `hostgroup_members` directive of the hostgroups used to bundle service checks together.

The hostgroups for the Citrix hosts have been split into multiple hostgroups based on the servers name. The `dhdrhxenp*` hosts are split into several subgroups based on the hosts number. All these are stored in the `objects/hostgroups/host_hostgroups_citrix.cfg`.

 - `citrix_xenp_group1_hosts` is xenp110-119
 - `citrix_xenp_group2_hosts` is xenp120-129
 - `citrix_xenp_group3_hosts` is xenp130-139
 - `citrix_xenp_group4_hosts` is xenp140-149
 - `citrix_xenp_group5_hosts` is xenp150-159
 - `citrix_xenr_group1_hosts` is all xenr hosts
 - `citrix_xent_group1_hosts` is all xent hosts

The difference between the `xenp`, `xenr` and `xent` are not known to me but must symbolise some difference of role, location or type. It was best to split them up now so that role, type or location specific service checks can be applied against the groups without major configuration changes.

## Host Objects

 Hosts are one of the central objects in the monitoring logic. Important attributes of hosts are as follows:

 - Hosts are usually physical devices on your network (servers, workstations, routers, switches, printers, etc), or virtual machines.
 - Hosts have an address of some kind (e.g. an IP or MAC address).
 - Hosts have one or more more services associated with them.
 - Hosts can have parent/child relationships with other hosts, often representing real-world network connections, which is used in the network reachability logic. See [Determining Status and Reachability of Network Hosts](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/networkreachability.html) for more info.

Hosts in this configuration are just the `use`, `host_name`, `alias` and `address` attributes. All other directives are sourced from the templates in the `use` directive.

 - The `use` directive is something that has been covered in [Templating](#using-a-template)
 - The `host_name` directive is used to define a short name used to identify the host. It is used in hostgroup definitions to reference this particular host. When used properly, the `$HOSTNAME$` macro will contain this short name.
 - The `alias` directive is used to define a longer name or description used to identify the host. It is provided in order to allow you to more easily identify a particular host. When used properly, the `$HOSTALIAS$` macro will contain this alias/description.
 - The `address` directive is used to define the address of the host. Normally, this is an IP address, although it could really be anything you want (so long as it can be used to check the status of the host). You can use a FQDN to identify the host instead of an IP address, but if DNS services are not available this could cause problems. When used properly, the `$HOSTADDRESS$` macro will contain this address. See the [Command Objects](#command-objects) section for how `$HOSTADDRESS$` is used.

Hosts have been sorted into several files to make it easier to find them.

 - Hosts that were only provided as an IP address are in the `objects/hosts/live/ip_address.cfg` file.
 - Hosts with FQDN that match:
   - `cprod.sometld.tld` are in the `objects/hosts/live/cprod.sometld.tld.cfg` file.
   - `gov.sometld.tld` are in the `objects/hosts/live/gov.sometld.tld.cfg` file.
   - `prod.sometld.tld` are in the `objects/hosts/live/prod.sometld.tld.cfg` file.

##### Example

In this example, the host object was picked from the `objects/hosts/live/prod.sometld.tld.cfg` file.

    define host{
    use                     host-template1,healthshare_host
    host_name               hbasp078.prod.sometld.tld
    alias                   hbasp078.prod.sometld.tld
    address                 203.0.113.177
    }

The above host has two templates (shown below) from the `objects/templates/hosts.cfg` file. The `host-template1` is setting the more generic of settings, how often to check the host, how often to retry checks, what timeperiod to use for checking the host, what timeperiod to use for notifications, etc. Full details can be found at [the host object definition documentation](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/objectdefinitions.html#host) on the nagios website.

    define host {
    name                            host-template1
    use                             generic-host
    check_command                   check-host-alive
    check_interval                  5
    retry_interval                  1
    max_check_attempts              5
    check_period                    24x7
    process_perf_data               0
    retain_nonstatus_information    0
    notification_interval           30
    notification_period             24x7
    notification_options            d,u,r
    register                        0
    }

The `healthshare_host` template is only used to change which contact group gets notified, but more directives can be set to override the directives from the `host-template1` template.

    # HealthShare Host Template
    define service{
    name                    healthshare_host
    use                     generic-host
    contact_groups          healthshare_admins
    register                0
    }

## Contact Objects

Contacts are people involved in the notification process and are added to contactgroup objects as members. While it is possible to add a contact directly to a service or host, it is preferable to use a contactgroup. All services and hosts in this configuration use contactgroups as the receiver of notifications.

Contacts can have different ways to be notified which are defined by the `service_notification_commands` and `host_notification_commands` directives, the options for these are command objects which are stored in the `objects/commands/notifications_commands.cfg` file.

Each contact can be configured to be notified only during specific timeperiods for services and hosts using the `service_notification_period` and `host_notification_period` directives. Timeperiods are objects that can be configured, e.g. 24x7, workhours, afterhours, etc. Timeperiods are defined in the `objects/timeperiods/timeperiods.cfg` file.

The `service_notification_options` and `host_notification_options` directives are used to specify which type of notifications are sent to the contact. These directives take a comma separated list of options like `w,u,c,r`.

 - `service_notification_options` options
   - w = notify on WARNING service states.
   - u = notify on UNKNOWN service states.
   - c = notify on CRITICAL service states.
   - r = notify on service recoveries (OK states).
   - f = notify when the service starts and stops flapping.
   - n = the contact will not receive any type of service notifications.
 - `host_notification_options` options
   - d = notify on DOWN host states.
   - u = notify on UNREACHABLE host states.
   - r = notify on host recoveries (UP states).
   - f = notify when the host starts and stops flapping.
   - s = send notifications when host or service scheduled downtime starts and ends.
   - n = the contact will not receive any type of host notifications.

##### Example

The default contact that is provided with this configuration is as shown below. Refer to the `objects/templates/contacts.cfg` file for the specifics of the templates used. Other examples are provided in the `objects/contacts/contacts.cfg` file.

    define contact{
    contact_name        admin
    alias               Default Admin Contact
    email               admin-user@example.com
    use                 contact_by_email,contact_24x7
    }


## Contactgroup Objects

A contact group definition is used to group one or more contacts together for the purpose of sending out alert and recovery notifications.

To change where service or host notifications are sent, refer to the `use` directive of the host or service object. Updating the template will update the `contact_group` for all the hosts or services using that template.

 - service objects import their `contact_groups` directive from templates in the `objects/templates/services.cfg` file.
 - host objects import their `contact_groups` directive from templates in the `objects/templates/hosts.cfg` file.

The contactgroup objects are limited to `contactgroup_name`, `alias`, `members` and `contactgroup_members`. Multiple contact names should be separated by commas in the `members` directive, and multiple contactgroups should be separated by commas in the `contactgroup_members` directive. This allows for nesting of contact groups.

##### Example


    # Group for HIPS Services
    define contactgroup{
    contactgroup_name       hips_admins
    alias                   HIPS Notification Group
    members                 admin
    }


    # Group for PCIS Services
    define contactgroup{
    contactgroup_name       pcis_admins
    alias                   PCIS Notification Group
    members                 admin
    }


    # Example of nested groups
    define contactgroup{
    contactgroup_name       all_groups
    alias                   All Notification Groups
    contactgroup_members    hips_admins,pcis_admins
    }

## Timeperiod Objects

A time period is a list of times during various days that are considered to be "valid" times for notifications and service checks. It consists of time ranges for each day of the week that "rotate" once the week has come to an end. Different types of exceptions to the normal weekly time are supported, including: specific weekdays, days of generic months, days of specific months, and calendar dates.

Timeperiod objects are used:

 - In the host templates in `objects/templates/hosts.cfg` to define when a host is checked and when notifications for a host can be sent.
 - In the contact templates in `objects/templates/hosts.cfg` to define when a specific contact can be notified for services and hosts.

Typically, all the hosts and services will be monitored on the `24x7` timeperiod defined in the `objects/timeperiods/timeperiods.cfg` file. A new timeperiod can be created to exclude specific hours if a service is deliberately stopped on a schedule (E.g. for a backup task) so that notifications are not sent for that host during that timeperiod.

##### Examples

The below two examples are in the `objects/timeperiods/timeperiods.cfg` file. The `24x7` timeperiod is applied to all the hosts and services, but the `workhours` timeperiod is applied to contacts. You can of course have contacts that use the `24x7` timeperiod too, like an after hours on-call engineers phone/email, or you can use the `afterhours` template (not shown here) so that it only receives notifications during non-work hours.

    # 24x7 Monitoring Hours
    define timeperiod{
    timeperiod_name 24x7
    alias			24 Hours A Day, 7 Days A Week
    sunday			00:00-24:00
    monday			00:00-24:00
    tuesday			00:00-24:00
    wednesday		00:00-24:00
    thursday		00:00-24:00
    friday			00:00-24:00
    saturday		00:00-24:00
    }

    # Standard Work Hours
    define timeperiod{
    timeperiod_name workhours
    alias			Standard Work Hours
    monday			08:00-17:00
    tuesday			08:00-17:00
    wednesday		08:00-17:00
    thursday		08:00-17:00
    friday			08:00-17:00
    }

The below example is from the [the timeperiod object definition documentation](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/objectdefinitions.html#timeperiod) on the nagios website. It shows you can get quite complex with the timeperiods.


    define timeperiod {
    timeperiod_name                 misc-date-ranges
    alias                           Misc Date Ranges
    2007-01-01 - 2008-02-01         00:00-24:00         ; January 1st, 2007 to February 1st, 2008
    monday 3 - thursday 4           00:00-24:00         ; 3rd Monday to 4th Thursday of every month
    day 1 - 15                      00:00-24:00         ; 1st to 15th day of every month
    day 20 - -1                     00:00-24:00         ; 20th to the last day of every month
    july 10 - 15                    00:00-24:00         ; July 10th to July 15th of every year
    april 10 - may 15               00:00-24:00         ; April 10th to May 15th of every year
    tuesday 1 april - friday 2 may  00:00-24:00         ; 1st Tuesday in April to 2nd Friday in May of every year
    }
