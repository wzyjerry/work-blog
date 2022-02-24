# Kubernetes文档

## 目录
1. [Gitlab CI/CD](0-Gitlab_CICD.md)
2. [Kubernetes](1-Kubernetes.md)
3. [存储](2-存储.md)


### 排水

```bash
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <node>
```
