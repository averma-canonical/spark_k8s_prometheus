## Setup Instructions

Build a docker based image for the sample application to monitor using prometheus 
```bash
cd app
docker build -t sample-app:local -f Dockerfile .
```
Import the docker image into local microk8s registry.

```bash
docker save sample-app > /tmp/sample-app.tar
microk8s ctr image import /tmp/sample-app.tar
```
Deploy prometheus on kubernetes.

```bash
cd ../kubernetes	
kubectl apply -f prometheus/namespace.yaml &&   kubectl apply -f prometheus/
kubectl apply -f kubernetes/app/
```

You can verify the annotations in sample application to expect prometheus scraping

```bash
kubectl get services app -o jsonpath='{.metadata.annotations}'
```

Now launch a Spark job

```bash
sudo snap install spark-client --edge

spark-client.service-account-registry create --username hello

spark-client.spark-submit \
      --username hello \
      --conf spark.ui.prometheus.enabled=true \
      --conf spark.kubernetes.driver.service.annotation.se7entyse7en.prometheus/scrape=true \
      --conf spark.kubernetes.driver.service.annotation.se7entyse7en.prometheus/path=/metrics/executors/prometheus \
      --conf spark.kubernetes.driver.service.annotation.se7entyse7en.prometheus/port=4040 \
      --class org.apache.spark.examples.SparkPi \
      local:///opt/spark/examples/jars/spark-examples_2.12-3.3.2.jar 100000
```


In a separate shell, setup kubernetes port forwarding on localhost to view the Prometheus dashboard.

```bash
kubectl get deployment --namespace monitoring
kubectl port-forward --namespace monitoring deployment/prometheus 9090:9090
```

Navigate in a browser to **_http://localhost:9090/targets_** to list discovered applications with metrics. You can also add graphs with historical data collected in prometheus.

