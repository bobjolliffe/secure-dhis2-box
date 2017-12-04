# secure-dhis2-box
Description of a standalone dhis2 installation

This repository describes an installation of dhis2 running on a vps server.  The server is intended to be used for collecting ART data in a public hospital using dhis2 tracker.  The load is expected to be low with very few users.  The constraints on the resources are very tight: a single VPS server with a single ssd disk and 8GB of RAM.  The emphasis is on a reasonably secure setup rather than high performance.

## Overall approach

### Containerisation
In order to provide some isolation, the web proxy, the web application and the database are each running in separate containers.  The host system is running ubuntu server 16.04 LTS.  The three containers are implemented using lxd.

Details of the lxd setup are decribed in [containers.md].

### System hardening
The base system and containers are all based on ubuntu 16.04.  Various hardening measures which have been implemented are described in [hardening.md].  These measures implement a subset of the CIS benchmarks for ubuntu systems as well as specific measures for tomcat and apache.

### Encryption at rest
The database container is running postgresql 9.5, with access controls configured so that it is only accessable over the network from the application server.  The data for the database cluster is stored on an encrypted "disk" to achieve encryption at rest.

The implementation of the encrypted data area is not ideal, but tailored to the constraints of the environment.  A 60GB disk file in the host system is mounted on a loopback device.  *cryptsetup* is used to encrypt this block device and an ext4 file system is installed above this. The encrypted filesystem is then made available and mounted on the database server.  The postgresql is configured to store its data in the encrypted area.

A limitation is that the system cannot be started without manual intervention.  When the system is booted, the lxc containers are not started by default.  First the encrypted disk needs to be opened by providing a password.  After this the containers can be started.  This is achieved by logging in via ssh and running a startup script.

Details of the encryption setup are decribed in [encryption.md].

### Networking
The containers are all linked to one another on an internal virtual network.

The host machine runs a host-based firewall, ufw, which allows tcp traffic to the ssh server only (on port 822).  ssh connections are rate limited, meaning it will deny connections from an IP address that has attempted to initiate 6 or more connections in the last 30 seconds.  In addition the firewall is configured to forward all incomng traffic on ports 80 and 443 through to the web proxy container.

Firewall configuration is described in [firewall.md].

### Proxy
The proxy server container is running apache 2.4.  All http traffic is redirected to https.  The apache server is configured with a certificate from letsencrypt.org.  The ssl configuration is rated as A+ by ssllabs.com (2017-12-01).

The proxy server setup is described in [proxy.md].

### Application server
The application server is running the ubuntu standard package of tomcat8 on top of oracle java 8.

### Db server
