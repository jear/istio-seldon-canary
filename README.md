# Canary Roll Out using Seldon-Core and Istio

This folder provides resources to illustrate how to do a canary roll out of one MNIST model to another using the canary pattern where a small amount of traffic is sent to the new model to validate it before sending all traffic to the new model.

There is a [Jupyter Notebook](canary.ipynb) that provides a step by step demo.

    conda create -n canary python=3.6
    conda activate canary
    pip install seldon-core>=0.2.6.1
    jupyter notebook  --no-browser  --allow-root --ip `hostname -i`   --port 8890
    
    kubectl create namespace seldon
    kubectl config set-context $(kubectl config current-context) --namespace=seldon

    kubectl create -f ../../../notebooks/resources/seldon-gateway.yaml
    k get gateways.networking.istio.io 
    k describe gateways.networking.istio.io
    
    kubectl label namespace seldon istio-injection=enabled

    INGRESS_HOST=`kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'`
    INGRESS_PORT=`kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}'`
    ISTIO_GATEWAY=$INGRESS_HOST:$INGRESS_PORT
    echo $ISTIO_GATEWAY

    helm install seldon-core-operator --name seldon-core --repo https://storage.googleapis.com/seldon-charts --set usageMetrics.enabled=true --namespace seldon-system --set istio.enabled=true
    kubectl rollout status statefulset.apps/seldon-operator-controller-manager -n seldon-system

    kubectl apply -f mnist_v1.json
    kubectl rollout status deploy/mnist-deployment-sk-mnist-predictor-73d7608

    helm install ../../../helm-charts/seldon-core-loadtesting --name loadtest  \
    --namespace seldon \
    --repo https://storage.googleapis.com/seldon-charts \
    --set locust.script=mnist_rest_locust.py \
    --set locust.host=http://$ISTIO_GATEWAY \
    --set rest.pathPrefix=/seldon/seldon/mnist-classifier \
    --set oauth.enabled=false \
    --set oauth.key=oauth-key \
    --set oauth.secret=oauth-secret \
    --set locust.hatchRate=1 \
    --set locust.clients=1 \
    --set loadtest.sendFeedback=1 \
    --set locust.minWait=0 \
    --set locust.maxWait=0 \
    --set replicaCount=1 \
    --set data.size=784
