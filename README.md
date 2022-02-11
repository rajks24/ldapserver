# LDAP Server setup with docker-compose

This project aims to build an LDAP Server setup with docker-compose. Within this project, the *phpldapadmin* is integrated with the *openldap* so that, LDAP management can be performed seamlessly.

## Features
- lightweight ldap server
- Environment variables to manage mandatory ldap params
- Ldapserver insecure and secure access
- Add/modify the user and groups in server
- Console to manage LDAPserver
- image versions
  - osixia/openldap:1.5.0
  - osixia/phpldapadmin:0.9.0

## Commands
To access ldapserver externally
- insecure access
```
ldapsearch -x -b "dc=txvlab,dc=local" -H ldap://ldapserver:389 -D "cn=user-ro,dc=txvlab,dc=local" -w pass123
```
- secure access
```
ldapsearch -x -b "dc=txvlab,dc=local" -H ldaps://ldapserver:636 -D "cn=user-ro,dc=txvlab,dc=local" -w pass123
```

#### Note
1. To install openldap client in external VM
```
sudo yum install -y openldap openldap-clients openldap-servers
```
2. To get ldapserver cacert either we can copy from container, or use the extracted cacert from this repo
```
docker exec ldapserver cat /container/service/:ssl-tools/assets/default-ca/default-ca.pem > ldapca.crt
```
3. To enable external VM to secure access, need to add in `/etc/openldap/ldap.conf` the cacert and param to check certs
```
TLS_CACERT	/home/user1/certs/ldapca.crt
TLS_REQCERT     demand
```
4. To disable use of SSL during secure access, we can set variable `export LDAPTLS_REQCERT=never` in external VM

## LDAP Environment Variables

```
LDAP_ORGANISATION=txvlab
LDAP_DOMAIN=txvlab.local
LDAP_ADMIN_USERNAME=admin
LDAP_ADMIN_PASSWORD=pass123
LDAP_CONFIG_PASSWORD=pass123
"LDAP_BASE_DN=dc=txvlab,dc=local"
LDAP_TLS_CRT_FILENAME=server.crt
LDAP_TLS_KEY_FILENAME=server.key
LDAP_TLS_CA_CRT_FILENAME=ldapcacert.crt
LDAP_TLS_VERIFY_CLIENT=try
LDAP_READONLY_USER=true
LDAP_READONLY_USER_USERNAME=user-ro
LDAP_READONLY_USER_PASSWORD=pass123
LDAP_SEED_INTERNAL_LDIF_PATH=/openldap/ldif
```
#### Note
1. we can provide ldap user and group in `ldif` format via the volume mounted to the container and integrated via the LDAP_SEED_INTERNAL_LDIF_PATH env variable
2. To enable debuging in openldap server, we can use the command `command: "--loglevel debug"`


## Exposed ports

**LDAP** : 389
**LDAPS**: 636
**phpldapadmin console** : 8000

## Access phpldapadmin console
**URL:** http://ldapserver.txvlab.local:8000
**username:** cn=admin,dc=txvlab,dc=local
**password:** pass123

## Troubleshooting 
As the ldapserver is running within container, but to access externally, the hostname or domainname for host would be used. Thus, need to ensure that host machine has matching hostname or the domain-name should be resolvable to *ldapserver*.
