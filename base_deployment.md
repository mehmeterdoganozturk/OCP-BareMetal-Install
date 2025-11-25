# OpenShift Hello World Deployment Örneği

Bu doküman, OpenShift üzerinde basit bir **Hello World** uygulaması
çalıştırmak için gerekli olan **Deployment**, **Service** ve **Route**
örneklerini içermektedir. Proje adı **test** olarak hazırlanmıştır.

------------------------------------------------------------------------

## 1. Deployment

Aşağıdaki Deployment, `hashicorp/http-echo` imajını kullanarak 5678
portundan "Hello World" çıktısı üretir.

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
  namespace: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
        - name: hello-world
          image: hashicorp/http-echo:0.2.3
          args:
            - "-text=Hello World"
          ports:
            - containerPort: 5678
```

------------------------------------------------------------------------

## 2. Service

Bu Service, Deployment'taki Pod'u 80 portundan erişilebilir hale
getirir.

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world-svc
  namespace: test
spec:
  selector:
    app: hello-world
  ports:
    - name: http
      port: 80
      targetPort: 5678
```

------------------------------------------------------------------------

## 3. Route

Route, uygulamayı dış dünyaya açar.

``` yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: hello-world
  namespace: test
spec:
  to:
    kind: Service
    name: hello-world-svc
  port:
    targetPort: http
  tls:
    termination: edge
```

------------------------------------------------------------------------

## Uygulama Komutları

``` bash
oc apply -f deployment.yaml
oc apply -f service.yaml
oc apply -f route.yaml
```

Route'un adresini görmek için:

``` bash
oc get route -n test
```

Tarayıcıdan Route URL'sine eriştiğinizde **Hello World** çıktısını
görürsünüz.
