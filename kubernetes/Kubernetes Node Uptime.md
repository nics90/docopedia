
To collect the metrics of Node uptime in a Kubernetes cluster on a daily basis, you can use a combination of Kubernetes API, Prometheus, and Grafana.

Here are the steps you can follow:

1.  Configure Prometheus to scrape metrics from the Kubernetes API server. You can do this by adding the following configuration to your Prometheus configuration file:

```
scrape_configs:
  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
    - role: node
    relabel_configs:
    - source_labels: [__meta_kubernetes_node_name]
      target_label: node
    - source_labels: [__address__]
      target_label: __metrics_path__
      replacement: /api/v1/nodes/$1/proxy/metrics

```

2.  Set up a Grafana dashboard to display the Node uptime metrics. You can create a new dashboard and add a graph panel to it. In the query editor, you can use the following PromQL query to retrieve the Node uptime metric:

```
sum(up{job="kubernetes-nodes"}) by (node)

```

This query will return the total uptime for each Node in the Kubernetes cluster.

3.  Set up a Prometheus alert to trigger when the Node uptime drops below a certain threshold. You can create a new alert rule and use the following PromQL query:

```
sum(up{job="kubernetes-nodes"} == 0) by (node)
```

This query will return the number of Nodes that are currently down. You can set a threshold for this alert, such as triggering when more than 10% of Nodes are down.

4.  Set up a cron job to run once a day and export the Node uptime metrics to a CSV file. You can use the Prometheus API to retrieve the Node uptime metrics and export them to a CSV file. Here is an example Python script that retrieves the metrics and exports them to a CSV file:

```
import requests
import csv
import datetime

url = 'http://prometheus:9090/api/v1/query'
query = 'sum(up{job="kubernetes-nodes"}) by (node)'

response = requests.get(url, params={'query': query})

data = response.json()['data']['result']
rows = [[datetime.datetime.now().strftime("%Y-%m-%d"), r['metric']['node'], r['value'][1]] for r in data]

with open('node_uptime.csv', 'a') as f:
    writer = csv.writer(f)
    writer.writerows(rows)
```

This script retrieves the Node uptime metrics using the PromQL query and exports them to a CSV file named 'node_uptime.csv'. The script also adds the current date to each row of data.

You can schedule this script to run once a day using a cron job or a similar scheduling tool.

By following these steps, you can collect the metrics of Node uptime in a Kubernetes cluster on a daily basis and visualize them using Grafana. Additionally, you can set up alerts to notify you when Nodes go down and export the metrics to a CSV file for further analysis.