- 👋 Hi, I’m @rishabhdhing29
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
rishabhdhing29/rishabhdhing29 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker

kind: ConfigMap
apiVersion: v1
metadata:
  name: envoy-proxy-frontend-config
  namespace: default
data:
  envoy.yaml: |-
    static_resources:
      listeners:
      - address:
          socket_address:
            address: 0.0.0.0
            port_value: 8000
        filter_chains:
        - filters:
          - name: envoy.filters.network.http_connection_manager
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
              codec_type: AUTO
              stat_prefix: frontend_ingress_http
              access_log:
                - name: envoy.file_access_log
                  typed_config:
                    "@type": type.googleapis.com/envoy.extensions.access_loggers.file.v3.FileAccessLog
                    path: "/dev/stdout"
              route_config:
                name: local_route
                virtual_hosts:
                - name: service
                  domains:
                  - "*"
                  routes:
                  - match:
                      prefix: "/"
                    route:
                      cluster: frontend_service
              http_filters:
              - name: envoy.filters.http.router
                typed_config: {}
      clusters:
      - name: frontend_service
        connect_timeout: 0.25s
        type: STRICT_DNS
        lb_policy: ROUND_ROBIN
        load_assignment:
          cluster_name: frontend_service
          endpoints:
          - lb_endpoints:
            - endpoint:
                address:
                  socket_address:
                    address: frontend-service.default.svc.cluster.local
                    port_value: 9000
    admin:
      address:
        socket_address:
          address: 0.0.0.0
          port_value: 8081
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: envoy-proxy-frontend
  namespace: default
  labels:
    app: envoy-proxy-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: envoy-proxy-frontend
  template:
    metadata:
      labels:
        app: envoy-proxy-frontend
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8081'
        prometheus.io/path: '/stats/prometheus'
    spec:
      containers:
      - name: envoy-proxy-frontend
        image: envoyproxy/envoy:v1.18.2
        command:
        - "envoy"
        - "-c"
        - "/etc/envoy/envoy.yaml"
        ports:
        - containerPort: 8000
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - name: envoy-proxy-frontend-config # loading the config using config map
          mountPath: /etc/envoy/
      volumes:
      - name: envoy-proxy-frontend-config
        configMap:
          name: envoy-proxy-frontend-config
---
kind: Service
apiVersion: v1
metadata:
  name: envoy-proxy-frontend-service
  namespace: default
  labels:
    app: envoy-proxy-frontend
spec:
  selector:
    app: envoy-proxy-frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  sessionAffinity: None
  type: ClusterIP
  
  
  
  
  
  
  # pinger-all-in-one.yaml
  
  
  
  apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: default
  labels:
    app: frontend
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: localhost/frontend:v1
        env:
        - name: PINGER_BASE_URL
          value: "http://pinger-v1-service:3000"
        - name: DETAILS_BASE_URL
          value: "http://details-service:4000"
        ports:
        - containerPort: 9000
        imagePullPolicy: IfNotPresent # else might try to pull images remotely always

---
# Service for frontend
apiVersion: v1
kind: Service
metadata:
  labels:
    app: frontend
    version: v1
  name: frontend-service
  namespace: default
spec:
  ports:
  - port: 9000
    protocol: TCP
    targetPort: 9000
  selector:
    app: frontend
  sessionAffinity: None
  type: ClusterIP
---
# Deployment for details service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: details
  namespace: default
  labels:
    app: details
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: details
  template:
    metadata:
      labels:
        app: details
    spec:
      containers:
      - name: details
        image: localhost/details:v1
        ports:
        - containerPort: 4000
        imagePullPolicy: IfNotPresent
---
# Service for details
apiVersion: v1
kind: Service
metadata:
  labels:
    app: details
    version: v1
  name: details-service
  namespace: default
spec:
  ports:
  - port: 4000
    protocol: TCP
    targetPort: 4000
  selector:
    app: details
  sessionAffinity: None
  type: ClusterIP
---
# Deployment for pinger app v1
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pinger-v1
  namespace: default
  labels:
    app: pinger
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pinger
      version: v1
  template:
    metadata:
      labels:
        app: pinger
        version: v1
    spec:
      containers:
      - name: pinger
        image: localhost/pinger:v1
        ports:
        - containerPort: 3000
        imagePullPolicy: IfNotPresent
---
# Service for pinger app v1
apiVersion: v1
kind: Service
metadata:
  labels:
    app: pinger
    version: v1
  name: pinger-v1-service
  namespace: default
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 3000
  selector:
    app: pinger
    version: v1
  sessionAffinity: None
  type: ClusterIP
---
# Deployment for pinger app v2
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pinger-v2
  namespace: default
  labels:
    app: pinger
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pinger
      version: v2
  template:
    metadata:
      labels:
        app: pinger
        version: v2
    spec:
      containers:
      - name: pinger
        image: localhost/pinger:v2
        ports:
        - containerPort: 3000
        imagePullPolicy: IfNotPresent
---
# Service for ping app v2
apiVersion: v1
kind: Service
metadata:
  labels:
    app: pinger
    version: v2
  name: pinger-v2-service
  namespace: default
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 3000
  selector:
    app: pinger
    version: v2
  sessionAffinity: None
  type: ClusterIP
---
