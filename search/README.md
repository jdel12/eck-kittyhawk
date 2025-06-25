# Search Example

## Base Diagram

```mermaid
flowchart LR
	req(Requests) --> Elasticsearch_Cluster
	subgraph Elasticsearch_Cluster
	es1 --- es2
	end
	Elasticsearch_Cluster --- Kibana
	Elasticsearch_Cluster --- A[/Kubernetes Secret: License\]
```


## Deployable Assets in this Directory

| Elastic Cluster | Filename |  Resource (Kind) | Count | Features Added |
| :-------------: |:-------------:| :-------------: | :-------------: | :-------------: |
|main|elasticsearch.yml|Elasticsearch|2|[Virtual Memory](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-virtual-memory.html), [Persistent Storage](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-volume-claim-templates.html), [Compute Resources](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-managing-compute-resources.html), [Custom Configuration Files (Synonyms)](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-bundles-plugins.html), [Elastic Audit Settings](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s_audit_logging.html), [Internal Monitoring](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-stack-monitoring.html)|
|main|kibana.yml|Kibana|1|[Kibana APM Self Monitoring](https://www.elastic.co/guide/en/kibana/current/kibana-debugging.html), [Compute Resources](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-kibana-advanced-configuration.html)|
|main|rbac.yml|[RBAC Roles](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)||Kubernetes RBAC for Agents, Fleet|
|main|synonyms-configmap.yml|configMap||[Custom Configuration Files](https://www.elastic.co/guide/en/cloud-on-k8s/master/k8s-bundles-plugins.html)|
|ECK-Wide|trial-license.yml|Secret:[License](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-licensing.html)||Trial License to Enable All Features|
|_optional_|fleet.yml|Fleet Server(Agent)|1|[Fleet](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-elastic-agent-fleet.html), [APM Integration](https://www.elastic.co/guide/en/apm/guide/current/upgrade-to-apm-integration.html) |
|_optional_|fleet.yml|[Agents](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-elastic-agent-fleet-configuration-examples.html)|1+n|[System](https://docs.elastic.co/en/integrations/system), [Kubernetes](https://docs.elastic.co/integrations/kubernetes)|


---

## Search Cluster

We have taken our previous quickstart cluster but made a few additions like persistent storage and a custom synonyms file to make a relatively straightforward 2-3 node Elasticsearch cluster.  The table above should reflect the assets being deployed and documentation references to the added features and settings. You can deploy the yamls in the base directory flat and as is with a normal kubectl command. 

`kubectl apply -f elasticsearch.yml -f kibana.yml -f synonyms-configmap.yml -f legacy-apmserver.yml -f trial-license.yml`

However, next to the base directory is the overlay directory which has  `dev` and `prod` subdirectories.  By using a tool called `Kustomize` that is built into every kubectl command line, we can do some simple templating by environment and "overlay" the values appropriately.  The goal is that in lieu of your own preferred strategy, this should be a low depency method to build out and maintain a core base yaml set with various overlays making changes to say resources, version, or node counts. You are by no means forced to use this and can generate a template and just use the output. 

You can also optionally deploy Agents with Fleet by following the instructions further down the page.

> NOTE: You may also install `Kustomize` into your CLI separately and this often is a more current version than the packaged one with the kubectl tool - consider this if you see different funcitonality). 


## Kustomize Instrucitons

`kubectl kustomize` or `kubectl kustomize build` will generate a templated set of YAMLs when run in the right place and in this case, either the `dev` or `prod` sub-directories under `overlay`.

The way it works is that in your terminal, place yourself in one of the the overlay subdirectories, either dev or prod, and run a command similar to the examples below.  Each directory's kustomizaton.yaml file will reference where it pulls the base YAML files from (our base directory) and then changes (patches) to be "overlayed" on top of them; things like compute settings or node counts are typically larger in the production.  The `Base Values` chart below shows some of the differences between each environment. These are generic use case starter values and I would encourage you to iteratively test or engage with Elastic Professional Services etc. to gauge these if you aren't sure.  You can edit any of the values for dev or prod in the `kustomization.yaml` file in each and/or the patches directory and some should have commented values like namespace if you want to add more.  

**Example Commands:**

- Deploy Dev Cluster

While in the `overlay/dev` directory, run `kubectl kustomize | kubectl apply -f -` or `kubectl kustomize build | kubectl apply -f -`

- Deploy Prod Cluster*

While in the `overlay/prod` directory, run `kubectl kustomize | kubectl apply -f -` or `kubectl kustomize build | kubectl apply -f -`

>NOTE: While I look for an easier way to update all the CRD object name references, you will need to manually change the elasticsearch cluster name if you deploy these to the same namespace or change the namespaces. 

## Base Values

| Value | Base | Dev | Prod |
| :-------------: | :-------------: | :-------------: |:-------------: |
|Count|2|2|3|
|Version|9.0.0|9.0.0|9.0.0|
|Elastic Compute Settings|-|CPU:(*request*:2-*limit*:6)<br>Memory:(*request*:8GB-*limit*:16GB)|CPU:(*request*:12-*limit*:15)<br>Memory:(*request*:64-*limit*:64)|
|JVM Options|Auto|8GB Heap|30GB Heap|
|Kibana Compute Settings|-|CPU:.5, Mem:1GB|CPU:2, Mem:4GB|
|Volume Claim Size|10GB|25GB|250GB|
|SnapshotSettings|-|-|-|

## Adding Fleet with Preconfigured Agents

You need to do three things to bolt this functionality one. 
1) Uncomment the `fleet.yml` line in the base directory `kustomization.yaml` file.
2) Uncomment the Fleet configuration in the `kibana.yml` base file.
3) Uncomment the Fleet settings in each overlay directories' (dev,prod) `kustomization.yaml` file.

