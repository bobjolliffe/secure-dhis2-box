# Configuring mail

It is important for our server to be able to send out alerts.  The simplest way to achieve this in a standalone environment is to configure a mail transport agent.  For this purpose we will use exim4.

# Installation
sudo apt install exim4
sudo dpkg-reconfigure exim4-config

Select internet connected host, hostname=slart.dhis2.org
Add interface 10.103.19.1 to list.
Allow forwarding of mail from 10.103.19.0/24

Add to /etc/aliases:
root: admins
admins: bobjolliffe@gmail.com

sudo newaliases

send a test message :  mail bobjolliffe@gmail.com
check log: tail /var/log/exim4/mainlog
check mail (you might need to look in spam and possibly create a filter)

