# helm-charts

## OEML API

Order and Execution Management Layer (OEML) is a self-hosted software that managing orders, executions and exposure in an efficient, fast, cost-effective, and straightforward manner. An OEML allows you to route orders to multiple cryptocurrency exchanges simultaneously using a simple, robust, and unified Application Programming Interface (API).

## Installing Charts from this Repository

Add the Repository to Helm:

```console
$ helm repo add coinapi-charts https://coinapi.github.io/coinapi-charts/
```

Install oeml-api:

```console
$ helm install oeml-api coinapi-charts/oeml-api -f acct-dev.yaml --set oemlAPI.tag={version}
```

## Settings

Using WebUI (oeml-api/values.yaml)

```console
webUI:
  enabled: true
```

Using composite (oeml-api/values.yaml)

```console
oemlCompositeAPI:
  enabled: true
```

Setting the CoinAPI key (oeml-api/values.yaml)

```console
extraEnv:
  - name: CoinAPI__ApiKey
    value: "{ApiKey}"
```    

Use of the stock exchange (oeml-api/acct-dev.yaml)

```console
accounts:
  - name: binance
    env:
      - name: OEML__ExchangeId
        value: "BINANCE"
      - name: OD__PublicApiKey
        value: "{PublicApiKey}"
      - name: OD__PrivateApiKey
        value: "{PrivateApiKey}"
```
