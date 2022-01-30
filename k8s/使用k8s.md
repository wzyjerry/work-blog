# 使用k8s

1. 重启服务

   ``` bash
   kubectl get deployment.apps/org-service -o yaml | kubectl replace --force -f -
   ```

   

