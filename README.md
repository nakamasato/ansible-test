# Ansible

## Ping

```
ansible localhost -m ping
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Deploy docker stack with Ansible

### 1. Docker Compose

#### Deploy

```
docker-compose -f compose/docker-compose.yml up
```

#### Check

```
curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### Clean up

```
docker-compose -f compose/docker-compose.yml down
```

### 2. Docker stack

#### Deploy

```
docker stack deploy -c compose/docker-compose.yml stack-test
Creating network stack-test_default
Creating service stack-test_nginx
```

#### Check

##### Stack

```
docker stack ls
NAME                SERVICES            ORCHESTRATOR
stack-test          1                   Swarm
```

##### Container

```
docker ps | grep nginx
76bea0739e67        nginx:latest           "nginx -g 'daemon of…"   16 seconds ago      Up 15 seconds       80/tcp              stack-test_nginx.1.uyzlqsgqit8a32vgnv109ijhd
```
##### Service

```
docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
70wu1ybdrvex        stack-test_nginx    replicated          1/1                 nginx:latest        *:8080->80/tcp
```

##### Nginx

```
curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### Clean up

```
docker stack rm stack-test
Removing service stack-test_nginx
Removing network stack-test_default
```

### 3. Ansible

#### Deploy

```
ansible localhost -m docker_stack -a "name=mystack compose=compose/docker-compose.yml"
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | CHANGED => {
    "changed": true,
    "rc": 0,
    "stack_spec_diff": "{\"$replace\": {\"mystack_nginx\": {\"Name\": \"mystack_nginx\", \"Labels\": {\"com.docker.stack.image\": \"nginx\", \"com.docker.stack.namespace\": \"mystack\"}, \"TaskTemplate\": {\"ContainerSpec\": {\"Image\": \"nginx:latest@sha256:30dfa439718a17baafefadf16c5e7c9d0a1cde97b4fd84f63b69e13513be7097\", \"Labels\": {\"com.docker.stack.namespace\": \"mystack\"}, \"Privileges\": {\"CredentialSpec\": null, \"SELinuxContext\": null}, \"StopGracePeriod\": 10000000000, \"DNSConfig\": {}, \"Isolation\": \"default\"}, \"Resources\": {}, \"RestartPolicy\": {\"Condition\": \"any\", \"Delay\": 5000000000, \"MaxAttempts\": 0}, \"Placement\": {\"Platforms\": [{\"Architecture\": \"amd64\", \"OS\": \"linux\"}, {\"OS\": \"linux\"}, {\"Architecture\": \"arm64\", \"OS\": \"linux\"}, {\"Architecture\": \"386\", \"OS\": \"linux\"}, {\"Architecture\": \"ppc64le\", \"OS\": \"linux\"}, {\"Architecture\": \"s390x\", \"OS\": \"linux\"}]}, \"Networks\": [{\"Target\": \"xozs6ktbaof0ypa249eorfcao\", \"Aliases\": [\"nginx\"]}], \"ForceUpdate\": 0, \"Runtime\": \"container\"}, \"Mode\": {\"Replicated\": {\"Replicas\": 1}}, \"UpdateConfig\": {\"Parallelism\": 1, \"FailureAction\": \"pause\", \"Monitor\": 5000000000, \"MaxFailureRatio\": 0, \"Order\": \"stop-first\"}, \"RollbackConfig\": {\"Parallelism\": 1, \"FailureAction\": \"pause\", \"Monitor\": 5000000000, \"MaxFailureRatio\": 0, \"Order\": \"stop-first\"}, \"EndpointSpec\": {\"Mode\": \"vip\", \"Ports\": [{\"Protocol\": \"tcp\", \"TargetPort\": 80, \"PublishedPort\": 8080, \"PublishMode\": \"ingress\"}]}}}}",
    "stderr": "",
    "stderr_lines": [],
    "stdout": "Creating network mystack_default\nCreating service mystack_nginx\n",
    "stdout_lines": [
        "Creating network mystack_default",
        "Creating service mystack_nginx"
    ]
}
```

#### Check

##### Stack

```
docker stack ls
NAME                SERVICES            ORCHESTRATOR
mystack             1                   Swarm
```

##### Nginx

```
curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### Clean up

```
ansible localhost -m docker_stack -a "name=mystack compose=compose/docker-compose.yml state=absent"
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | CHANGED => {
    "changed": true,
    "err": "",
    "msg": "Removing service mystack_nginx\nRemoving network mystack_default\n",
    "rc": 0,
    "stderr": "",
    "stderr_lines": [],
    "stdout": "Removing service mystack_nginx\nRemoving network mystack_default\n",
    "stdout_lines": [
        "Removing service mystack_nginx",
        "Removing network mystack_default"
    ]
}
```

```
docker stack ls
NAME                SERVICES            ORCHESTRATOR
```

### 4. Ansible Playbook

#### Deploy

```
ansible-playbook docker-stack.yml --extra-vars="DOMAIN=test PROJECT=myproject STACK=mystack" -vvv
```

#### Check

```
docker stack ls
NAME                SERVICES            ORCHESTRATOR
mystack             1                   Swarm
```

#### Other commands

- `ansible-playbook --check docker-stack.yml --extra-vars="DOMAIN=test PROJECT=myproject STACK=mystack"`
- `ansible-playbook --diff docker-stack.yml --extra-vars="DOMAIN=test PROJECT=myproject STACK=mystack"`

#### Clean up

```
docker stack rm mystack
Removing service mystack_nginx
Removing network mystack_default
```
