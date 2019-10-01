# istio-seldon-canary
    # https://docs.seldon.io/projects/seldon-core/en/latest/examples/istio_canary.html

    kubectl create -f ../../../notebooks/resources/seldon-gateway.yaml
    k get gateways.networking.istio.io 
    NAME             AGE
    seldon-gateway   2m9s

    kubectl label namespace seldon istio-injection=enabled
    
    INGRESS_HOST=kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
    INGRESS_PORT=kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}'
    ISTIO_GATEWAY=INGRESS_HOST[0]+":"+INGRESS_PORT[0]
    
    
