### 排水

```bash
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <node>
```
