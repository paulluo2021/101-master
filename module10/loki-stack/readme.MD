#官网文档
#https://grafana.com/docs/loki/latest/installation/helm/
### add grafana repo
helm repo add grafana https://grafana.github.io/helm-charts
### install loki-stack
```
helm upgrade --install loki grafana/loki-stack --set grafana.enabled=true,prometheus.enabled=true,prometheus.alertmanager.persistentVolume.enabled=false,prometheus.server.persistentVolume.enabled=false
helm upgrade --install loki grafana/loki-stack  --set grafana.enabled=true,prometheus.enabled=true,prometheus.alertmanager.persistentVolume.enabled=false,prometheus.server.persistentVolume.enabled=false

```

### if you get the following error, that means your k8s version is too new to install
```
Error: unable to build kubernetes objects from release manifest: [unable to recognize "": no matches for kind "ClusterRole" in version "rbac.authorization.k8s.io/v1beta1", unable to recognize "": no matches for kind "ClusterRoleBinding" in version "rbac.authorization.k8s.io/v1beta1", unable to recognize "": no matches for kind "Role" in version "rbac.authorization.k8s.io/v1beta1", unable to recognize "": no matches for kind "RoleBinding" in version "rbac.authorization.k8s.io/v1beta1"]
```

### download loki-stack
```
helm pull grafana/loki-stack
tar -xvf loki-stack-2.4.1.tgz
```
```
cd loki-stack
```
### replace all `rbac.authorization.k8s.io/v1beta1` with `rbac.authorization.k8s.io/v1` by 
```
sed s#rbac.authorization.k8s.io/v1beta1#rbac.authorization.k8s.io/v1#g *.yaml
```
### install loki locally
```
helm upgrade --install loki ./loki-stack --set grafana.enabled=true,prometheus.enabled=true,prometheus.alertmanager.persistentVolume.enabled=false,prometheus.server.persistentVolume.enabled=false
```
### change the grafana service to NodePort type and access it
login password is in secret `loki-grafana`

kubectl get secret  loki-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
###
[root@kmaster loki-stack]# kubectl get secret loki-grafana -oyaml
apiVersion: v1
data:
  admin-password: VHhFR3U2RHNDeDIwMjBscjAxdXEzNjVkdTN5eGtjQ1FjR3k5dFUxMA==
  admin-user: YWRtaW4=
  ldap-toml: ""
[root@kmaster loki-stack]# echo YWRtaW4=|base64 -d
admin[root@kmaster loki-stack]# echo VHhFR3U2RHNDeDIwMjBscjAxdXEzNjVkdTN5eGtjQ1FjR3k5dFUxMA==|base64 -d
TxEGu6DsCx2020lr01uq365du3yxkcCQcGy9tU10[root@kmaster loki-stack]#

###web
kubectl port-forward service/loki-grafana 3000:80
##打开localhost:3000或者<IP地址>:3000来查看Grafana，登录过程输入用户名admin和上面获得到的密码。
