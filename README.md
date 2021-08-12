# CoinAPI OEML Helm Charts

[![Artifact HUB](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/coinapi)](https://artifacthub.io/packages/search?repo=coinapi)


Order and Execution Management Layer (OEML) is a self-hosted software that managing orders, executions and exposure in an efficient, fast, cost-effective, and straightforward manner. An OEML allows you to route orders to multiple cryptocurrency exchanges simultaneously using a simple, robust, and unified Application Programming Interface (API).

## Installing Charts from this Repository

#### Add the Repository to Helm

```console
$ helm repo add coinapi-charts https://coinapi.github.io/helm-charts/
$ helm repo update
```

## Configuration

The example config below will create two api pods, a composite pod, and a webui pod.
One exchange account corresponds to one entry in the values.yaml file. That means, for multiple exchange accounts,
please add one config entry with the matching API keys for each account i.e. binance-01, binance-02, kraken-01, kraken-02, etc. 
A complete list of all supported exchanges can be found in the [API documentation](https://docs.coinapi.io/oeml.html#oeml-order-params).

```
cat > values.yaml << EOF

# Using WebUI (oeml-api/values.yaml)
webUI:
  enabled: true

# Using composite (oeml-api/values.yaml)
oemlCompositeAPI:
  enabled: true

# Setting the CoinAPI key (oeml-api/values.yaml)
extraEnv:
  - name: CoinAPI__ApiKey
    value: "YOUR_COINAPI_APIKEY"

# Specify listing of the accounts that OEML should manage (oeml-api/acct-dev.yaml)
accounts:
  # Binance spot market 
  - name: "binance"
    env:
      - name: OEML__ExchangeId
        value: "BINANCE"
      - name: OD__PublicApiKey
        value: "XXX"
      - name: OD__PrivateApiKey
        value: "YYY"
        
  # Binance spot testnet
  - name: "binanceuat"
    env:
      - name: OEML__ExchangeId
        value: "BINANCEUAT"
      - name: OD__PublicApiKey
        value: "XXX"
      - name: OD__PrivateApiKey
        value: "YYY"
        
EOF
```

### Install oeml-api latest version

```console
$ helm install oeml-api coinapi-charts/oeml-api -f values.yaml
```

### Install oeml-api custom version

Version listing available here: http://coinapi-releases.s3-website-us-east-1.amazonaws.com/?prefix=oeml-api/

```console
$ helm install oeml-api coinapi-charts/oeml-api -f values.yaml --set oemlAPI.tag={version}
```

## Verify installation 

```
kubectl get pods | grep oeml
```

Expected output:
```
> oeml-api-binance-7754fc95cc-x2gkk        1/1     Running
> oeml-api-binance-test-757c8bbd95-2mxd5   1/1     Running
> oeml-api-composite-d5688f8f6-csbwh       1/1     Running
> oeml-webui-685cd94d77-hq68j              1/1     Running 
```

### Inspect

Inspect binance api pod:
```
kubectl logs oeml-api-binance-7754fc95cc-x2gkk
```

Inspect testnet api pod:
```
kubectl logs oeml-api-binance-test-757c8bbd95-2mxd5
```

Expected output for both api pods:
```
[02:22:04 WRN] Overriding address(es) 'http://+:80'. Binding
to endpoints defined in UseKestrel() instead.
Hosting environment: Production
Content root path: /app
Now listening on: http://[::]:80
Application started. Press Ctrl+C to shut down.
```

### List services
```
kubectl get svc | grep oeml
```

Expected output:
```
oeml-api-binance        ClusterIP   10.112.9.119    <none>        80/TCP
oeml-api-binance-test   ClusterIP   10.112.15.239   <none>        80/TCP
oeml-api-composite      NodePort    10.112.8.159    <none>        80:30578/TCP
oeml-webui              NodePort    10.112.4.195    <none>        80:30857/TCP              
```

## 5) Connect to OEML API

The composite pod gives unified access to all connected exchanges & accounts. Therefore,
the oeml-api-composite is the only pod required to connect to the OEML system.

### Connect from within the cluster 

Deploy a pod with a shell

```
kubectl apply -f https://k8s.io/examples/application/shell-demo.yaml
```

Verify the shell pod is online
```
kubectl get pod shell-demo
```

Get api-composite IP address:
```
kubectl get svc | grep oeml-api-composite

>oeml-api-composite      NodePort    10.112.8.159  
```

Exec into the pod
```
kubectl exec --stdin --tty shell-demo -- /bin/bash
```

CURL IP of the composite pod.
```
curl 10.112.8.159/v1/balances --header 'Accept: application/json'
```

CURL DNS of the composite pod.

DNS name may vary depending on cloud provider. 
```
curl oeml-api-composite.default.svc.cluster.local/v1/balances --header 'Accept: application/json'
```

Expected output:
```
[
  {
    "type": "BALANCE_SNAPSHOT",
    "exchange_id": "BINANCE",
  },
  {
    "type": "BALANCE_SNAPSHOT",
    "exchange_id": "BINANCEUAT",
    "data": [
      {
        "id": "XRP",
        "asset_id_exchange": "XRP",
        "asset_id_coinapi": null,
        "balance": 50000.00000000,
        "available": 50000.00000000,
        "locked": 0.00000000,
        "traded": 0.0,
        "last_updated_by": "EXCHANGE",
        "rate_usd": null
      },
      // more entries
      ]
  }
]
```

Exit the shell pod
```
exit 
```

Delete shell pod
```
kubectl delete pod shell-demo
```


### Connect cluster to localhost

Port-forward from cluster port 80 to localhost port 8080
```
 kubectl port-forward service/oeml-api-composite 8080:80
```

Expected output:
```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

CURL account balance.
```
curl localhost:8080/v1/balances --header 'Accept: application/json'
```



Expected output:
```
[
  {
    "type": "BALANCE_SNAPSHOT",
    "exchange_id": "BINANCE",
  },
  {
    "type": "BALANCE_SNAPSHOT",
    "exchange_id": "BINANCEUAT",
    "data": [
      {
      // more entries
      },
      ]
  }
]
```
