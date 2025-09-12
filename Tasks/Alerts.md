# CPU Usage

1.  expr: 100 - cpu_usage_idle{job="csttelegraf",cpu="cpu-total",component!="paytm-digital-cst-opensip"} > 70
2.  expr: (100 - avg_over_time (cpu_usage_idle{cpu="cpu-total", techteam="digitalcst"}\[15m\])) > 95
3.  expr: avg_over_time(cpu_usage_steal{techteam="digitalcst"}\[10m\]) > 10
4.  expr: avg_over_time(cpu_usage_steal{techteam="digitalcst"}\[10m\]) > 50
5.  expr: 100 \* sum(rate(container_cpu_usage_seconds_total{container!="",k8scluster="paytm-digital-prod-cst"}\[1m\])) by (pod,container) / sum(container_spec_cpu_quota/container_spec_cpu_period{}) by (pod,container) > 75
6.  expr: 100 - cpu_usage_idle{component="cst-ticket-elasticsearch-cluster",cpu="cpu-total"} > 20
7.  expr: cpu_usage_active{job="azure-machine-metrics", cpu="cpu-total"} > 20
8.  expr: cpu_usage_active{job="azure-machine-metrics", cpu!="cpu-total"} > 95

&nbsp;

# Alerts not triggered

100 - cpu_usage_idle{component="cst-ticket-elasticsearch-cluster",cpu="cpu-total", instance="10.4.51.236:9273"}

100 - cpu_usage_idle{component="cst-ticket-elasticsearch-cluster",cpu="cpu-total", instance="10.4.52.200:9273"}

&nbsp;

# 100% CPU usage

100 - cpu_usage_idle{job="csttelegraf",cpu="cpu-total",component!="paytm-digital-cst-opensip",instance="10.4.53.65:9273"}

&nbsp;

# Miss-spelled tags 

10.4.53.51,  10.4.53.44 - techteam tag

10.4.48.15,  10.4.51.110,  10.4.51.202,  10.4.52.29

&nbsp;

# No instance found

10.4.50.212

10.4.50.37