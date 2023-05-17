# Operation

Contains the files to orchestrate the entire restaurant review app--including the model service and frontend application. Currently this is just a `docker compose`-based setup. Later on `kubernetes` will be used as well.

## Assignment 1

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

## Assignment 2
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



## Code structure

    .
    ├── model-training                    # Code to pre-process and train the review classifier model
    ├── model-service                     # Flask service to serve the model
    ├── lib                               # Library that the frontend app depends on. Currently no functionality.
    ├── app                               # Frontend app that displays a UI and allows for a user to query the model-service
    └── operation                         # Orchestration code to run the app and service, currently just a docker-compose setup.
