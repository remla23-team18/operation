# Operation

Contains the files to orchestrate the entire restaurant review app--including the model service and frontend application.

[Assignment 1](#a1)

[Assignment 2](#a2)

[Assignment 3](#a3)



## Assignment 1 <a name="a1"></a>

### Running

In order to run, use the following command. Omitting the the `-d` makes `docker compose` take over your current shell instead of running as a daemon in the background.

```bash
docker compose up -d
```

To view the logs of a currently-running application use the following command. Omitting `-f` means that `docker` will only print out the current logs and exit, rather than showing logs as they appear.

```bash
docker compose logs
```

To stop everything run 

```bash
docker compose down
```

In case you would like to update the versions of images that the `compose` setup uses, run the following command:

```bash
docker compose pull
```

## Assignment 2 <a name="a2"></a>
Install the prometheus stack using the following command:
```bash
helm repo add prom-repo https://prometheus-community.github.io/helm-charts

helm repo update

helm install myprom prom-repo/kube-prometheus-stack
```

Initiate the prometheus monitoring using the following command. Note that this command always pulls the latest image from GitHub packages.

If you want to test local changes yourself with locally built images, see below for detailed instructions.



```bash
kubectl apply -f remla.yml
```
or deploy with helm:
```bash
helm install remla ./remla_chart/  
```

Access the Prometheus page:
```bash
minikube service myprom-kube-prometheus-sta-prometheus --url
```

Access the Grafana page:
```bash
minikube service myprom-grafana --url
```

The default username and password for Grafana is `admin` and `prom-operator`.

Add the Grafana dashboard by importing the json file `dashboard_setup.json` in the dashboard.

After running `minikube tunnel` in another terminal, you can access the front end page using this link:
```bash
localhost/app
```

Access the metrics served to Prometheus using this link:
```bash
localhost/metrics
```

Now you can play around with the front end page to monitor the metrics in Prometheus.
Remember to search with variable names defined in the script.



### Test with local images

For local test, you need to build the image locally and use the file `remla-local.yml`.


Run this script to allow building new images in minikube registry.
```bash
eval $(minikube -p minikube docker-env)
```

Under `app` directory, run this command to build the image, remember to change the token:
```bash
docker build --build-arg GH_TOKEN=<YOUR_TOKEN> \
      --build-arg REACT_APP_MODEL_SERVICE_URL=http://localhost:8080 -t test_app_k8s .
```     

Under `model-service` directory, run the following command to build the image:
```bash
docker build -t test_web_k8s .
```  

Then you can apply the yaml files to run kubernetes.
```bash
kubectl apply -f remla-local.yml

kubectl apply -f monitoring.yml
```

Reference link: https://medium.com/swlh/how-to-run-locally-built-docker-images-in-kubernetes-b28fbc32cc1d 


## Assignment 3 <a name="a3"></a>

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

kubectl apply -f istio-basic.yml
```

You can run `istioctl analyze` to check if there is any problem with the configuration.

In the current version, the different versions of `app` and `model-service` should be alternating.
The green button is `version1` and the orange button is `version2`. 

Rate limiting using istio:
```bash
kubectl apply -f istio-ratelimit.yml
```
Rate limiting in Istio is implemented using an Envoy filter.The Envoy proxy provides built-in rate limiting filters that can be configured to enforce rate limits on incoming requests. The rate limiting filters are applied to the istio-ingressgateway workload, and enable fine-grained control over request rates, ensuring fair resource allocation, preventing abuse, and protecting the service mesh from disruptions caused by excessive traffic, thereby improving system availability and providing a fair and reliable experience for all users. A limit of 10 requests every 2 minutes has been implemented.


Access the Prometheus and Kiali page:
```bash
istioctl dashboard prometheus

istioctl dashboard kiali
```

In the prometheus page, you should be able to query the app-specific metrics, and use PromQL to aggregate and filter the metrics.
For example, you can use the following commands:

```bash
sum by (version) (total_accuracy)

sum by (version) (text_length_bucket)
```

## Code structure

    .
    ├── model-training                    # Code to pre-process and train the review classifier model
    ├── model-service                     # Flask service to serve the model
    ├── lib                               # Library that the frontend app depends on. Currently no functionality.
    ├── app                               # Frontend app that displays a UI and allows for a user to query the model-service
    └── operation                         # Orchestration code to run the app and service, option to use docker compose, kubernetes, or a full helm chart.
