# neo-kube

## Introduction

The aim of this project is to make it as easy as possible to quickly deploy a privately-owned NEO network for smart contract and other development. This makes it easier for team members to collaborate.

To accomplish that goal, this repo contains a few [Kubernetes](https://kubernetes.io/) service and deployment yaml files that can be applied with `kubectl`.

## Instructions

### 1. Prepare cluster on cloud provider

The first step is to setup a cluster. You should have basic familiarity with how to do this and the details are beyond the scope of this document.

If you're on Google Cloud, you can review the [hello-app](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app) tutorial for the basics.

### 2. Deploy the NEO nodes

For the NEO network we use [neo-privatenet-docker](https://github.com/CityOfZion/neo-privatenet-docker) and specifically the [neo-privnet-with-gas](https://hub.docker.com/r/metachris/neo-privnet-with-gas/) image. On that page you can read about the precreated wallets and how to access your private NEO when done.

To get this repo and deploy run:

```
git clone https://github.com/slipo/neo-kube.git
cd neo-kube
kubectl apply -f deployments/neo-privnet-with-gas.yaml
kubectl apply -f services/neo-privnet.yaml
```

This exposes the private network to the Internet exposing port ranges 20333-20336 and 30333-30336. Run `kubectl get services` to get the external IP. It will say "pending" for some time while the load balancer is brought up. Wait for it. You need that IP.

### 3. Deploy light wallet client database

For some usecases a light wallet client is required. For example if you need a dApp webpage to display the user's balance.

#### neo-scan

[neo-scan](https://github.com/CityOfZion/neo-scan) is the recommended option.

Edit `deployments/neo-scan.yaml` and change `[NEOEXTERNALIP]` to the external IP you got in step 2.

Run

```
kubectl apply -f deployments/postgres.yaml
kubectl apply -f services/postgres.yaml
kubectl apply -f deployments/neo-scan.yaml
kubectl apply -f services/neo-scan.yaml
```

Run `kubectl get services` to get the IP for neo-scan. It will say "pending" for some time while the load balancer is brought up. When done, port 4000 will be open to the Internet.

Visit `http://[NEO-SCAN-IP]:4000/` to verify everything working. If it's not showing right away check the neo-scan pod's logs. It can take a little while to start up.

#### neon-wallet-db

[neon-wallet-db](https://github.com/CityOfZion/neon-wallet-db) is the older light wallet database option.

Edit `deployments/neon-wallet-db.yaml` and change `[NEOEXTERNALIP]` to the external IP you got in step 2.

Run

```
kubectl apply -f deployments/mongo.yaml
kubectl apply -f services/mongo.yaml
kubectl apply -f deployments/redis.yaml
kubectl apply -f services/redis.yaml
kubectl apply -f deployments/neon-wallet-db.yaml
kubectl apply -f services/neon-wallet-db.yaml
```

Run `kubectl get services` to get the IP for neon-wallet-db. It will say "pending" for some time while the load balancer is brought up. When done, port 5000 will be open to the Internet.

Visit `http://[NEONDB-IP]:5000/v2/network/nodes` to verify everything working. It will take neon-wallet-db a bit of time to index the preloaded blocks in the neo-privnet-with-gas image we used in step 2.

### 4 Profit

You can now connect to this network with clients such a [neo-python](https://github.com/CityOfZion/neo-python) using the IPs and ports mentioned above. You can read the documention for [neo-privnet-with-gas](https://hub.docker.com/r/metachris/neo-privnet-with-gas/) to get the keys for 100m NEO.



