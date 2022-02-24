在sidecar ready前阻止docker启动[参考](https://imroc.cc/post/202105/sidecar-startup-order/)

```bash
kubectl -n istio-system edit cm istio

apiVersion: v1
data:
  mesh: |-
    defaultConfig:
      holdApplicationUntilProxyStarts: true   <- This line
  meshNetworks: 'networks: {}'
kind: ConfigMap
```
