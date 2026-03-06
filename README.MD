Jenkins K8s Manifests (Namespace: dgb-ny2np2-jenkins)
This folder contains Kubernetes YAMLs to configure networking and credentials for a Jenkins setup running in the dgb-ny2np2-jenkins namespace.

## **1) networkpolicy.yaml**

Allows all ingress and egress for all pods in the dgb-ny2np2-jenkins namespace.



## **2) secret-admin.yaml**

Creates a generic opaque Secret with admin credentials (likely used by the Jenkins controller or bootstrap job).


## **3) secret-agent.yaml**

Creates two Secrets that hold Jenkins agent connection tokens (JENKINS_SECRET) used by inbound agents.


## **4)pvc.yaml — Jenkins Home Storage**
This PVC provides persistent storage for /var/jenkins_home so that Jenkins data (jobs, configs, plugins, secrets) survives pod restarts.

Key Points

RWX (ReadWriteMany) mount — required for Jenkins controller.

Uses enterprise storage class dynamicfiles.

Size = 25Gi.

Mounted in the Deployment at /var/jenkins_home.



## **deployment.yaml — Jenkins Controller**

Runs the Jenkins controller pod with:

Admin credentials from Secret

JCasC config from ConfigMap

Plugin list

Persistent Jenkins home

Kubernetes-based agents support

Important Features

Runs as non-root user (runAsUser: 1000)

FS permissions fixed via fsGroup=1000

mounts JCasC and plugins list

Readiness & liveness probes

Affinity to spread pods across nodes

Uses your air‑gapped Harbor image


## **config-jcasc.yaml — Jenkins Configuration-as-Code**

Bootstraps Jenkins automatically using the JCasC plugin.

What It Configures

Jenkins admin user (from Secret)

Welcome message

Disables setup wizard

Authorization strategy

Kubernetes cloud definition

Agent templates for:

default

agent1

agent2

agent3

## **config-plugins.yaml — Jenkins Plugins List**
Purpose
Provides a list of plugins that Jenkins will install on startup.


## **service.yaml — Jenkins Service (ClusterIP)**
Exposes the Jenkins controller inside the cluster at ports:

8080 (UI / HTTP)

50000 (JNLP agent port)


## **agent.deploy.yaml — Static Agent Deployments**

Runs two static Jenkins agents (agent-1 and agent-2) that connect to the controller via WebSocket using secrets agent-1-secret and agent-2-secret.

## **rbac.yaml — Controller RBAC**
Grants the Jenkins controller’s ServiceAccount permissions in this namespace needed by the Kubernetes plugin (create/list/watch/delete pods, read secrets, etc.).

deployment.yaml references serviceAccountName: dgb-ny2np2-jnks-svc-acc

The Role permits common operations the Jenkins Kubernetes plugin requires in-namespace.

# Order of Apply (Suggested)

kubectl apply -f networkpolicy.yaml

kubectl apply -f secret-admin.yaml

kubectl apply -f secret-agent.yaml

kubectl apply -f pvc.yaml

kubectl apply -f config-plugins.yaml

kubectl apply -f config-jcasc.yaml

kubectl apply -f rbac.yaml

kubectl apply -f deployment.yaml

kubectl apply -f service.yaml

kubectl apply -f agent.deploy.yaml


## **1. See Pods / Deployments / Services**
 
kubectl get pods -n <ns>

kubectl get deploy -n <ns>

kubectl get svc -n <ns>

kubectl get rs -n <ns>

kubectl get secrets -n <ns>


 ## **Check port connectivity using /dev/tcp**
 
 echo > /dev/tcp/google.com/443 && echo CONNECTED || echo FAIL
