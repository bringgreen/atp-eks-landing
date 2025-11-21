## 사전 작업
- helm 도구 설치
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
```
- helm repo 등록
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

```


## ingress-nginx-controller 설치

```
kubectl create namespace ingress-nginx

helm install ingress-nginx ingress-nginx/ingress-nginx  \
--namespace ingress-nginx \
--set controller.ingressClassResource.name=nginx \
--set controller.ingressClassResource.controllerValue="example.com/ingress-nginx" \
--set controller.ingressClassResource.enabled=true \
--set controller.ingressClassByName=true \--set controller.progressDeadlineSeconds=600

```

## deployment, service 설치
```
kubectl apply -f deploy-path1.yaml 
kubectl apply -f deploy-path2.yaml

kubectl apply -f svc-nodeapp1.yaml
kubectl apply -f svc-nodeapp2.yaml
```

## CLB 주소를 ingress-nginx.yaml 파일에 설정
 - host 값에 CLB 주소를 설정
 - kubectl apply -f ingress-nginx.yaml 또는 
 - kubectl apply -f ingress-nginx-path-rewrite.yaml
 
## 테스트
 - ingress-nginx.yaml을 적용했을 때
   * curl [CLB주소]/path1
   * curl [CLB주소]/path2
 - ingress-nginx-path-rewrite.yaml 을 적용했을 때
   * curl [CLB주소]/path1/abc
   * curl [CLB주소]/path2/def
  
## 테스트 종료후 리소스 정리 
```
kubectl delete -f ingress-nginx.yaml
- 또는 -
kubectl delete -f ingress-nginx-path-rewrite.yaml

kubectl delete -f svc-nodeapp1.yaml
kubectl delete -f svc-nodeapp2.yaml

kubectl delete -f deploy-path1.yaml
kubectl delete -f deploy-path2.yaml

helm uninstall ingress-nginx -n ingress-nginx
kubectl delete namespace ingress-nginx

```
   