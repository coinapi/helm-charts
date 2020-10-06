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
$ helm install coinapi-charts/oeml-api --set oemlAPI.tag={version}
```
