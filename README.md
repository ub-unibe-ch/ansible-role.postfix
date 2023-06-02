# Ansible Role: Postfix

An Ansible role that manages Postfix. Currently the role has the following
features:

* Install Postfix
* Manage config settings in `/etc/postfix/main.cf`, see
  [role variables](#role-variables)
* Manage the file-base alias db `/etc/alias`.
* Completely uninstall Postfix

## Requirements

No prerequisites necessary at the moment.

## Role Variables

Available variables are listed below, along with default values (see also `defaults/main.yml`):

Variables of the form `$varname` are literally transported to the config file as
Postfix is internally interpolating these on service startup. In other words,
technically, setting an `postfix_myorigin` to "$nyhostname" has the very same outcome
as setting it to `"{{ ansible_fqdn }}"`.

### postfix_myhostname

    postfix_myhostname: "{{ ansible_fqdn }}"

The `myhostname` parameter specifies the internet hostname of this mail system.
The default is to use the fully-qualified domain name

### postfix_mydomain

    postfix_mydomain: "undef"

The `mydomain` parameter specifies the local internet domain name. The default
is "undef" that meas Postfix will compute the value based on its defaults, which
results in $myhostname minus the first component.

### postfix_myorigin

    postfix_myorigin: "undef"

The `myorigin` parameter specifies the domain that locally-posted mail appears to
come from. The default is to append $myhostname, which is fine for small
sites.

### postfix_mydestination

    postfix_mydestination: "$myhostname, localhost.$mydomain, localhost"

The `mydestination` parameter specifies the list of domains that this machine
considers itself the final destination for.  
The default is \$myhostname + localhost.$mydomain + localhost.  On a mail domain
gateway, you should also include $mydomain.

### postfix_inet_interfaces

    postfix_inet_interfaces: localhost

The `inet_interfaces` parameter specifies the network interfaces addresses that
this mail system receives mail on. The default is to listen only on `localhost`.
You can specify more than one by comma separating them. Set to `all` for all
available interfaces or to a comma-separated list of IP addresses. See [postconf
manpage](https://www.postfix.org/postconf.5.html#inet_interfaces) for the
details about this option.

### postfix_inet_protocols

    postfix_inet_protocols: all

This parameter specifies the internet protocols to support. The default is
`all`, which means IPv4, and IPv6 if supported. See [postconf
manpage](https://www.postfix.org/postconf.5.html#inet_protocols) manpage for the
details about this option.

### postfix_relayhost

    postfix_relayhost: undef

The relayhost parameter specifies the default host to send mail to. Specify a
domain, host, host:port, [host]:port, [address] or [address]:port; the form
[host] turns off MX lookups. Examples:

    # Set to $mydomain => mail will be sent to the smtp found in the MX entry of
    $mydomain
    postfix_relayhost = $mydomain

    # No MX lookup is done, all mails are sent the given MTA
    postfix_relayhost = [gateway.my.domain]

    # Same as above, MTA set by IP address
    postfix_relayhost = [an.ip.add.ress]

    postfix_state: started

### Manipulating Postfix Lookup Maps

#### Alias Map

    postfix_alias_map: {}

This allows manipulations of the `hash:/etc/alises` map. The default is not to edit
this file in any way. Use this to forward local mails to external addresses,
which is useful for mails to the local root account.

Examples:

    postfix_alias_map:
      www: "user1@test.com"
      root: ["user2@test.com", "user3@example.com"]

Use the local account you want to forward mails for as the key and either a
string or list of strings for the mail addresses to forward these mails to. In
the above example, mails to root are forwarded to two external addresses instead.

### Configuring Package and Service State

    postfix_enabled: true

This sets the boot time status of the Postfix daemon. Should remain to `true`
unless the daemon shouldn't be automatically started at boot time.

    postfix_state: started

Set the initial state of the postfix daemon when this role is run. This should
generally remain `started`. There occasions where setting this to `stopped`
might come in handy, i.e. during a maintenance down. In combination with the
`postfix_packages_state` this also allows to purge Postfix from a system.

    postfix_restart_state: restarted

This determines the measure that is taken when the `postfix` handlers
called. The default is to restart the Postfix process, when the handlers kicks
in. Can be set to `reloaded` to only reload the Postfix process.

    postfix_packages_state: present

The state of the Postfix installation packages. Defaults to `present` to make
sure it's installed. Can also be set to `latest` if Postfix should get removed
from the machine or prior switching repo or the like.

## Dependencies

This role has no dependencies to other roles.

## Example Playbook

Including an example of how to use your role (for instance, with variables
passed in as parameters) is always nice for users too:

    - hosts: servers

      roles:
         - unibe_idsys.postifx

<!-- add an example which illustrates a standard usage internally? -->

The following example illustrates how to remove Postfix from the systems again:

    - hosts: servers

      roles:
        - role: unibe_idsys.postifx
          vars:
            postfix_service_state: stopped
            postfix_packages_state: absent

**Note:** Removing Postfix from the system will also purge any configuration in
`/etc/postfix`. If you want to preserve it, make a backup before applying the
above play.

## Compatibility

This role has been written for and tested on and is therefore compatible with:

* CentOS-7
* Rocky-8, Rocky-9
* Ubuntu-20.04, Ubuntu-22.04

## License

MIT

## Author Information

The role was created in 2023 by the IT-Services Office of the University of Bern
