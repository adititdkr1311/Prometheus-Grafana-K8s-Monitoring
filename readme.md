
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

# 3 pods get created
   - prometheus-kube-state-metrics = collects cluster metrics
   - prometheus-node-exporter-gsf = collects worker1 OS/Kernel level metrics
   - prometehus-node-exporter-teg = collects worker2 OS/Kernel level metrics

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

# Create alerting rules 
	   - Add prometheus as source 
		  - Go to alert tab and create alert 
		  - define alert rule 
		  - cofigure notification setting to route alerts to desired contacts 
