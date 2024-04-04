# Elastic K8s POC

Simple Elastic Cloud on K8s POC setup.

## Install

On a new Ubuntu Server 22.04 (distro choice may vary, but that is what I used), perform the following:

## K3s & Operator Installation

```bash
# Install K3s
curl -sfL https://get.k3s.io | sh -

# Grab Kubeconfig
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown {$USER}:{$USER} ~/.kube/config
export KUBECONFIG=~/.kube/config

# Install Operator
kubectl create -f https://download.elastic.co/downloads/eck/2.12.1/crds.yaml
kubectl apply -f https://download.elastic.co/downloads/eck/2.12.1/operator.yaml
```

## Grab a coffee to make sure its installed all the way

Should only take a few minutes

## Installing the Elastic Cloud

The config file is located in this repo. You can clone the repo, or copy it to the server however you would like.

This is a multi-step process. Essentially, we are going to deploy it, modify it, then deploy it again. If you would like to give Elasticsearch more storage, please find the storage request in the `elastic-cloud.yaml` and change it from `20Gi` to whatever you would like.

```bash
# install the Elastic Cloud
kubectl apply -f ./elastic-cloud.yaml
```

Now, we need to update the `xpack.fleet.agents.elasticsearch.hosts` and `xpack.fleet.agents.fleet_server.hosts` variable so we can use fleet outside of the cluster.

```bash
# get the "LoadBalancer" IP/Port
kubectl get svc
```

This should return output similar to:
```
[demo@es-k3s1:~/]$ k get svc
NAME                             TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
kubernetes                       ClusterIP      10.43.0.1       <none>          443/TCP          4h16m
quickstart-es-transport          ClusterIP      None            <none>          9300/TCP         132m
quickstart-es-http               ClusterIP      10.43.76.109    <none>          9200/TCP         132m
quickstart-es-internal-http      ClusterIP      10.43.211.110   <none>          9200/TCP         132m
quickstart-es-default            ClusterIP      None            <none>          9200/TCP         132m
elasticsearch-es-transport       ClusterIP      None            <none>          9300/TCP         76m
elasticsearch-es-internal-http   ClusterIP      10.43.218.147   <none>          9200/TCP         76m
elasticsearch-es-default         ClusterIP      None            <none>          9200/TCP         76m
kibana-kb-http                   LoadBalancer   10.43.175.88    192.168.1.1     5601:31383/TCP   26s
elasticsearch-es-http            LoadBalancer   10.43.32.161    192.168.1.1     9200:31128/TCP   12s
fleet-server-agent-http          LoadBalancer   10.43.226.194   192.168.1.1     8220:30467/TCP   9s
```

You want the "External-IP" and the mounted port. For example, for `elasticsearch-es-http`, it would be `192.168.1.1:31128`.

Using the `elasticsearch-es-http` and `fleet-server-agent-http`, update `xpack.fleet.agents.elasticsearch.hosts` and `xpack.fleet.agents.fleet_server.hosts` in the `elastic-cloud.yml`, keeping the URI format. Then re-deploy witht the following command:

```bash
kubectl apply -f ./elastic-cloud.yaml
```

## Getting Logged In

The username by default is `elastic`, with a randmonly generated password. The below command will get you the password.

```bash
kubectl get secret elasticsearch-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo
```

To login, use the IP & port from above for `kibana-kb-http`, and go in your browser with `https://IP:PORT`.

# Misc Stuff

## Check ES / Kibana Health

```bash
# es
kubectl get elasticsearch

# kibana
kubectl get kibana
```

## Adding your first external Agent

In Kibana, go to Fleet -> Agent policies -> Create agent policy. Give it a name and select create

Now, go to Agents -> Add Agent. Select the policy you previously created.

Copy the commands for your host type. When you run the `elastic-agent install` command, add the `--insecure` flag to bypass any certificate errors. Do not do this for a production install.
