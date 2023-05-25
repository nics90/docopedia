
To collect metrics of the Kubernetes cluster uptime on a daily basis, you can use a combination of Kubernetes API, Prometheus, and Grafana.

Here are the steps you can follow:

1.  Configure Prometheus to scrape metrics from the Kubernetes API server. You can do this by adding the following configuration to your Prometheus configuration file:


```
scrape_configs:
  - job_name: 'kubernetes-apiserver'
    kubernetes_sd_configs:
    - role: endpoints
    relabel_configs:
    - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
      separator: ;
      target_label: __metrics_path__
      replacement: /apis/metrics.k8s.io/v1beta1;

```

2.  Set up a Grafana dashboard to display the Kubernetes cluster uptime metrics. You can create a new dashboard and add a graph panel to it. In the query editor, you can use the following PromQL query to retrieve the Kubernetes cluster uptime metric:

```
sum(kube_apiserver_up{job="kubernetes-apiserver"}) by (instance)

```

This query will return the total uptime of the Kubernetes API server.

3.  Set up a Prometheus alert to trigger when the Kubernetes cluster uptime drops below a certain threshold. You can create a new alert rule and use the following PromQL query:

```
kube_apiserver_up{job="kubernetes-apiserver"} == 0

```

This query will return a non-zero value if the Kubernetes API server is down. You can set a threshold for this alert, such as triggering when the uptime drops below 99%.

4.  Set up a cron job to run once a day and export the Kubernetes cluster uptime metrics to a CSV file. You can use the Prometheus API to retrieve the Kubernetes cluster uptime metrics and export them to a CSV file. Here is an example Python script that retrieves the metrics and exports them to a CSV file:

```
import requests
import csv
import datetime

url = 'http://prometheus:9090/api/v1/query'
query = 'sum(kube_apiserver_up{job="kubernetes-apiserver"}) by (instance)'

response = requests.get(url, params={'query': query})

data = response.json()['data']['result']
rows = [[datetime.datetime.now().strftime("%Y-%m-%d"), r['metric']['instance'], r['value'][1]] for r in data]

with open('k8s_cluster_uptime.csv', 'a') as f:
    writer = csv.writer(f)
    writer.writerows(rows)

```

This script retrieves the Kubernetes cluster uptime metrics using the PromQL query and exports them to a CSV file named 'k8s_cluster_uptime.csv'. The script also adds the current date to each row of data.

You can schedule this script to run once a day using a cron job or a similar scheduling tool.

By following these steps, you can collect metrics of the Kubernetes cluster uptime on a daily basis and visualize them using Grafana. Additionally, you can set up alerts to notify you when the Kubernetes API server goes down and export the metrics to a CSV file for further analysis.