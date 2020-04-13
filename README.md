# microk8s-gitlab
[gitlab-ce](https://github.com/helm/charts/tree/master/stable/gitlab-ce)

## 1. Setup
```sh
$ git clone https://github.com/helm/charts.git
$ mkdir helm
$ mv charts helm/

$ cd microk8s-gitlab/
$ cp -r ../helm/charts/stable/gitlab-ce ./

$ kubectl create ns gitlab
$ vi gitlab-ce/templates/deployment.yaml	# set version to 'v1'
$ helm dependency update ./gitlab-ce
$ vi gitlab-ce/templates/deployment.yaml 
$ vi gitlab-ce/charts/postgresql/templates/deployment.yaml 
$ vi gitlab-ce/charts/redis/templates/deployment.yaml

$ helm install --name hela-gitlab --namespace gitlab --set externalUrl=http://hela.avisow.com/,gitlabRootPassword=pass1234 ./gitlab-ce
NAME:   hela-gitlab
LAST DEPLOYED: Sun Apr 12 12:27:17 2020
NAMESPACE: gitlab
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                   AGE
hela-gitlab-gitlab-ce  1s

==> v1/Deployment
NAME                    AGE
hela-gitlab-gitlab-ce   0s
hela-gitlab-postgresql  0s
hela-gitlab-redis       0s

==> v1/PersistentVolumeClaim
NAME                        AGE
hela-gitlab-gitlab-ce-data  1s
hela-gitlab-gitlab-ce-etc   1s
hela-gitlab-postgresql      1s
hela-gitlab-redis           1s

==> v1/Pod(related)
NAME                                     AGE
hela-gitlab-gitlab-ce-594d5f87dc-bc5qd   0s
hela-gitlab-postgresql-564bc9979b-7k6nq  0s
hela-gitlab-redis-6cd84798cf-7fl6t       0s

==> v1/Secret
NAME                    AGE
hela-gitlab-gitlab-ce   1s
hela-gitlab-postgresql  1s
hela-gitlab-redis       1s

==> v1/Service
NAME                    AGE
hela-gitlab-gitlab-ce   0s
hela-gitlab-postgresql  1s
hela-gitlab-redis       1s


NOTES:

##############################################################################
This chart has been deprecated in favor of the official GitLab chart:
http://docs.gitlab.com/ce/install/kubernetes/gitlab_omnibus.html
##############################################################################

1. Get your GitLab URL by running:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc -w hela-gitlab-gitlab-ce'

  export SERVICE_IP=$(kubectl get svc --namespace gitlab hela-gitlab-gitlab-ce -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
  echo http://$SERVICE_IP/

2. Login as the root user:

  Username: root
  Password: pass1234


3. Point a DNS entry at your install to ensure that your specified
   external URL is reachable:

   http://hela.avisow.com/

# Monitoring
$ kubectl get all -n gitlab
$ helm delete --purge hela-gitlab

$ helm repo add gitlab https://charts.gitlab.io/
"gitlab" has been added to your repositories
$ helm repo update

$ helm upgrade --install hela-gitlab --namespace gitlab gitlab/gitlab \
--timeout 600 \
--set global.edition=ce \
--set global.hosts.domain=hela.avisow.com \
--set global.hosts.externalIP=192.168.1.101 \
--set certmanager-issuer.email=ryan.wibawa@gmail.com \
--set certmanager.rbac.create=false \
--set nginx-ingress.rbac.createRole=false \
--set prometheus.rbac.create=false \
--set gitlab-runner.rbac.create=false
```

## 2. With `helm3`:
[omnibus-gitlab](https://gitlab.com/gitlab-org/omnibus-gitlab)

### Find the docker image that works:
The docker image version that works is `gitlab/gitlab-ce:12.8.7-ce.0`.
```sh
$ docker run -d -p 1180:80 -p 11443:443 -p 1122:22 --name hela-gitlab --network mynetwork gitlab/gitlab-ce:12.8.7-ce.0

$ docker tag gitlab/gitlab-ce:12.8.7-ce.0 localhost:32000/gitlab/gitlab-ce:12.8.7-ce.0
$ docker push localhost:32000/gitlab/gitlab-ce:12.8.7-ce.0
```

### Try it with helm3:
```sh
$ helm3 install hela-gitlab --namespace gitlab --set externalUrl=http://hela.avisow.com/,gitlabRootPassword=pass1234 ./gitlab-ce
WARNING: This chart is deprecated
NAME: hela-gitlab
LAST DEPLOYED: Sun Apr 12 19:58:08 2020
NAMESPACE: gitlab
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
##############################################################################
This chart has been deprecated in favor of the official GitLab chart:
http://docs.gitlab.com/ce/install/kubernetes/gitlab_omnibus.html
##############################################################################

1. Get your GitLab URL by running:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc -w hela-gitlab-gitlab-ce'

  export SERVICE_IP=$(kubectl get svc --namespace gitlab hela-gitlab-gitlab-ce -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
  echo http://$SERVICE_IP/

2. Login as the root user:

  Username: root
  Password: pass1234


3. Point a DNS entry at your install to ensure that your specified
   external URL is reachable:

   http://hela.avisow.com/

$ kubectl -n gitlab get pods
NAME                                      READY   STATUS             RESTARTS   AGE
hela-gitlab-gitlab-ce-59d677f668-pwtnk    0/1     CrashLoopBackOff   14         62m
hela-gitlab-postgresql-564bc9979b-bfrvf   1/1     Running            0          62m
hela-gitlab-redis-6cd84798cf-hgq7r        1/1     Running            0          62m

$ helm3 uninstall hela-gitlab -n gitlab

```
