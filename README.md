<img src="hub/logo.png" width="350">

# Docker configuration :whale2:

## Prerequisites

Before starting, [download and install docker (machine)](https://docs.docker.com/machine/install-machine/).

### Host setup

Create a Docker host ('machine'), e.g. on Openstack:

`docker-machine create` \
`--driver openstack \` \
`--openstack-auth-url [...] \` Identity API endpoint, located in `dashboard/project/access_and_security/`. \
`--openstack-flavor-name m1.xlarge \` \
`--openstack-image-id [...] \` Base image to launch (which operating system to use), a list of which are located in `dashboard/project/images/`. \
`--openstack-tenant-id [...] \` Tenant (project) ID (where to launch the instance), located in `dashboard/identity/`. \
`--openstack-net-name [...] \` Network in which to create instance, a list of which are located in `dashboard/project/networks/`. \
`--openstack-floatingip-pool public \` \
`--openstack-sec-groups [...] \` Security groups allowing `ssh (22)`, `docker (2376)`, `http/https (80/443)` and port `8080` access. \
`--openstack-username [...] \` \
`--openstack-password [...] \` \
`--openstack-ssh-user [...] \` SSH user to login as, depending on instance base image (e.g. CentOS).
`[machine-name]`

If not supplying existing keys when creating the machine, ensure generated keys in `.docker/machine/machines/[machine-name]` are secured with passphrases, e.g. `ssh-keygen -p -f id_rsa` and `ssh-keygen -p -m PEM -f key.pem` for SSH and TLS keys respectively.

Or, connect to an existing machine using software such as [machine-share](https://github.com/bhurlow/machine-share).

## Configuration

### JupyterHub ([hub](hub/))

JupyterHub image, configured for an (encrypted) MySQL (MariaDB) backend, LDAP authentication and Dockerized JupyterLab container spawning.

Supply the following environment variables (via a `.env` file in the repository root):

`ADMIN_USERS=` List of admin users \
`DATA_LOCATION=` The location of a shared (data) directory on the (remote) host, to be mounted in the home directory of each JupyterHub user.

### JupyterLab ([lab](lab/))

JupyterLab image, tailored for R and Python, plus additional packages as required. Created by JupyterHub on a per-user basis.

1. Edit [lab/Dockerfile](lab/Dockerfile) to reflect any additional R packages needed.

### Database ([db](db/))

(Encrypted) MariaDB backend for JupyterHub.

1. [Setup db encryption keys](https://mariadb.com/kb/en/file-key-management-encryption-plugin/) in [db/encrypt/keys](db/encrypt).

2. Supply the following environment variables:

```
MYSQL_ROOT_PASSWORD=
MYSQL_DATABASE=
MYSQL_USER=
MYSQL_PASSWORD=
```

### Proxy ([proxy](proxy/))

NGINX proxy to sit in front of JupyterHub (and other services).

1. Edit server name in [proxy/locations/jupyter](proxy/locations/jupyter) if listening on anything other than localhost.

2. Generate root and domain certificates using [proxy/certs/gen-ca-cert.sh](proxy/certs/gen-ca-cert.sh) and [proxy/certs/gen-domain-cert.sh](proxy/certs/gen-domain-cert.sh).

### LDAP ([ldap](ldap/))

Authentication server for JupyterHub.

1. Copy [proxy/certs](proxy/certs) to [ldap/certs](ldap/).

2. Supply the following environment variables:

`LDAP_MANAGER_PASSWORD=` \
`LDAP_TLS_CRT_FILENAME=` `.crt` file generated by [proxy/certs/gen-domain-cert.sh](proxy/certs/gen-domain-cert.sh). \
`LDAP_TLS_KEY_FILENAME=` `.key` file generated by [proxy/certs/gen-domain-cert.sh](proxy/certs/gen-domain-cert.sh). \
`LDAP_TLS_CA_CRT_FILENAME=` `.crt` file generated by [proxy/certs/gen-ca-cert.sh](proxy/certs/gen-ca-cert.sh). \
`LDAP_ORGANISATION=` \
`LDAP_DOMAIN=` \
`LDAP_BASE_DN=` \
`LDAP_ADMIN_PASSWORD=` \
`LDAP_TLS_CIPHER_SUITE=` \
`LDAP_TLS_VERIFY_CLIENT=`

### phpLDAPadmin ([phpldapadmin](phpldapadmin/))

UI for administering ldap (e.g. adding new users).

1. Copy [proxy/certs](proxy/certs) to [phpldapadmin/certs](phpldapadmin/).

2. Supply the following environment variables:

`PHPLDAPADMIN_SERVER_PATH=` \
`PHPLDAPADMIN_HTTPS=true` \
`PHPLDAPADMIN_HTTPS_CRT_FILENAME=` `.crt` file generated by [proxy/certs/gen-domain-cert.sh](proxy/certs/gen-domain-cert.sh). \
`PHPLDAPADMIN_HTTPS_KEY_FILENAME=` `.key` file generated by [proxy/certs/gen-domain-cert.sh](proxy/certs/gen-domain-cert.sh). \
`PHPLDAPADMIN_HTTPS_CA_CRT_FILENAME=` `.crt` file generated by [proxy/certs/gen-ca-cert.sh](proxy/certs/gen-ca-cert.sh). \
`PHPLDAPADMIN_LDAP_HOSTS=`

## Deployment

Ensure (remote) machine is activated:

```
eval $(docker-machine env [machine-name])
```

Build these containers:

```
docker-compose build
```

Run these containers:

```
docker-compose up -d
```

## Usage

The main application runs on ports `80` and `443`. `phpldapadmin` runs on port `8080`.

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/martinchapman/nokia-health/tags).

## Authors

[kclhi](https://kclhi.org)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.

## Acknowledgments

* [Medium-scale JupyterHub deployments](https://opendreamkit.org/2018/10/17/jupyterhub-docker/).
