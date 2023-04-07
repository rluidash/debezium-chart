# debezium-chart
This is a debezium helm chart using strimiz-kafka-operator to create the following resources

	1. strimzi kafka operator
	2. kafka cluster
	3. zookeeper
	4. debezium kafka connector
	5. mysql server 

This setup is based on the documentation for "[Deploying Debzium on Kubernetes](https://debezium.io/documentation/reference/stable/operations/kubernetes.html)"

The strimzi-kakfka-operator version being used is 0.34.0
The kafka connect version is 3.3.2
The dbezium-connector-mysql version is [2.1.3.Final](https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/2.1.3.Final/debezium-connector-mysql-2.1.3.Final-plugin.tar.gz)
 

## How to use or install this helm chart

1. Add the strimzi helm repo

 ```
  helm repo add strimzi https://strimzi.io/charts/
 ```

2. Update repo and run the chart dep update

 ```
  helm repo update
  helm dep up
 ```

3. install the chart 

	install the chart from local git cloned repo "debezium-chart"

	 ```
	  helm upgrade -i debezium ./debezium-chart
	 ```
 
	 OR
 
	install the chart from public oci repo

	 ```
	  helm install debezium oci://public.ecr.aws/a5t5z1t5/debezium --devel
	 ```
 
4. Checking the created resources

	```
	silui@SILUI-M-Q8CG debezium-chart % kubectl get all 
	NAME                                                    READY   STATUS    RESTARTS      AGE
	pod/debezium-cluster-entity-operator-7fddf57b9c-w9wx2   3/3     Running   0             51m
	pod/debezium-cluster-kafka-0                            1/1     Running   0             52m
	pod/debezium-cluster-zookeeper-0                        1/1     Running   0             52m
	pod/debezium-connect-cluster-connect-54df6b8d69-5s45k   1/1     Running   3 (52m ago)   52m
	pod/mysql-88f5d6fd-bfp8w                                1/1     Running   0             52m
	pod/strimzi-cluster-operator-675749c788-p2dww           1/1     Running   0             52m
	
	NAME                                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
	service/debezium-cluster-kafka-0                    NodePort    10.105.158.231   <none>        9094:30078/TCP                        52m
	service/debezium-cluster-kafka-bootstrap            ClusterIP   10.110.155.241   <none>        9091/TCP,9092/TCP,9093/TCP            52m
	service/debezium-cluster-kafka-brokers              ClusterIP   None             <none>        9090/TCP,9091/TCP,9092/TCP,9093/TCP   52m
	service/debezium-cluster-kafka-external-bootstrap   NodePort    10.102.173.147   <none>        9094:31490/TCP                        52m
	service/debezium-cluster-zookeeper-client           ClusterIP   10.106.41.129    <none>        2181/TCP                              52m
	service/debezium-cluster-zookeeper-nodes            ClusterIP   None             <none>        2181/TCP,2888/TCP,3888/TCP            52m
	service/debezium-connect-cluster-connect-api        ClusterIP   10.105.49.78     <none>        8083/TCP                              52m
	service/kubernetes                                  ClusterIP   10.96.0.1        <none>        443/TCP                               31h
	service/mysql                                       ClusterIP   None             <none>        3306/TCP                              52m
	
	NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
	deployment.apps/debezium-cluster-entity-operator   1/1     1            1           51m
	deployment.apps/debezium-connect-cluster-connect   1/1     1            1           52m
	deployment.apps/mysql                              1/1     1            1           52m
	deployment.apps/strimzi-cluster-operator           1/1     1            1           52m
	
	NAME                                                          DESIRED   CURRENT   READY   AGE
	replicaset.apps/debezium-cluster-entity-operator-7fddf57b9c   1         1         1       51m
	replicaset.apps/debezium-connect-cluster-connect-54df6b8d69   1         1         1       52m
	replicaset.apps/mysql-88f5d6fd                                1         1         1       52m
	replicaset.apps/strimzi-cluster-operator-675749c788           1         1         1       52m
	```

 
5. Verify the setup

 start watching  kafka topic: msyql.inventory.customers
 ```
  kubectl rn -it --rm --image=quay.io/debezium/tooling:1.2  --restart=Never watcher -- kcat -b debezium-cluster-kafka-bootstrap:9092 -C -o beginning -t mysql.inventory.customers
  ```
  
 connect to mysql server
  
  ```
  kubectl run -it --rm --image=mysql:8.0 --restart=Never --env MYSQL_ROOT_PASSWORD=debezium mysqlterm -- mysql -hmysql -P3306 -uroot -pdebezium
  ```
  
 make change on customers table
 
  ```
   update customers set first_name="Sally Marie" where id=1001;
  ```
  
 Observe change events on the watching kafka topic
 
 
 
6. Reference

   ```
	https://github.com/strimzi/strimzi-kafka-operator/tree/main/helm-charts/helm3/strimzi-kafka-operator
	https://strimzi.io/docs/operators/latest/deploying.html#creating-new-image-from-base-str
	https://strimzi.io/blog/2020/01/27/deploying-debezium-with-kafkaconnector-resource/
   ```
   

