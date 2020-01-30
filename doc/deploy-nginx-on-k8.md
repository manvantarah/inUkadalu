# Deploy the Nginx to k8 node

### Before Proceeding

1. Must have Kadalu operator, Storage, Namespace.
2. Nginx running in local host.

## Assign some resource quota

Create a file name manage-quota.yaml, and add these content into the file

	apiVersion: kadalu-operator.storage/v1alpha1
	kind: KadaluStorage
	metadata:
  		name: mem-cpu-quota
  		namespace: kadalu
	spec:
		hard:
    		requests.cpu: "2"
    		requests.memory: 4Gi
    		limits.cpu: "4"
    		limits.memory: 8Gi

Create the manage-quota using the YAML

	$ sudo kubectl create -f manage-quota.yaml

Verify the file has deployed to namespace

	$ sudo kubectl describe ns kadalu

### Expected O/P:

	Name:         kadalu
	Labels:       <none>
	Annotations:  <none>
	Status:       Active
	Resource Quotas
		Name:            mem-cpu-quota
		Resource         Used  Hard
		--------         ---   ---
		limits.cpu       1     2
		limits.memory    2Gi   8Gi
		requests.cpu     500m  2
		requests.memory  1Gi   2Gi
		No LimitRange resource.

## Create a Deployment

Use the public Nginx image for this deployment

### Note: Can use any public image like prometheus,grafana etc.

Create a file named deployment.yaml and copy the following YAML to the file

	apiVersion: apps/v1
	kind: Deployment
	metadata:
		name: nginx
		labels:
    		app: nginx
		namespace: kadalu
		annotations:
    		monitoring: "true"
	spec:
		replicas: 1
		selector:
    		matchLabels:
      			app: nginx
		template:
    		metadata:
      		labels:
        		app: nginx
    	spec:
      		containers:
      		- image: nginx
        	  name: nginx
        	  ports:
        		- containerPort: 80
        		resources:
          			limits:
            			memory: "2Gi"
            			cpu: "1000m"
          			requests: 
            			memory: "1Gi"
            			cpu: "500m" 

Create the deployment using kubectl
	
	$ sudo kubectl create -f  deployment.yaml

Verify the file has deployed or not

	$ sudo kubectl get deployments -n kadalu

Check the deployments by describing the deployment in YAML format using the kubectl command

	$ sudo kubectl get deployment -n kadalu --output yaml

Create a service for the deployment by copying this code into the file

	apiVersion: v1
	kind: Service
	metadata:
		labels:
    		app: nginx
		name: nginx
		namespace: kadalu
	spec:
		ports:
		- nodePort: 30001 # port number must be from 30000-45000 for Nginx services
    	  port: 80
    	  protocol: TCP
          targetPort: 80
		selector:
    		app: nginx
    	type: NodePort

Create the service using kubectl command

	$ sudo kubectl create -f service.yaml

View the service created using kubectl command

	$ sudo kubectl get services -n kadalu

### Now the Nginx service will be access on kuberenetes node ip

	$ sudo kubectl get nodes -o wide

### http://Kube-node-ip:30001/