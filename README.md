# ics-threat-intel
Threat intel platform for ICS

This repo contains Vagrantfile and backend Nodejs API server.  
Vagrantfile automatically runs shell scripts to install ELK stack and Node.  

Please note that this repo DOES NOT include React front end, which can be found in different repo.

# Installation
## Install Virtual Box and Vagrant
Make sure you meet prerequisites
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
```bash
cd ~
git clone https://github.com/mistersiddd/react-fileupload.git
cd react-fileupload
npm install
npm start
```
This will start React app which runs on port 3000

You can access this web app from your Host computer (Windows/Mac/Linux) at http://192.168.33.30:3000 

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

### Filebeat
#### /etc/filebeat/filebeat.yml
```
- type: filestream
    enabled: true
    paths:
        - /vagrant/Webapp/public/swat/*.csv
```
Note the path will be a directory where you save SWAT csv files

Other than the above, keep all default

### Kibana
You need to install Kibana separately because Kibana itself is not necessary module for the purpose of research.  
However you can use Kibana for debuging or simply query data in a simple way.  

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

# configure Node backend
cd /vagrant/WebApp
npm install
npm run server
```

Keep in mind that ELK stack requires some time to boot up (around a couple of minutes to fully functional)  

## Services and ports
| Service | Port | Accessible from Host? |
----|----|----
| Elasticsearch | 9200 | False |
| Kibana | 5601 | True |
| Logstash | 9600 | False |
| Node API | 4500 | True |
| React | 3000 | True |

Everytime you restart/boot machine, you need to start Node and React manually, but ELK will start automatically

---

## Known Issues
We are aware that there is a npm known issue.  
If you have npm errors when you run `npm install` or `npm start` or `npm run server`, you can run below command to fix this.  

```bash
echo fs.inotify.max_user_watches= 65536 | sudo tee -a /etc/sysctl.conf
```

If you have some errors due to Node versions, you can install nvm (Node version manager) to switch Node easily  
```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash

# install node version 15.x
nvm install 15
```
