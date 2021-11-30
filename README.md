# ansible-playbook-windows-lab

W.I.P.

## Manual tasks pre AD

### all hosts

- set static ip
- run updates
- set correct time
- setup WinRM for ansible

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)

powershell.exe -ExecutionPolicy ByPass -File $file -DisableBasicAuth
```

- update FW rule to only allow in from Ansible host
- update FW rule to not be active on Public network profile

- set network profile to private

```powershell
Get-NetConnectionProfile | Set-NetConnectionProfile -NetworkCategory Private
```

### w10 client
- set built-in administrator password
- enable built-in administrator

### Test

```
$ ansible -i inventory_pre_ad.yml all -m win_ping
FS01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
DC02 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
DC01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
EXC01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
w10_01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Setup AD

```
$ ansible-playbook -i inventory_pre_ad.yml playbooks/ad.yml
```

## Post AD 

- set dns to DCs
- install krb5-user

/etc/krb5.conf

```
[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = true
 dns_lookup_kdc = true
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 default_realm = AD.DOMAIN.TEST
 default_ccache_name = KEYRING:persistent:%{uid}

[domain_realm]
 .ad.domain.test = AD.DOMAIN.TEST
 ad.domain.testm = AD.DOMAIN.TEST
```

- sudo apt-get install gcc libkrb5-dev
- pip3 install pywinrm[kerberos]


### Test Kerberos auth

```
$ ansible -i inventory_post_ad.yml all -m win_ping
```
