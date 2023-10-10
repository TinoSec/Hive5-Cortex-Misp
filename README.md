# Hive5CortexMisp
Integrate Hive5, Cortex & Misp for your Open-Source SOC.

If you want to develop your own Open-Source SOC, this guide will help you, files included here are tested for Ubuntu 22.04 enviroments. 

The original installation guide can be found https://docs.strangebee.com/thehive/setup/installation/step-by-step-guide/

# Install required packages TheHive-5

```bash
sudo apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl software-properties-common python3-pip lsb-release
```

# Install Java Virtual Machine - Amazon Corretto 11

```bash
wget -O- https://apt.corretto.aws/corretto.key | sudo apt-key add -
sudo add-apt-repository 'deb https://apt.corretto.aws stable main'
sudo apt-get update
sudo apt-get install -y java-11-amazon-corretto-jdk
echo JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto" | sudo tee -a /etc/environment 
export JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto"
```

# Install Cassandra 4
```bash
wget -qO -  https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor  -o /usr/share/keyrings/cassandra-archive.gpg
echo "deb [signed-by=/usr/share/keyrings/cassandra-archive.gpg] https://debian.cassandra.apache.org 40x main" |  sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
sudo apt update
sudo apt install cassandra
sudo rm -rf /var/lib/cassandra/*
```
Download cassandra.yaml to */etc/cassandra/*

[cassandra.yaml](https://github.com/TinoSec/Hive5-Cortex-Misp/blob/main/cassandra.yaml)

Edit the file, change *"192.168.1.2"* for your-sever-ip.

Once done, start service and Cassandra DB

```bash
sudo systemctl enable cassandra REVISAR NECESIDAD
sudo systemctl start cassandra
cassandra -R &
```
Try connection with Cassandra (default pass: cassandra)

```bash
cqlsh -u cassandra <IP ADDRESS> -e "SELECT table_name,gc_grace_seconds FROM system_schema.tables WHERE keyspace_name='thehive'"
```

Example:

*cqlsh -u cassandra 192.168.1.2 -e "SELECT table_name,gc_grace_seconds FROM system_schema.tables WHERE keyspace_name='thehive'"*


# Install Elasticsearch 7

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch |  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" |  sudo tee /etc/apt/sources.list.d/elastic-7.x.list 
sudo apt update
sudo apt install elasticsearch
```
Download elasticsearch.yml to */etc/elasticsearch/* & edit the file, change *192.168.1.2* for your-sever-ip

[elasticsearch.yml](https://github.com/TinoSec/Hive5-Cortex-Misp/blob/main/elasticsearch.yml)

Download jvm.options to */etc/elasticsearch/jvm.options.d/* 

XXXXXXXX jvm.options FIle


**This last file is used to define your ram consumption**

```bash
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
sudo systemctl stop elasticsearch
sudo rm -rf /var/lib/elasticsearch/*
sudo systemctl start elasticsearch
```

# Install TheHive-5
```bash
apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl software-properties-common python3-pip lsb-release
wget -O- https://archives.strangebee.com/keys/strangebee.gpg | sudo gpg --dearmor -o /usr/share/keyrings/strangebee-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/strangebee-archive-keyring.gpg] https://deb.strangebee.com thehive-5.2 main' | sudo tee -a /etc/apt/sources.list.d/strangebee.list
sudo apt-get update
sudo apt-get install -y thehive
sudo mkdir -p /opt/thp/thehive/files
chown -R thehive:thehive /opt/thp/thehive/files
```
Download application.conf to */etc/thehive/* & edit the file, change *192.168.1.2* for your-sever-ip

[application.conf (hive)](https://github.com/TinoSec/Hive5-Cortex-Misp/blob/main/application.conf(hive))


```bash
sudo systemctl enable thehive
sudo systemctl start thehive
```
Login TheHive-5 http://your-server-ip:9000/
*user: admin@thehive.local *
*pass: secret *

# Install Cortex

```bash
apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl  software-properties-common python3-pip lsb-release
wget -O- "https://raw.githubusercontent.com/TheHive-Project/Cortex/master/PGP-PUBLIC-KEY"  | sudo apt-key add -
wget -qO- https://raw.githubusercontent.com/TheHive-Project/Cortex/master/PGP-PUBLIC-KEY |  sudo gpg --dearmor -o /usr/share/keyrings/thehive-project.gpg
echo 'deb https://deb.thehive-project.org release main' | sudo tee -a /etc/apt/sources.list.d/thehive-project.list
apt update
apt install cortex
sudo systemctl enable cortex
sudo systemctl start cortex
```
In order to install analyzers and responders locally for Cortex, it is necssary to create an enviroment with python, this avoids errors in the installation of dependencies. This Will take a while

```bash
sudo apt install python3-venv
python3 -m venv /etc/cortex/cortex-env
source /etc/cortex/cortex-env/bin/activate
sudo apt remove python3-pip
sudo apt install python3-pip
sudo apt install build-essential libssl-dev libffi-dev python3-dev
pip3 install --upgrade pip
sudo apt install -y --no-install-recommends python3-pip python3-dev ssdeep libfuzzy-dev libfuzzy2 libimage-exiftool-perl libmagic1 build-essential git libssl-dev
sudo pip3 install -U pip setuptools
git clone https://github.com/TheHive-Project/Cortex-Analyzers /opt/Cortex-Analyzers
chown -R cortex:cortex /opt/Cortex-Analyzers
cd /opt
for I in $(find Cortex-Analyzers -name 'requirements.txt'); do sudo -H pip3 install -r $I || true; done
```
Once finished, restart Cortex service

```bash
sudo systemctl restart cortex
```
In order to configure analyzers to run locally use this configuration guide as example

[application.conf (cortex)](https://github.com/TinoSec/Hive5-Cortex-Misp/blob/main/application.conf(cortex))

Login Cortex http://your-server-ip:9001/




