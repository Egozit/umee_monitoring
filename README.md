# Umee node monitoring tool

To monitor you node your should install and configure:
* [InfluxDB](https://www.influxdata.com/products/influxdb/)
* [Grafana](https://grafana.com/)

Advantages  of using our free service:
* Our monitoring service is working on dedicated server (24/7 online)
* No need to install database  (InfluxDB)
* No need to install and configure  Grafana Dashboard
* On Grafana dashboard you will find all necessary metrics of your node (we use this monitoring service by ourselves, so we've configured dashboard properly)

## Community dashboard
Check out our free community dashboard: [Dashboard link](http://95.216.76.51:3000/d/C0q3X5P7k/umee-mainnet-monitoring?orgId=1&refresh=30s&var-datasource=InfluxDB&var-inter=2m&var-server=Egozit)

## Manual installation of telegraf and monitoring script

Install telegraf
```
sudo apt update
sudo apt -y install curl jq bc

# install telegraf
sudo cat <<EOF | sudo tee /etc/apt/sources.list.d/influxdata.list
deb https://repos.influxdata.com/ubuntu bionic stable
EOF
sudo curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -

sudo apt update
sudo apt -y install telegraf

sudo systemctl enable --now telegraf
sudo systemctl is-enabled telegraf

# make the telegraf user sudo and adm to be able to execute scripts as umee user
sudo adduser telegraf sudo
sudo adduser telegraf adm
sudo -- bash -c 'echo "telegraf ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers'
```
You can check telegram service status:
```
sudo systemctl status telegraf
```
Status can be not ok with default Telegraf's config. Next steps will fix it.

Clone this project repo and copy variable script template
```
git clone https://github.com/Egozit/umee_monitoring
cd umee_monitoring
nano variables.sh
```

Insert your parameters to **variables.sh**:
* full path to umeed binary to COS_BIN_NAME ( check ```which umeed```)
* node PRC port to COS_PORT_RPC ( check in file ```path_to_umee_node_config/config/config.toml```)
* node API port to COS_PORT_API ( check in file ```path_to_umee_node_config/config/app.toml```)
* node validator address to COS_VALOPER ( like ```umeevaloper********```)
* validator wallet address to COS_WALADDR
* peggo bridge Umee wallet address (orchestrator) to COS_BR_ADDR
* peggo bridge Etherium wallet address to ETH_BR_ADDR
* way to Etherium node RPC to ETH_NODE_RPC ( like ```http://localhost:8545``` )
Save changes in variables.sh and enable execution permissions:

```
chmod +x monitor.sh variables.sh monitor_bal.sh
```

Edit telegraf configuration
```
sudo mv /etc/telegraf/telegraf.conf /etc/telegraf/telegraf.conf.orig
sudo mv telegraf.conf /etc/telegraf/telegraf.conf
sudo nano /etc/telegraf/telegraf.conf
```
Set you name to identify yourself in grafana dashboard and check correctness of the path to your monitor.sh and monitor_bal.sh files and username your validator runs at. Correct these two settings in the configuration file:
```
[agent]
  hostname = "YOUR_MONIKER/SERVER_NAME" # set this to a name you want to identify your node in the grafana dashboard
  flush_interval = "15s"
  interval = "15s"
...
...
...
[[inputs.exec]]
  commands = ["sudo su -c /root/umee_monitoring/monitor.sh -s /bin/bash root"] # change home and username to the useraccount your validator runs at
  interval = "15s"
  timeout = "5s"
  data_format = "influx"
  data_type = "integer"
[[inputs.exec]]
  commands = ["sudo su -c /root/umee_monitoring/monitor_bal.sh -s /bin/bash root"] 
  interval = "15s"
  timeout = "5s"
  data_format = "influx"
  data_type = "integer"
```

```
systemctl daemon-reload
systemctl restart telegraf
```
