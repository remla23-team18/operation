# Operation

Contains the files to orchestrate the entire restaurant review app--including the model service and frontend application.

The following shows the steps to run the app:


Set up your minikube cluster:
```bash
minikube start --memory=16384 --cpus=4

minikube addons enable ingress

minikube tunnel
```

Set up the istio service mesh:
```bash
istioctl install

kubectl label ns default istio-injection=enabled

kubectl apply -f istio_addon/.

helm install remla remla_chart/

```



Deployment for Istio is included in the helm chart, including the basic setup and the rate limiting.
Rate limiting in Istio is implemented using an Envoy filter. The Envoy proxy provides built-in rate limiting filters that can be configured to enforce rate limits on incoming requests. 
A limit of 10 requests every 2 minutes has been implemented.

You can run `istioctl analyze` to check if there is any problem with the configuration.
In the current version, we have one version of `app` and two versions of `model-service`.


Access the Prometheus and Grafana page:
```bash
istioctl dashboard prometheus

istioctl dashboard grafana
```

In the prometheus page, you should be able to query the app-specific metrics, and use PromQL to aggregate and filter the metrics.
For example, you can use the following commands:

```bash
sum by (version) (total_accuracy)

sum by (version) (text_length_bucket)
```

In the Grafana page, you can use the dashboard to monitor the metrics by importing the `dashboard.json` file.

