# gloo route table demo

#### Requirements:
[k8s](https://kubernetes.io/), [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) and [glooctl](https://docs.solo.io/gloo-edge/latest/installation/gateway/kubernetes/#install-command-line-tool-cli).

##### Step 1 / install gateway

```
glooctl install gateway
```

Verify your Installation:

```
kubectl get all -n gloo-system
```

```
NAME                                 READY   STATUS    RESTARTS   AGE
pod/discovery-58b5dcfb6d-n2k9p       1/1     Running   2          24s
pod/gateway-65868445b9-d4fgt         1/1     Running   2          24s
pod/gateway-proxy-564b5db48f-6dqjh   1/1     Running   0          24s
pod/gloo-65b4648df4-jk7bz            1/1     Running   2          24s

NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
service/gateway         ClusterIP      10.101.97.50     <none>        443/TCP                               24s
service/gateway-proxy   LoadBalancer   10.110.202.153   localhost     80:31316/TCP,443:30527/TCP            24s
service/gloo            ClusterIP      10.100.181.128   <none>        9977/TCP,9976/TCP,9988/TCP,9979/TCP   24s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/discovery       1/1     1            1           24s
deployment.apps/gateway         1/1     1            1           24s
deployment.apps/gateway-proxy   1/1     1            1           24s
deployment.apps/gloo            1/1     1            1           24s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/discovery-58b5dcfb6d       1         1         1       24s
replicaset.apps/gateway-65868445b9         1         1         1       24s
replicaset.apps/gateway-proxy-564b5db48f   1         1         1       24s
replicaset.apps/gloo-65b4648df4            1         1         1       24s
```

##### Step 2 / create service

```
kubectl apply -f services/demo-service.yaml 
```

Verify your services:

```
glooctl get virtualservice -n gloo-system
```

```
+-----------------+--------------+-----------+------+---------+-----------------+--------+
| VIRTUAL SERVICE | DISPLAY NAME |  DOMAINS  | SSL  | STATUS  | LISTENERPLUGINS | ROUTES |
+-----------------+--------------+-----------+------+---------+-----------------+--------+
| localhost       |              | localhost | none | Warning |                 | / ->   |
+-----------------+--------------+-----------+------+---------+-----------------+--------+
```

##### Step 3 / create route tables

```
kubectl apply -f route-tables/
```

Verify your rotes:

```
glooctl get routetable -n gloo-system
```

```
+-------------+------------------------+----------+
| ROUTE TABLE |         ROUTES         |  STATUS  |
+-------------+------------------------+----------+
| bar         | /bar -> google.com     | Accepted |
|             | /foo/bar -> google.com |          |
| foo         | /foo -> yahoo.com      | Accepted |
+-------------+------------------------+----------+
```

#### Demo
 
##### One

```
curl -v http://localhost:80/foo
```

Should redirect to yahoo.com

```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 80 (#0)
> GET /foo HTTP/1.1
> Host: localhost
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< location: http://yahoo.com/foo
< date: Sun, 24 Jan 2021 16:41:56 GMT
< server: envoy
< content-length: 0
< 
* Connection #0 to host localhost left intact
* Closing connection 0
```

##### Two

```
curl -v http://localhost:80/bar
```

Should redirect to google.com

```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 80 (#0)
> GET /bar HTTP/1.1
> Host: localhost
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< location: http://google.com/bar
< date: Sun, 24 Jan 2021 16:43:07 GMT
< server: envoy
< content-length: 0
< 
* Connection #0 to host localhost left intact
* Closing connection 0
```

##### Three

```
curl -v http://localhost:80/foo/bar
```

Should redirect to google.com

```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 80 (#0)
> GET /foo/bar HTTP/1.1
> Host: localhost
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< location: http://google.com/foo/bar
< date: Sun, 24 Jan 2021 16:43:51 GMT
< server: envoy
< content-length: 0
< 
* Connection #0 to host localhost left intact
* Closing connection 0
```
