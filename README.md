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


### Kubernetes Metrics Server
https://github.com/kubernetes-sigs/metrics-server

$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml


### Horizontal Pod Autoscaler
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
--2023-01-19 18:19:22--  https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
Deploymentのimage versionをclusterに合わせる
<YOUR CLUSTER NAME> → sample-eks-cluster

$ kubectl apply -f cluster-autoscaler-autodiscover.yaml

stress
replicasを10
$ kubectl apply -f nginx.yaml
 or
$ kubectl scale deployment --replicas 10 nginx


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
