
To collect metrics of the total CPU and memory usage in a Kubernetes cluster on a daily basis, you can use Prometheus and Grafana.

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

2.  Set up a Grafana dashboard to display the CPU and memory usage metrics. You can create a new dashboard and add two graph panels to it, one for CPU usage and one for memory usage. In the query editor for each panel, you can use the following PromQL query to retrieve the CPU and memory usage metrics:

```
sum(container_cpu_usage_seconds_total{namespace="<namespace>"}) by (pod)

sum(container_memory_usage_bytes{namespace="<namespace>"}) by (pod)

```

Replace `<namespace>` with the namespace you want to monitor.

These queries will return the total CPU and memory usage, respectively, of all containers in the specified namespace.

3.  Set up a cron job to run once a day and export the CPU and memory usage metrics to a CSV file. You can use the Prometheus API to retrieve the CPU and memory usage metrics and export them to a CSV file. Here is an example Python script that retrieves the metrics and exports them to a CSV file:

```
import requests
import csv
import datetime

url = 'http://prometheus:9090/api/v1/query'
cpu_query = 'sum(container_cpu_usage_seconds_total{namespace="<namespace>"}) by (pod)'
mem_query = 'sum(container_memory_usage_bytes{namespace="<namespace>"}) by (pod)'

cpu_response = requests.get(url, params={'query': cpu_query})
mem_response = requests.get(url, params={'query': mem_query})

cpu_data = cpu_response.json()['data']['result']
mem_data = mem_response.json()['data']['result']

rows = [[datetime.datetime.now().strftime("%Y-%m-%d"), r['metric']['pod'], r['value'][1]] for r in cpu_data] + [[datetime.datetime.now().strftime("%Y-%m-%d"), r['metric']['pod'], r['value'][1]] for r in mem_data]

with open('k8s_cluster_usage.csv', 'a') as f:
    writer = csv.writer(f)
    writer.writerows(rows)

```

This script retrieves the CPU and memory usage metrics using the PromQL queries and exports them to a CSV file named 'k8s_cluster_usage.csv'. The script also adds the current date to each row of data.

You can schedule this script to run once a day using a cron job or a similar scheduling tool.

By following these steps, you can collect metrics of the total CPU and memory usage in a Kubernetes cluster on a daily basis and visualize them using Grafana. Additionally, you can export the metrics to a CSV file for further analysis.