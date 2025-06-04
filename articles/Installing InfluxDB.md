![Alt](/articles/assets/InfluxDB_280x250_q85.png) 

# Installing InfluxDB
I'm now at the stage where I want to ensure that I can monitor my home lab.
Installing InfluxDB is a good initial stage which will allow me to gather, initially, metrix from proxmox which we can later view in Grafana.

The documentation for InfluxDB can be found [here](https://docs.influxdata.com/).

This article is partly based on Christian Lempa's [work](https://github.com/ChristianLempa) and [videos](https://www.youtube.com/@christianlempa).

I will install InfluxDB on my Docker server as it has more storage than my k3s cluster VMs.

InfluxDB 3 is a little bit to new - it was only released a few months ago.  I will use InfluxDB 2.x.

I will use Christian's compose.yaml as my starting point for the deployment:
![abc](https://raw.githubusercontent.com/ChristianLempa/boilerplates/refs/heads/main/docker-compose/influxdb/compose.yaml)