That should allow you to spin up Fleet with Agents adjacent to this setup and be a simple three node standard cluster.

## Viewing your Data in Kibana

You can use the built-in `elastic` user and the password is stored in a default secret. The command below should pull this value for you. 

`kubectl get secret elasticsearch-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo`

You can then access the running Kibana by running a kubectl port-forward command like below or naturally, working to expose the cluster some other way.

`kubectl port-forward service/kibana-kb-http 5601`


## Adding Kube-State-Metrics

To add the kube-state-metrics server (a Kubernetes official resource) for the `state_*` metricsets, you can visit the repo or deploy the standard example directory resources after cloning the repo locally.

eg:
1. `git clone https://github.com/kubernetes/kube-state-metrics.git`
2. `kubectl apply -f kube-state-metrics/examples/standard/`


## What next?

Some final things to round out a cluster might be topics like...

- Want to see the API specs for each CRD? -> [CRD API Reference](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-api-reference.html)
- Want to add Elasticsearch cluster settings? -> [Node Configurations](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-node-configuration.html)
- Add an Ingress and/or expose your cluster wider -> [Ingress Example](https://github.com/elastic/cloud-on-k8s/tree/main/config/recipes/traefik),[Istio Ingress](https://github.com/elastic/cloud-on-k8s/tree/main/config/recipes/istio-gateway), [ECK Services](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-services.html), [ECK Traffic Splitting](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-traffic-splitting.html)
- Add a comprehensive DR/SLM strategy -> [ECK Automated Snapshots](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-snapshots.html)
- Setup a Certificate Strategy -> [Custom HTTP Layer Certs](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-custom-http-certificate.html), [TLS Certs](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-tls-certificates.html), [Cert Manager]()
- Customize the image or pod further eg: update strategies or security -> [Update Strategies](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-update-strategy.html), [Customize Pod](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-customize-pods.html)
- Bind the nodes to specific node groups/pools -> [Node Topology](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-advanced-node-scheduling.html)
- Setup SAML Auth (often a copy/paste from your previous elasticsearch.yml) -> [SAML/Auth Config](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-saml-authentication.html)
- Use Your/Custom Certificates -> [Custom Certs](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-custom-http-certificate.html)
- Want to setup persistent storage
	+ [Volume Claim Templates](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-volume-claim-templates.html)
	+ Bonus Reading: [Storage Recommendations](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-storage-recommendations.html) 