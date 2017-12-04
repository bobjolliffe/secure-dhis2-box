# Installing OSSEC host based intrusion detection system

Please ignore this document for now.  ossec is not installed.  Considering installing 
wazu instead ....

## Setting up repository
On the host:
wget -q -O - https://updates.atomicorp.com/installers/atomic | sudo bash
sudo apt-update
# Installing the server
sudo apt install --no-install-recommends ossec-hids-server 
