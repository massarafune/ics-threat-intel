# ics-threat-intel
This is simple Proof of Concept repository. The main purpose is to build threat intel platform for ICS environment.  

This repo contains Vagrantfile and ELK configuration.  
Vagrantfile automatically runs shell scripts to install ELK stack and Node.  

Please note that this repo DOES NOT include React front end application, which can be found in different repo as well as signature based attack detection python script.  

# Installation
## Install Virtual Box and Vagrant
Make sure you have followings in your host machine, or use your *nix virtual machine:
- Vagrant
- VirtualBox

1. Clone this repo
```bash
git clone https://github.com/massarafune/ics-threat-intel.git
```
2. Create virtual machine then login to the box
```bash
vagrant up
vagrant ssh
```

## Install React App
In side the vagrant box/virtual machine run the followings to install web application
```bash
cd ~
git clone https://github.com/massarafune/ICS-threat-hunting-reactapp.git
cd ICS-threat-hunting-reactapp
npm install
npm start
```
This will start React app.  
But you can build this application to serve static files, which can be accessed via apache web server at port 80
```bash
npm run build
```

More detailed instruction can be found in the repository [https://github.com/massarafune/ICS-threat-hunting-reactapp.git](https://github.com/massarafune/ICS-threat-hunting-reactapp.git)  

## Configure ELK stack
### Elasticsearch
#### /etc/elasticsearch/elasticsearch.yml
Modify yml file as follows:
```
cluster.name: ICS-threat-hunting
node.name: node-1
network.host: "localhost"
http.port: 9200
discovery.seed_hosts: ["127.0.0.1", "[::1]"]
cluster.initial_master_nodes: ["node-1"]
```

### Kibana
You need to install Kibana separately because Kibana itself is not necessary module for the purpose of research.  
It is not required to install and use Kibana for this project but it can be useful to review data in ElasticSearch.  

#### Install Kibana
```bash
sudo apt install kibana
```

#### /etc/kibana/kibana.yml
```
server.port: 5601
server.host: "192.168.33.30"
server.name: "ics-threat-hunting"
elasticsearch.hosts: ["http://localhost:9200"]
logging.dest: /var/log/kibana/kibana.log
```

## Start services
Once you set up all the above components, you are ready to run all tools.  

```bash
# Let Elasticsearch start when machine boots up
sudo systemctl enable elasticsearch
# start elasticsearch
sudo systemctl start elasticsearch

# Let Kibana start when machine boots up
sudo systemctl enable kibana
# start kibana
sudo systemctl start kibana

# Let filebeat....
sudo systemctl enable filebeat
# start filebeat
sudo systemctl start filebeat

# then logstash
sudo systemctl enable logstash
# sudo systemctl start logstash

```

Keep in mind that ELK stack requires some time to boot up (around a couple of minutes to fully functional)  

## Services and ports
| Service | Port |
----|----
| Elasticsearch | 9200 | 
| Kibana | 5601 |
| Logstash | 9600 |


Everytime you restart/boot machine, you need to start Node and React manually, but ELK will start automatically

---

## Known Issues
We are aware that there is a npm known issue.  
If you have npm errors when you run `npm install` or `npm start` or `npm run server`, you can run below command to fix this.  

```bash
echo fs.inotify.max_user_watches= 65536 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

If you have some errors due to Node versions, you can install nvm (Node version manager) to switch Node easily  
```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash

# install node version 15.x
nvm install 15
```
