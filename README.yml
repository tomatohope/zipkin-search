#### introduction
Zipkin是一个分布式链路跟踪系统，可以采集时序数据来协助定位延迟等相关问题。数据可以存储在cassandra,MySQL,ES,mem中
\\ https://zipkin.io/pages/architecture.html


####architecture

instrumention --------> transport------>collector------->storage
-         -                                           -     -
-         -                                          -      -
-         -                                        -        -
span   repoter                                 WEBUI      search



####prepare
##prepare images
es      \\ zipkin db storage
zipkin  \\ api link tracking

docker pull openzipkin/zipkin
dpcker pull elasticsearch:7.6.2

##prepare start es server by docker


mkdir -p 777 /data/elasticsearch
chmod -R 777 /data/elasticsearch

vi elasticsearch.yml

cluster.name: "docker-cluster"
network.host: 0.0.0.0
xpack.security.enabled: true


docker run -d \
    --user root \
    -p 192.168.0.121:9200:9200 \
    -p 9300:9300 \
    --log-driver json-file \
    --log-opt max-size=10m \
    --log-opt max-file=3 \
    --name elasticsearch \
    --restart=always \
    -e "discovery.type=single-node" \
    -v /data/elasticsearch:/usr/share/elasticsearch/data \
   -v /root/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
    elasticsearch:7.6.2

#set es pass
docker exec -it elasticsearch /bin/bash
cd bin
elasticsearch-setup-passwords interactive
elastic
123456

docker restart elasticsearch


##prepare k8s yml
\\ https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
\\ https://github.com/openzipkin-attic/docker-zipkin


#zipkin.yml

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: zipkin
  namespace: zipkin
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      k8s-app: zipkin
  template:
    metadata:
      labels:
        k8s-app: zipkin
    spec:
      containers:
      - name: zipkin
        imagePullPolicy: IfNotPresent
        image: 192.168.0.121:5000/basic/zipkin
        securityContext:
          capabilities:
            add: ["SYS_PTRACE"]
        env:
        - name: release_environment
          value: "live"
        - name: STORAGE_TYPE
          value: "elasticsearch"
        - name: ES_HOSTS
          value: "192.168.0.121"
        - name: ES_USERNAME
          value: "elastic"
        - name: ES_PASSWORD
          value: "123456"
        ports:
          - containerPort: 9411
            name: zipkin
            protocol: TCP
        resources:
          requests:
            cpu: 200m
            memory: 500Mi
          limits:
            cpu: 500m
            memory: 1Gi
      imagePullSecrets:
        - name: harborsecretname

---
apiVersion: v1
kind: Service
metadata:
  name: zipkin
  namespace: zipkin
  labels:
    k8s-app: zipkin
spec:
  type: NodePort
  ports:
    - port: 9411
      targetPort: 9411
      protocol: TCP
  selector:
    k8s-app: zipkin


####zipkin test
\\ python test
pip install py-zipkin
pip install pyramid
pip install pyramid-zipkin

git clone https://github.com/openzipkin/pyramid_zipkin-example.git
cd pyramid_zipkin-example
vi transport.py
        requests.post(
        #'http://localhost:9411//api/v2/spans',
        'http://192.168.0.118:30235//api/v2/spans',

python setup.py install


nohup python backend.py &
http://localhost:9000/api

nohup python frontend.py &
visit http://localhost:8081/

