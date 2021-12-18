# Deploy a log monitoring system

**Disclaimer: This is a challenge project that is not meant to be used in production!**

## Project steps

1. Setup K8s cluster on https://cloud.digitalocean.com/
2. Deploy Elasticsearch and Kibana to store and browse logs
3. Deploy FluentBit to collects and push logs to Eelasticsearch

### Setup K8s cluster

Create an account on DigitalOcean and follow instruction on UI to create Kubernetes cluster. The process of creating a cluster is pretty straight forward, I've created a cluster with 2 nodes, it should be more than enough for this challenge.

![image](https://user-images.githubusercontent.com/15143151/146643310-b1e58e6b-a7be-4ebf-8727-e0b7f725b2a4.png)

### Deploy Elasticsearch and Kibana

I used Elastic Cloud on Kubernetes (ECK) for this and followed quick start guide which you can find here https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html

First we need to install custom CRDs and the operator:

```
kubectl create -f https://download.elastic.co/downloads/eck/1.9.1/crds.yaml
kubectl apply -f https://download.elastic.co/downloads/eck/1.9.1/operator.yaml
```

I can see that operator is up and running.

![image](https://user-images.githubusercontent.com/15143151/146643556-50fa1fe9-3d77-40a7-a6fa-9c568d94a689.png)

Next, deploy Elasticsearch cluster. You can check my manifests in the repo, there is the command to apply it:

```
kubectl apply -f manifests/elasticsearch.yaml
```

After a few seconds, Elasticsearch should be running in `default` namespace.

![image](https://user-images.githubusercontent.com/15143151/146643567-09816f5a-50ef-4165-9226-e0ad033da013.png)

Let's check if it works by sending a request. Before doing so, I need to port-forward my local port to K8s Elasticsearch port.

![image](https://user-images.githubusercontent.com/15143151/146643663-337a554e-90f4-493e-8d38-4a33b4716f1e.png)

And finally send a reqeust.

![image](https://user-images.githubusercontent.com/15143151/146643595-ac1fa04a-791e-4725-a32a-e68c2a7662cb.png)

Yup, so far so good. 👌

Let's move on by deploying Kibana:

```
kubectl apply -f manifests/kibana.yaml
```

Then, let's do port forwarding again:

![image](https://user-images.githubusercontent.com/15143151/146643797-4e5639fa-84df-4ef2-8ae4-e3cac18377a2.png)

And now I'm able to access Kibana from my browser.

![image](https://user-images.githubusercontent.com/15143151/146643807-6ccc2c85-24a9-45db-a034-4d9cd927b484.png)

### Deploy FluentBit

In the last step, I used Helm to deploy FluentBit, you can check chart values at `packages/fluent-bit-values.yaml`. The important part here is output configuration, to allow FluentBit to push logs to Elasticsearch:

```yaml
  outputs: |
    [OUTPUT]
        Name es
        Match kube.*
        Host quickstart-es-http
        Logstash_Format On
        Retry_Limit False
        HTTP_User elastic
        HTTP_Passwd 6byJ6IfOYgyLZ8m472cg2926
        tls On
        tls.verify Off
```

Now it's time to install FluentBit:

```
helm install fluent-bit fluent/fluent-bit -f packages/fluent-bit-values.yaml
```

After a few seconds, I see that FluentBit successfully installed and is running on both nodes as a DaemonSet.

![image](https://user-images.githubusercontent.com/15143151/146643955-87c14542-30e8-4e1b-920f-1ed6ebb74b06.png)

When opening Kibana, I can see that logs are successfully being pushed to Elasticsearch.

![image](https://user-images.githubusercontent.com/15143151/146644040-d816e6f8-dc9f-48e5-b357-d58cfa872eab.png)
