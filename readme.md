
Spin K8s cluster using GKE

===============================================================
# On K8s cluster install prometheus using helm (pkg installer in k8s)

On master node

 Steps:

# Add helm repo
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# update helm repo
  helm repo update

# Install helm
  helm install prometheus prometheus-community/prometheus

# Check prometheus objects created
  kubectl get all

# 3 exporter pods get created , which prometheus polls to pull metrcis from
   - prometheus-kube-state-metrics = collects cluster metrics
   - prometheus-node-exporter-gsf = collects worker1 OS/Kernel level metrics (Daemonset)
   - prometehus-node-exporter-teg = collects worker2 OS/Kernel level metrics (Daemonset)

# Expose prometheus service to access it from browser
     kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext

# Access prometheus using
   <node-server-IP>:9090
   
# Prometheus Alerts 	    
		  - write alert.rules.yml file 
		        - write alert definition with alert severity (like moderate / critical / warning)
				      - high cpu utilization 
					     - high memory utilization 
					     - high http error rate 		  
		  - update prometheus.yml file to include this alert rule file 	  
		  - configure prometheus Alert Manager to receive alerts from Prometheus and route them to desired contacts (email/slack) 
		  - restart prometheus service 

# Fire promQL queries for getting state of system 
  - CPU utilization
  - Disk usage
  - RAM utilization 
  - Number of time init container restarted 
  - Number of configmaps in cluster

============================================================================
# On K8s cluster install grafana using helm (pkg installer in k8s)

On master node

 Steps:

# Add helm repo
 helm repo add grafana https://grafana.github.io/helm-charts

# Update helm repo
 helm repo update

# Install helm
 helm install grafana grafana/grafana

# Login creds
 username = admin
 password =

 kubectl get secret --namespace default grafana -o jsonpath = "{.data.admin-password}" | base64 --decode;echo

# Expose grafana service to nodeport
 kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext

# Access grafana from browser
 <Node-server-IP>:3000

# Add data source as prometheus
  Add prometheus URL - <Node-server-IP>:9090

Create alerting rules
   - Add prometheus as source 
	  - Go to alert tab and create alert 
	  - define alert rule 
	  - cofigure notification setting to route alerts to desired contacts 
    
===========================================================================
# Implement Custom metrics in Node.js Application

# Instrumentation
Refers to process of adding code in application to collect metrics/logs and traces

# **Implement Custom Metrics in Node.js Application**
- **Prometheus Metrics with prom-client**: Integrates Prometheus for monitoring HTTP requests using the prom-client library:
    - `http_requests_total`: counter
    - `http_request_duration_seconds`: histogram. Here we have different buckets for different request duration.
    - `http_request_duration_summary_seconds`: summary
    - `node_gauge_example`: gauge for tracking async task duration / CPU utilization

- In serviceMonitor.yml we define service discovery
  i.e. which application targets prometheus will scarpe for collecting custom metrics

### Basic Routes: In Nodejs application 
- `/metrics`: Exposes Prometheus metrics endpoint.
- `/crash`: Simulates a server crash by exiting the process.
- `/call-service-b`: To call service b & receive data from service b
- `/logs`: Generates logs using the custom logging function.

# Steps:
1. Dockerize and push images to dockerhub 
docker build -t adititdkr1311/demoservice-a:v1 application/service-a/ 
docker build -t adititdkr1311/demoservice-b:v1 application/service-b/ 

2. Apply manisfest files to the cluster 
kubectl create ns dev
kubectl apply -k kubernetes-manifest/

3. Test the endpoints 
<service-a-DNS-Name>/metrics     #### to get the application metrics
<service-a-DNS-Name>/logs     #### to get the application logs


4. Configure alertmanager 
# Triggers alerts for 
    - HighCpuUsage : Triggers a warning alert if the average CPU usage across instances exceeds 50% for more than 5 minutes.
    - PodRestart : Triggers a critical alert immediately if any pod restarts more than 2 times.

# We write 2 manifest files 
   - alerts.yml : Here we define the expression when alerts to be fired
   - alertmanagerconfig.yml : Here we define where alerts should be sent to (slack/gmail)    

kubectl apply -k alerts-alertmanager-servicemonitor-manifest/

5. Testing alerts 
# Manually crash the container 2 times calling endpoint
<service-a-DNS-Name>/crash 

# Alerts will be fired to the gmail account 



