# Install Neo4J on Ubuntu

## Download Mobaxterm
 - https://download.mobatek.net/2032020060430358/MobaXterm_Portable_v20.3.zip

## SSH to the Cloud Machine
 - Extract MobaXterm_Portable_v20.3.zip
 - Run MobaXterm_Personal_20.3.exe
 - Take your VM IP address and credentials
 - Create SSH session to the cloud machine

## Install Ubuntu various utilities
```
sudo apt update
sudo apt -y install tree
sudo apt -y install unzip
sudo apt -y install htop
sudo apt -y install jq
```

## Install Java
```
sudo add-apt-repository universe
sudo add-apt-repository -y ppa:openjdk-r/ppa
sudo apt update
```

## Install Ne04J
```
sudo wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo apt-key add -
echo 'deb https://debian.neo4j.com stable 4.1' | sudo tee -a /etc/apt/sources.list.d/neo4j.list
sudo apt update
sudo apt install -y neo4j-enterprise=1:4.1.3
ulimit -n 60000
```

## Start Neo4J service
```
sudo systemctl enable neo4j
sudo systemctl start neo4j
sudo systemctl status neo4j
```

## Test if Neo4J is working
```
curl localhost:7474
```

## Neo4J Logs
```
sudo journalctl -e -u neo4j
```

## Enable below Ports on Firewall/ Inbound Port Rules
 - 7687,5000,6000,7000,7688,2003,2004,3637,5005,7474

## Allow All IP Addresses to access Neo4J
```
sudo vim /etc/neo4j/neo4j.conf
```
	- Uncomment:
		- dbms.default_listen_address=0.0.0.0

## Restart Neo4J
sudo systemctl restart neo4j

## Open Neo4J from your local computer:
 - Visit
	- http://<dns-name>:7474/
		- Default Username: neo4j
		- Default Password: neo4j