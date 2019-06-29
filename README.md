# hpa
follow [the guide](https://github.com/cheferrari/k8s-env-setting-up#9-install-metrics-server) to install metrics-server on your kubernetes cluster

## Clone this repo to your local machine
```
git clone https://github.com/cheferrari/hpa
```
## Create namespace: test
```
kubectl create ns test
```
## Deploy web deploy and svc to the `test` namespace 
```
kubectl apply -f web-deploy.yaml
kubectl apply -f web-svc.yaml
```
## Create HPA
```
kubectl apply -f web-hpa.yaml
# kubectl get hpa -owide -w
NAME      REFERENCE        TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
web-hpa   Deployment/web   0%/50%    2         8         3          3m21s
```
## Increase the CPU usage
### run a load test with `rakyll/hey`
```
#install hey
go get -u github.com/rakyll/hey

#do 10K requests
hey -n 10000 -q 10 -c 5 http://<WEB_SVC_IP>:80/
```
### or run the test use `hpa-load.yaml`
```
kubectl apply -f hpa-load.yaml
```
### You can monitor the HPA events with:
```
# kubectl describe hpa

Events:
  Type     Reason             Age                From                       Message
  ----     ------             ----               ----                       -------
  Normal   SuccessfulRescale  29m                horizontal-pod-autoscaler  New size: 5; reason: cpu resource utilization (percentage of request) above target
  Normal   SuccessfulRescale  28m                horizontal-pod-autoscaler  New size: 8; reason: cpu resource utilization (percentage of request) above target
```
monitor the hpa:
```
# kubectl get hpa -owide -w
NAME      REFERENCE        TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
web-hpa   Deployment/web   0%/50%    2         8         3          3m21s
web-hpa   Deployment/web   80%/50%   2         8         3          5m45s
web-hpa   Deployment/web   80%/50%   2         8         5          6m
web-hpa   Deployment/web   76%/50%   2         8         5          6m16s
web-hpa   Deployment/web   82%/50%   2         8         5          6m46s
web-hpa   Deployment/web   82%/50%   2         8         8          7m1s
web-hpa   Deployment/web   78%/50%   2         8         8          7m17s
web-hpa   Deployment/web   77%/50%   2         8         8          7m47s
```
### decrease the CPU usage
```
kubectl delete -f hpa-load.yaml
```
monitor the HPA events:
```
# kubectl describe hpa web-hpa
Events:
  Type     Reason             Age                From                       Message
  ----     ------             ----               ----                       -------
  Normal   SuccessfulRescale  29m                horizontal-pod-autoscaler  New size: 5; reason: cpu resource utilization (percentage of request) above target
  Normal   SuccessfulRescale  28m                horizontal-pod-autoscaler  New size: 8; reason: cpu resource utilization (percentage of request) above target
  Normal   SuccessfulRescale  5m28s              horizontal-pod-autoscaler  New size: 4; reason: All metrics below target
  Normal   SuccessfulRescale  4m57s              horizontal-pod-autoscaler  New size: 2; reason: All metrics below target
```
