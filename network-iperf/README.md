# Testing network throughput across clusters

This tutorial demonstrates how to perform real-time network throughput measurements on an application router network using the iperf3 tool.

In this tutorial, you will deploy iperf3 servers in separate clusters. You will also create an application router network, which will enable the iperf3 instances to run in client mode and access peer iperf3 servers running on the different clusters (e.g. private and public).

To complete this tutorial, do the following:

* [Prerequisites](#prerequisites)
* [Step 1: Set up the demo](#step-1-set-up-the-demo)
* [Step 2: Create the application router network](#step-2-create-the-application-router-network)
* [Step 3: Deploy the iperf3 servers](#step-3-deploy-the-iperf3-servers)
* [Step 4: Run benchmark tests across the clusters](#step-4-run-benchmark-tests-across-the-clusters)
* [Next steps](#next-steps)

## Prerequisites

You should have access to three OpenShift clusters:
* A "private cloud" cluster running on your local machine
* Two public cloud clusters running in public cloud providers

## Step 1: Set up the demo

1. On your local machine, make a directory for this tutorial and clone the following repos into it:

   ```bash
   $ mkdir network-iperf-demo
   $ cd network-iperf-demo
   $ git clone git@github.com:skubaproject/skoot.git # for creating the application router network
   $ git clone git@github.com:skubaproject/demos.git # for deploying the iperf3 servers
   ```

2. Prepare the OpenShift clusters.

   1. Log in to each OpenShift cluster in a separate terminal session. You should have one cluster running locally on your machine, and two clusters running in public cloud providers.
   2. In each cluster, create a namespace for this demo.
  
      ```bash
      $ oc new-project network-iperf-demo
      ```

## Step 2: Create the application router network

The application router network provides connectivity across the three clusters without the need for special network and firewall configuration rules.

1. Open the `network-iperf-demo/demos/topology2/skoot-topology2.conf` file.

2. Replace the variables with the names of your OpenShift clusters and namespaces.

   <dl>
   <dt>`${OPENSHIFT_CLUSTER_NAME_1}`</dt>
   <dd>The name of the first public cloud cluster.</dd>
   <dt>`${NAMESPACE_1}`</dt>
   <dd>The name of the namespace on the first public cloud cluster.</dd>
   <dt>`${OPENSHIFT_CLUSTER_NAME_2}`</dt>
   <dd>The name of the second public cloud cluster.</dd>
   <dt>`${NAMESPACE_2}`</dt>
   <dd>The name of the namespace on the second public cloud cluster.</dd>
   <dt>`${NAMESPACE_3}`</dt>
   <dd>The name of the namespace on the private cloud cluster that is running on your local machine.</dd>
   </dl>

3. Use `skoot` to create the application router network.

   ```bash
   $ cd ~/network-iperf-demo/skoot/python/tools
   $ source export_path.sh
   $ skoot -c -o ~/network-iperf-demo/demos/topology2/skoot-topology2.conf
   ```

4. Verify that the application router network is created.
  
   1. In a web browser, access the web console for the application router network. The URL for the web console is `console.${NAMESPACE_3}.127.0.0.1.nip.io`.
   2. Click the **Topology** tab.

   TODO: add a screenshot
 
## Step 3: Deploy the iperf3 servers

After creating the application router network, you deploy the three iperf3 servers to each of the clusters.

The `demos/network-iperf` directory contains the YAML files that you will use to create the servers. Each YAML file describes the set of Kubernetes resources needed to create an iperf3 server and connect it to the application router network.

TODO: create a project/namespace, same as topology deployment

1. In the terminal for the private cloud, deploy the first iperf3 server:

   ```bash
   $ cd ~/network-iperf-demo/demos/network-iperf/
   $ oc apply -f deployment-iperf3-a.yaml
   ```

2. In the terminal for the first public cloud, deploy the second iperf3 server:

   ```bash
   $ cd ~/network-iperf-demo/demos/network-iperf/
   $ oc apply -f deployment-iperf3-b.yaml
   ```

3. In the terminal for the second public cloud, deploy the third iperf3 server:

   ```bash
   $ cd ~/network-iperf-demo/demos/network-iperf/
   $ oc apply -f deployment-iperf3-c.yaml
   ```

## Step 4: Run benchmark tests across the clusters

After deploying the iperf3 servers into the private and public cloud clusters, the application router network connects the servers and enables communications even though they are running in separate clusters.

1. In the terminal for the private cloud, run the iperf3 client benchmark against each server:

   ```bash
   $ iperf3 -c  $(oc get service iperf3-svc-a -o=jsonpath='{.spec.clusterIP}')
   $ iperf3 -c  $(oc get service iperf3-svc-b -o=jsonpath='{.spec.clusterIP}')
   $ iperf3 -c  $(oc get service iperf3-svc-c -o=jsonpath='{.spec.clusterIP}')   
   ```

2. In the terminal for the first public cloud, attach to the iperf3 server container running in the cluster and run the iperf3 client benchmark against each server:

   ```bash
   $ oc exec -it $(oc get pod -l application=iperf3-server-b -o=jsonpath='{.items[0].metadata.name}') -- iperf3 -c  $(oc get service iperf3-svc-a -o=jsonpath='{.spec.clusterIP}')
   $ oc exec -it $(oc get pod -l application=iperf3-server-b -o=jsonpath='{.items[0].metadata.name}') -- iperf3 -c  $(oc get service iperf3-svc-b -o=jsonpath='{.spec.clusterIP}')
   $ oc exec -it $(oc get pod -l application=iperf3-server-b -o=jsonpath='{.items[0].metadata.name}') -- iperf3 -c  $(oc get service iperf3-svc-c -o=jsonpath='{.spec.clusterIP}')   
   ```

3. In the terminal for the second public cloud, attach to the iperf3 server container running in the cluster and run the iperf3 client benchmark against each server:

   ```bash
   $ oc exec -it $(oc get pod -l application=iperf3-server-c -o=jsonpath='{.items[0].metadata.name}') -- iperf3 -c  $(oc get service iperf3-svc-a -o=jsonpath='{.spec.clusterIP}')
   $ oc exec -it $(oc get pod -l application=iperf3-server-c -o=jsonpath='{.items[0].metadata.name}') -- iperf3 -c  $(oc get service iperf3-svc-b -o=jsonpath='{.spec.clusterIP}')
   $ oc exec -it $(oc get pod -l application=iperf3-server-c -o=jsonpath='{.items[0].metadata.name}') -- iperf3 -c  $(oc get service iperf3-svc-c -o=jsonpath='{.spec.clusterIP}')   
   ```

## Next steps

TODO: describe what the user should do after completing this tutorial
