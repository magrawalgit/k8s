
https://docs.bitnami.com/tutorials/process-data-spark-kubernetes/

https://itnext.io/kafka-on-kubernetes-the-strimzi-way-part-1-bdff3e451788


-------OCI

oci ce cluster create-kubeconfig --cluster-id ocid1.cluster.oc1.ap-mumbai-1.aaaaaaaailrkzdaq2qjst4vboorlp6zgpqow7q5ay55fm4r6ycoe7xpcvlca --file $HOME/.kube/config --region ap-mumbai-1 --token-version 2.0.0 

to suppress these warnings and to remove those permissions
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/mataprasad/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/mataprasad/.kube/config

ls -al ~/.kube/config
chmod o-r ~/.kube/config
chmod g-r ~/.kube/config

---------SPARK BITNAMI installation with PVC
helm repo add stable https://charts.helm.sh/stable
helm install nfs stable/nfs-server-provisioner \
  --set persistence.enabled=true,persistence.size=5Gi
  
kubectl apply -f https://raw.githubusercontent.com/mata1234/k8s/master/spark-pvc.yml

docker run -v /tmp:/tmp -it bitnami/spark -- find /opt/bitnami/spark/examples/jars/ -name spark-examples* -exec cp {} /tmp/my.jar \;

echo "how much wood could a woodpecker chuck if a woodpecker could chuck wood" > /tmp/test.txt

kubectl cp /tmp/my.jar spark-data-pod:/data/my.jar
kubectl cp /tmp/test.txt spark-data-pod:/data/test.txt

kubectl exec -it spark-data-pod -- ls -al /data

kubectl delete pod spark-data-pod
------------
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install spark bitnami/spark -f https://raw.githubusercontent.com/mata1234/k8s/master/spark-chart.yml

##
export SERVICE_IP=$(kubectl get --namespace default svc spark-master-svc -o jsonpath="{.status.loadBalancer.ingress[0]['ip', 'hostname'] }")
  echo http://$SERVICE_IP:80
  
Run the commands below to obtain the master IP and submit your application.

  export EXAMPLE_JAR=$(kubectl exec -ti --namespace default spark-worker-0 -- find examples/jars/ -name 'spark-example*\.jar' | tr -d '\r')
  export SUBMIT_IP=$(kubectl get --namespace default svc spark-master-svc -o jsonpath="{.status.loadBalancer.ingress[0]['ip', 'hostname'] }")

  kubectl run --namespace default spark-client --rm --tty -i --restart='Never' \
    --image docker.io/bitnami/spark:3.1.1-debian-10-r68 \
    -- spark-submit --master spark://$SUBMIT_IP:7077 \
    --deploy-mode cluster \
    --class org.apache.spark.examples.SparkPi \
    $EXAMPLE_JAR 1000
##
    
kubectl run --namespace default spark-client --rm --tty -i --restart='Never' \
    --image docker.io/bitnami/spark:3.0.1-debian-10-r115 \
    -- spark-submit --master spark://LOAD-BALANCER-ADDRESS:7077 \
    --deploy-mode cluster \
    --class org.apache.spark.examples.JavaWordCount \
   /data/my.jar /data/test.txt
   
kubectl run --namespace default spark-client --rm --tty -i --restart='Never' \
    --image docker.io/bitnami/spark:3.0.1-debian-10-r115 \
    -- spark-submit --master spark://129.151.45.166:7077 \
    --deploy-mode cluster \
    --class org.apache.spark.examples.JavaWordCount \
   /data/my.jar /data/test.txt
   
kubectl get pods -o wide | grep WORKER-NODE-ADDRESS
kubectl get pods -o wide | grep 10.244.0.133

kubectl exec -it WORKER-POD-NAME -- bash
kubectl exec -it spark-worker-0 -- bash

cd /opt/bitnami/spark/work
cat SUBMISSION-ID/stdout

exit
--------------Kafka strimzi installation
helm repo add strimzi https://strimzi.io/charts/
helm install strimzi-kafka strimzi/strimzi-kafka-operator

kubectl get pods

kubectl get crd | grep strimzi

kubectl get crd kafkas.kafka.strimzi.io -o jsonpath="{.spec.versions[*].name}{'\n'}"


kubectl apply -f https://raw.githubusercontent.com/mata1234/k8s/master/kafka-pvc.yml

to delete deployment - helm uninstall strimzi-kafka
