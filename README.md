# Hydrated Quickstart

I've always been frustrated with tutorials and quickstarts that show you how to draw stick figures and then follow that up with the Sistene Chappel so this is my attempt to give out some more formulated examples that hopefully help you get some air under your wings with some `Kustomize` mixed in and the ability to shape it as needed from there. 

For now, there are three examples; an ephemeral quickstart cluster, a generic three node (search) cluster, a tiered storage cluster. These are by no means the only way, the singular best way, or anything more than a more complete starting point than a basic quickstart cluster and one you can tinker and learn from. Each cluster will have some things different about it and each will have it's own readme file to explain those differences further.


## Initial Setup of ECK Operator

You are required to setup the ECK operator before you are able to use any of its custom resources.  We need to install the custom resourse definitions(CRDs) and then spin up the ECK Operator container that will manage the new custom resources we defined.

As always, please consult the current documentation first to be safe. -> https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-eck.html


#### *Quickstart Steps*
1. Run these two commands
```
# Install the Custom Resource Definitions
kubectl create -f https://download.elastic.co/downloads/eck/2.5.0/crds.yaml
# Install the Elastic Operator
kubectl apply -f https://download.elastic.co/downloads/eck/2.5.0/operator.yaml
```

The operator should be installed and running at this point in the `elastic-system` namespace. Check the logs iwth this command if you would like.

`kubectl -n elastic-system logs -f statefulset.apps/elastic-operator`

2. Choose one of the listed clusters and head to a "baseline", "dev", or "prod" directory.
- Quickstart
- 3 Node Search Cluster(s)
- Tiered Storage Cluster(s)

---





