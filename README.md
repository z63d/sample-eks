### create cluster
$ eksctl create cluster -f cluster.yaml  


### AWS Load Balancer Controller
https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/deploy/installation/  

$ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.yaml  

$ wget https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases/download/v2.4.6/v2_4_6_full.yaml  
your-cluster-name → sample-eks-cluster  

$ kubectl apply -f v2_4_6_full.yaml  


### create Deployment,Service,Ingress
$ kubectl apply -f nginx.yaml  
$ kubectl apply -f nginx-ingress.yaml  


### Kubernetes Metrics Server
https://github.com/kubernetes-sigs/metrics-server  

$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml  


### Horizontal Pod Autoscaler
※ Metrics Serverが必要
$ kubectl apply -f nginx-hpa.yaml  

stress  
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://nginx; done"  
 or  
$ kubectl run apache-bench -i --tty --rm --image httpd -- /bin/sh  
<!-- $ kubectl exec -it apache-bench -- sh -->  
  while true;  
  do ab -n 10 -c 10 http://nginx.default.svc.cluster.local/ > /dev/null;  
  done  


### Kubernetes Autoscaler
https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler/cloudprovider/aws/examples  

$ wget https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml  

Deploymentのimage versionをclusterに合わせる  
YOUR CLUSTER NAME → sample-eks-cluster  

$ kubectl apply -f cluster-autoscaler-autodiscover.yaml

stress  
replicasを10  
$ kubectl apply -f nginx.yaml  
 or  
$ kubectl scale deployment nginx --replicas 10  


### Container Insights
https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-EKS-quickstart.html  
$ ClusterName='sample-eks-cluster'
RegionName='ap-northeast-1'
FluentBitHttpPort='2020'
FluentBitReadFromHead='Off'
[[ ${FluentBitReadFromHead} = 'On' ]] && FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'
[[ -z ${FluentBitHttpPort} ]] && FluentBitHttpServer='Off' || FluentBitHttpServer='On'
curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluent-bit-quickstart.yaml | sed 's/{{cluster_name}}/'${ClusterName}'/;s/{{region_name}}/'${RegionName}'/;s/{{http_server_toggle}}/"'${FluentBitHttpServer}'"/;s/{{http_server_port}}/"'${FluentBitHttpPort}'"/;s/{{read_from_head}}/"'${FluentBitReadFromHead}'"/;s/{{read_from_tail}}/"'${FluentBitReadFromTail}'"/' | kubectl apply -f -  

CloudWatch  
 Container Insights  
 ロググループ  


### EKSアップデート
clusterとnodeをアップデートする必要がある  

cluster-autoscaler無効化  
$ kubectl scale deployment cluster-autoscaler -n kube-system --replicas 0  

node冗長化  
$ eksctl scale nodegroup --cluster sample-eks-cluster --nodes 2 --name  sample-eks-ng  

pod冗長化  
$ kubectl scale deployment nginx --replicas 2  

$ kubectl apply -f nginx-pdb.yaml  

バージョン確認  
$ kubectl version --short  
Serverをアップデートしたい  

cluster アップデート  
$ eksctl upgrade cluster --name sample-eks-cluster --approve  

アドオンのプログラム アップデート  
$ eksctl utils update-aws-node --cluster sample-eks-cluster --approve  
$ eksctl utils update-coredns --cluster sample-eks-cluster --approve  
$ eksctl utils update-kube-proxy --cluster sample-eks-cluster --approve  

cluster-autoscaler アップデート  
cluster-autoscaler-autodiscover.yaml  
Deployment containers imageをclusterに合わせて変更（v1.23.1）  
$ kubectl apply -f cluster-autoscaler-autodiscover.yaml  

nodegroupアップデート  
cluster.yaml metadata versionを1.23  
nodeGroups nameをバージョン分かる様変更  
$ eksctl create nodegroup -f cluster.yaml  

古いnodegroup削除  
eksctl delete nodegroup -f cluster.yaml --only-missing -approve  

cluster-autoscaler有効化  
$ kubectl scale deployment cluster-autoscaler -n kube-system --replicas 1  

アップデート確認  
cluster  
$ kubectl version --short (Server Version確認)  
node  
$ kubectl get node  


### nodegroup再作成
$ eksctl delete nodegroup --config-file=cluster.yaml --approve  
($ eksctl delete nodegroup -f cluster.yaml --approve)  
$ eksctl create nodegroup --config-file=cluster.yaml  
($ eksctl create nodegroup -f cluster.yaml)  


### delete cluster
$ eksctl delete cluster -f cluster.yaml  

消えない時は以下確認  
セキュリティグループ  
ネットワークインターフェイス  
ロードバランサー  
CloudFormation  

消えているか確認  
VPC  
CloudFormation  
