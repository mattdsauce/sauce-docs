---
id: kubernetes
title: Sauce Connect Proxy 5 in Kubernetes
sidebar_label: Kubernetes
---

[Kubernetes](https://kubernetes.io/), an industry-standard environment for management of containerized applications, may be used for the automation of much of the operational effort required to run Sauce Connect Proxy.
This article provides documentation for running Sauce Connect Proxy in [Kubernetes](https://kubernetes.io/).

## Running the Sauce Connect Docker Container

If you need a Sauce Connect Proxy to stay up indefinitely, we recommend using a [Helm chart](https://helm.sh/docs/topics/charts/) to manage your Sauce Connect Proxy instance or pool.

The Sauce Connect Proxy Docker GitHub repository provides [a reference Helm chart](https://github.com/saucelabs/sauce-connect-docker/tree/main/chart/sauce-connect) that may be used as is, or adapted to your needs.
To use that chart:

- Define a values file containing your configuration, for example:

```yaml
sauceApiRegion: us-west
sauceUser: johndoe
sauceApiKey: "xxx-xxx-xxx"
tunnelName: "my-k8s-tunnel"
tunnelPool: true
tunnelPoolSize: 2
```

- Run Helm install

```bash title="helm install"
helm install sauce-connect  ./chart/sauce-connect --values /path/to/values.yaml
```

- Use the following commands in order to get the Sauce Connect Proxy application status and logs

```bash title="SC logs"
$ POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=sauce-connect,app.kubernetes.io/instance=sauce-connect" -o jsonpath="{.items[0].metadata.name}")
$ CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
$ kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
$ curl -s 127.0.0.1:8080/api/v1/status | jq .
$ curl -s http://127.0.0.1:8080/api/v1/status | jq .
{
  "firstConnectedTime": 1662098351,
  "tunnelID": "11111",
  "tunnelName": "my-k8s-tunnel",
  "tunnelServer": "tunnel-59569b.tunnels.us-west-1.saucelabs.com",
  "lastStatusChange": 1662098350,
  "reconnectCount": 0,
  "tunnelStatus": "connected"
}
$ kubectl logs $POD_NAME -f
...
2022-08-02 02:59:11.464 [8] [CLI] [info] Connection state has changed: previous: INIT, actual: CONNECTED.
2022-08-02 02:59:11.464 [8] [CLI] [info] Sauce Connect is up, you may start your tests.
```

## Additional Resources

- [Kubernetes](https://kubernetes.io)
- [Helm charts](https://helm.sh/docs/topics/charts/)
