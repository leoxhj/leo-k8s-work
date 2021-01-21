Project name: `fastapi-project-template`

p.s.
the time consuming part is: to build up image since need doing apt-get update && apt-get upgrade -y, it really sucks in domestic:(, need to save up time, so don't have too much time to hack the debian config file to point to domestic apt.sources to making this faster.

what I'm going to do for the challenge:
* build up image use docker build:
	```bash
	# update project Dockerfile with demostic pypi source
	vi Dockerfile
	#the line modified
	RUN pip install -i https://mirrors.aliyun.com/pypi/simple/ --trusted-host=mirrors.aliyun.com --upgrade pip \ 
	 pip install -i https://mirrors.aliyun.com/pypi/simple/ --trusted-host=mirrors.aliyun.com pytest psycopg2 cython && pip install poetry && poetry install

	# build Docker image: 
	docker build -t leo-python:3.8-buster .
	```
* write all those kubernetes yaml manifest files, including deployment, service, configmaps, namespace, [optional: serviceacount, clusterrolebinding], hpa... 
* put all together manifest files into kustomization.yaml, then use kustomize tool to deploy to k8s cluster, alternatively, I can deploy manually one by one by using kubectl or even by shell scripts
* set up those `env` in deployment to connect to postgresql database.
* verify and debug during all resources deployment and exceptions happened.
* use kubectl logs pod, kubectl describe pod ...  to dig into issues. 
* verify from `curl` command to test the application from its http restpai to see is ok or not.

# Tasks:
## Step One:
folder name: `leo-k8s-task`

## Step Two:

Requirements:

1. the application supposed to be having zero downtime when performing upgrades.  
`A: Deployment workload will performing rolling upgrades which means zero downtime, the new replicaset will be created and once all new pods in new replicaset is ready, the pods in old replicaset will be deleted.`
2. the application supposed to be auto-scaled based on its CPU/memory usage.  
`A: here uses a hpa.yaml file to do so`
3. the application supposed to be auto restarted when crashed.  
`yes, deployment workload, the replicaset will take care of when instance is down, to make sure the expected pods are same in which defined in deployment.`
4. the application supposed to be out-of-serving when it's unhealthy.  
`dont quite sure of this, if some node is failure or due to regular maintenance, then the pod should never get up on this node if toleration has set for same host. try to cordon node or drain in this situation`
5. the application supposed to be exposed to the outside of the cluster.  
`here uses service with type: LoadBalancer to expose service outside cluster, like GCP and other cloud which providing load balancing service.`
6. the configuration of the application supposed to be separated from the application object resource.  
`use configmap and then VolumeMount , Volume(point the configmaps names) in pod`
7. the replicas of the application object should be as much as possible separated from each other on the cluster node.  
```bash
# use affirnity, podAntiAffirnity, like below sample which we have done in our project:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: topolvm-test
            topologyKey: kubernetes.io/hostname
```	    
8. (optional) the application supposed to be automatically catch-up with the new changes in its configuration without deployment.  
9. it should come with a README file documenting how to deploy it on a different cluster

## NOTES:
The application required configuration(environment variables) are:
```bash
DB_HOST=localhost
DB_USER=leotest
DB_PASS=leotest123
DB_NAME=leotest
HTTP_AUTH_KEY=dummy_key
```

## Step Three:
deploy the above resources to the cluster and make sure they work correctly, in the end, output their status as long as the command you are using to the file resources-output.txt.

requirements:
1. the commands of getting k8s cluster information.  
`kubectl cluster-info`
2. the commands of deploying the above resource manifests.  
```bash
# deploy use kustomize tools, <manifests directory>
kubectl kustomize manifests
# manually deployement resources files.
kubectl apply -f ns.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
```
3. the commands of retrieving the list/status of the resources you were deployed.  
```bash
kubectl get deploy,rs -n leo-k8s-work
kubectl get svc,ep -n leo-k8s-work
kubectl get ns -n leo-k8s-work
kubectl get po -n leo-k8s-work
# manily use to debug, looking into event when something is wrong.
kubectl describe pod .... -n leo-k8s-work
kubectl edit pod .... -n leo-k8s-work
kubectl delete pod  .... -n leo-k8s-work
kubectl rollout status deploy ... -n leo-k8s-work
kubectl rollout history deploy ... -n leo-k8s-work
....

```
4. the output of the above commands.  

## Step Four:
Validating the application you deployed for the endpoint of /my/email/validate  

with the following command:

curl --header 'HTTP_AUTH_KEY: dummy_key' -X POST -d '{"email": "example@company.com"}' http://xxx/my/email/validate
```bash
curl --header 'HTTP_AUTH_KEY: dummy_key' -X POST -d '{"email": "example@company.com"}' http://192.168.2.20/my/email/validate
```

Write down the command you are using above and the response of the application to the validation-output.txt

When you're done, send all your artifacts (as a zip file) to platform@top20talent.com, these files should allow us to deploy the same result to any other k8s cluster.
