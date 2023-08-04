# Samba server

Tested features:

- Join the domain (Windows & Debian-like)
- Log in
- Basic account and group management via RSAT Tools (or samba-tool)
- Group policy management via RSAT Tools (or samba-tool)
- Share management via configuration and Computer Management.

Sources:

https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller

https://adamtheautomator.com/samba-active-directory/

https://www.considerednormal.com/2022/11/samba-based-active-directory-on-ubuntu-22-04/

https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/add-computer

https://www.server-world.info/en/note?os=Debian_11&p=realmd

# Set hostname as "DC1"

sudo hostnamectl hostname dc1

sudo reboot

# Example resolv.conf

sudo nano /etc/resolv.conf

```
domain example.lan
search example.lan
nameserver aaa.bbb.ccc.ddd
nameserver 1.1.1.1
```

# Install needed packages from apt

sudo apt -y install acl attr chrony samba samba-common samba-common-bin samba-dsdb-modules samba-vfs-modules smbclient winbind libpam-winbind libnss-winbind libpam-krb5 krb5-config krb5-user dnsutils net-tools dnsmasq

# Example hosts for DNSMasq

sudo nano /etc/hosts

Place to first lines:
```
aaa.bbb.ccc.ddd dc1.example.lan dc1
aaa.bbb.ccc.ddd example.lan
```

# Example records for DNSMasq

sudo nano /etc/dnsmasq.d/example.records.conf

Place to lines:

```
srv-host=_ldap._tcp.example.lan,dc1.example.lan,389`
srv-host=_kerberos._udp.example.lan,dc1.example.lan,88
srv-host=_kerberos._tcp.example.lan,dc1.example.lan,88
srv-host=_kerberos-adm._tcp.example.lan,dc1.example.lan,749
srv-host=_gc._tcp.example.lan,dc1.example.lan,3268
srv-host=_ldap._tcp.dc._msdcs.example.lan,dc1.example.lan,389
srv-host=_ldap._tcp.gc._msdcs.example.lan,dc1.example.lan,3268
srv-host=_kerberos._tcp.dc._msdcs.example.lan,dc1.example.lan,88
srv-host=_kerberos._udp.dc._msdcs.example.lan,dc1.example.lan,88
```

# Restart DNSMasq

sudo service dnsmasq restart

# Enable Samba ad dc

sudo systemctl disable --now smbd nmbd winbind

sudo systemctl stop --now smbd nmbd winbind

sudo systemctl unmask samba-ad-dc

sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.og

sudo mv /etc/krb5.conf /etc/krb5.conf.og

sudo samba-tool domain provision

Type: example.lan two times, none and your administrator password two times

sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf

sudo chown root:_chrony /var/lib/samba/ntp_signd

sudo nano /etc/chrony/chrony.conf

Place to last lines:

```
# Bind the chrony service to IP Address
bindcmdaddress aaa.bbb.ccc.ddd
# Allow clients on the network to connect to the Chrony NTP server
allow 0.0.0.0/8
# Specify the ntpsigndsocket directory for the Samba AD
ntpsigndsocket /var/lib/samba/ntp_signd
```

sudo systemctl enable samba-ad-dc

sudo systemctl start samba-ad-dc


# Check services

sudo systemctl status dnsmasq

sudo systemctl status samba-ad-dc

sudo systemctl status chronyd

# Test server records

host -t A example.lan && host -t A dc1.example.lan && host -t SRV _kerberos._udp.example.lan && host -t SRV _ldap._tcp.example.lan

# Example open ports for ufw

sudo apt -y ufw

sudo ufw allow proto tcp from 0.0.0.0/8 to any port 53,636

sudo ufw allow proto udp from 0.0.0.0/8 to any port 53,636

sudo ufw allow proto tcp from 0.0.0.0/8 to any port 3268,3269

sudo ufw allow proto udp from 0.0.0.0/8 to any port 3268,3269

sudo ufw allow proto tcp from 0.0.0.0/8 to any port 49152,65535

sudo ufw allow proto udp from 0.0.0.0/8 to any port 49152,65535

sudo ufw enable

# Test your domain

kinit administrator@EXAMPLE.LAN <- DOMAIN IN UPPERCASE

klist

# Windows Workstation

Open Powershell with Administrator Rights.

Type: Add-Computer -DomainName example.lan -Restart

Type EXAMPLE\Administrator for user, then your administrator password.

Computer will join the domain and it will take a time, then restarts.

# Debian-like Workstation

sudo apt -y install krb5-user realmd libnss-sss sssd sssd-tools adcli samba-common-bin oddjob oddjob-mkhomedir package libpam-mkhomedir libsss-sudo packagekit

sudo nano /etc/sudoers.d/domain_admins

Type to lines:

```
%domain\ admins@example.lan ALL=(ALL) ALL
```

# Discover your domain

sudo realm discover example.lan

# Join the domain

sudo realm join -U administrator example.lan

sudo nano /etc/pam.d/common-session

Type:

```
session optional pam_mkhomedir.so skel=/etc/skel/ umask=0022
```

sudo reboot

Log in as administrator: administrator@example.lan
Then your administator password.

Should be done... and enjoy.

# Notes

- aaa.bbb.ccc.ddd is your server ip.
- dc1.example.lan and example.lan are your server domains.
- EXAMPLE is your netbios name.
