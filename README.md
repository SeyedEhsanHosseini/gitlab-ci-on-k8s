
## Step 1: (GITLAB-CE DEPLOYMENT)

#### gitlab-deployment.yml:

```
kubectl apply -f gitlab/gitlab.yml
```

*** This stage will take about 40 minutes ***

#### Change gitlab default configuration for reducing resource usage:

```
kubectl exec -it gitlab-deployment-7f5d55d69-6lw4q -- vi /etc/gitlab/gitlab.rb
```

###### In config file uncomment below lines and edit them like this: 

```
postgresql['shared_buffers'] = "256MB"

prometheus_monitoring['enable'] = false 

sidekiq['max_concurrency'] = 2 
```
###### Apply new config
```
kubectl exec -it gitlab-deployment-7f5d55d69-6lw4q -- gitlab-ctl reconfigure
```

##### Get root password of gitlab :

```
kubectl exec -it gitlab-deployment-7f5d55d69-6lw4q -- cat /etc/gitlab/initial_root_password | grep Password:
```
###### Output:

```
Password: qKIoOdGl4tHaI83qU/3bsBN6emN/eZz8GRncZCEgKw8=
```
##### Get Node IP

```
kubectl describe pod <pod-name> | grep "Node:"
```
##### Output: 
```
Node:             node002.cluster.local/192.168.56.13
```
### Access Gitlab UI from browser:

```
http://192.168.56.13:30080
```
##### 3- Sign in with :

```
Username: root

Password: <password-from-previous-step>

```

###### On gitlab UI:

On the top bar, select Main menu > Admin.
On the left sidebar, select Settings > Network.
Expand Outbound requests.
Select the Allow requests to the local network from web hooks and services checkbox.
Save changes.



###### On gitlab UI, Create a new project:


create a blank project and name it "sample-project"



## Install Gitlab Runner using Helm (recommended)

From a terminal, connect to your cluster and run this command. The token is included in the command.

```
sudo snap install helm --classic
```

```
helm repo add gitlab https://charts.gitlab.io
```

#### Get these to value from Settings > CI/CD > Runners > Specific Runners

```
Register the runner with this URL:
http://192.168.56.13:30080/ 

And this registration token:
GR1348941gpsJo71bnku75uVXiCkN 

```
#### Label one of nodes
```
kubectl label nodes node003.cluster.local role=gitlabrunner
```

#### Install gitlab runner 
```
helm upgrade --install --namespace gitlab-runner --create-namespace gitlab-runner -f gitlab-runner/values.yaml gitlab/gitlab-runner --set nodeSelector.role=gitlabrunner
```



#### Build stage Environment variables

##### Navigate to your project's Settings > CI/CD > Environment variables page
##### and add the following ones (replace them with your current values, of course):
##### !!! Only set this variables if you will upload the image to DockerHub !!!

```
CI_REGISTRY_USER: dockerhubuser (DockerHub Username)
```
```
CI_REGISTRY_PASSWORD: dockerhubpassword (DockerHub Password) 
```
```
CI_REGISTRY: docker.io (By default DockerHub registy url  is docker.io, it must be set) 
```
```
CI_REGISTRY_IMAGE: dockerhubuser/projectname (Registy image tag from dockerhub repostory created after this step)
```
****************************************************************************************


###### Add remote repository:

```
cd html-nginx/

rm -rf .git

git init --initial-branch=main

git remote add origin <http://192.168.56.13:30080>/root/sample-project.git

git add .

git commit -m "Initial commit"

git push -u origin main

Input your username and password from step 2-2

```


#### Apply "gitlab-service-account.yaml" 

```
kubectl apply -f gitlab-service-account.yaml

```

#### Go to Settings -> CI / CD -> Environment variables:

#### Navigate to your project's Settings > CI/CD > Environment variables page
#### and add the following ones (replace them with your current values, of course):


###### 1: CERTIFICATE_AUTHORITY_DATA: This is the CA configuration for the Kubernetes cluster 

###### *** Copy and paste this in "certificate-authority-data.txt" file ***

###### Get it's value by running:
```
kubectl config view --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' |base64  -d
```
```
-----BEGIN CERTIFICATE-----
MIIC/jCCAeagAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl
zIpwoe0NLaM9Jcv6V3tJPgiTCbM42JVD1pPCYwqxTP1F74JfdG2BWqKNCQbCIJ5I
AxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAM2w
9ffoCUPRz6i8nAGVmHTBOSKa0O7LJ21/BT2u3e4O8pRnqwYrf8N6LKt5XhHim7PX
cX7ianG5rm/Z7mxSnNO35SNQYtWDWd6swkZxcl+8qwNIdp5ziaJ4q4jaTiGB3p9n
uH/TJlXYHZluMxtvRjTX4HGq16NjTk/8LzhOccSdlPrNIiBu9D+D0q04DXTWwloRs
pt5RBPkTKDM3Uavop0IUX+9vn23nz7X6+X9vmmibYKtnS2FEgcXFUWgcu0VqdGlp
f1CaMGA6i8nAGVmHTBOSKa0O7LJ21/BT2u3e4O8pRnqwYrf8N6LKt5XhHi/9OZHf
A++U3r0InmWMTOo9xtcCAwEAAaNZMFcwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB
/wQFMAMBAf8wHQYDVR0OBBYEFMJ07LD7D5E8Q/H1oiu+Sojqa2nTMBUGA1UdEQQO
MAyCCmt1YmVybmV0ZXMwDQYJKoZIhvcNAQELBQADggEBADt1P49fXp+ry1keIH1L
ZNdPLTYOBZj/JmFKKV3FQyb6H1i1vMoQ7D7fnZXw0wFQ5Mze9VWw3wuu/vmsUvBA
zIpwoe0NLaM9Jcv6V3tJPgiTCbM42JVD1pPCYwqxTP1F74JfdG2BWqKNCQbCIJ5I
o+r7SNLokTQB+Uc48/BDL6Hcn5wJCXbEC91OlBcqmMNCXa7ch+lQJxi6Wuy5cmLq
uH/TJlXYHZluMxtvRjTX4HGq16NjTk/8LzhOccSdlPrNIiBu9D+D0q04DXTWwloR
TXbBIRIPmellf56QA4IrvM+x9UtUO8lgM4XNN3jMIDdAXjnWqW0+P4vQs3mbo/Ri
6C0=
-----END CERTIFICATE-----
```

###### 2: SERVER. This is the endpoint to the Kubernetes API for our cluster. 

###### Get it's value by running:

```
kubectl config view | grep server 
```
```
https://192.168.56.10:6443
```

###### 3: USER_TOKEN. This is the token for the user that we'll use to connect to the Kubernetes cluster.


```
kubectl -n gitlab-runner create token default 
```

```
eyJhbGciOiJSUzI1NiIsImtpZCI6ImtJaXdFR2ppS3ItNkpIU1VtemVuXzY4ek8wR0U4MHV6ZDh5Unp4UnA3Rm8ifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjc0MTk4OTA4LCJpYXQiOjE2NzQxOTUzMDgsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJnaXRsYWItcnVubmVyIiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRlZmF1bHQiLCJ1aWQiOiI2NGE1Y2MyMS1iMThkLTRkNTQtYTI2Ny0yZTNiNWRjMDVjOGMifX0sIm5iZiI6MTY3NDE5NTMwOCwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmdpdGxhYi1ydW5uZXI6ZGVmYXVsdCJ9.p8RITKsGW7T12plK33p9Gn7OaxZ0ImBTxDs3yvA-m7UCfRdbNKC27u29XL8oTr7XfMVi21NigpAob_xtnsvqpw_H-bjxjqxsy_uV0TZh0bwEBHC3gFJHQMYcJL1GtrT-C4LhdHULaQ3HlVtcZvxap4owjKRR4HR3EoiDGae36tNtQ1Sisz8CXrpuMUCfsM_X7PxL39FxTaGYdPCrvBwvGR4iKbCNnzHbA_Y6vAOEhInAPNOUPvXFLc0PGYr-hv4dEzu8giG_e7AnzLx4sE09esoWa_ca1zwbydWf0n-tMKlFRVirQm07-WSPc2W9ikoM26iyfYsJ36tqLKy578WixQ
```



```
kubectl -n gitlab-runner get pods
```
```
NAME                                  READY   STATUS    RESTARTS   AGE
gitlab-runner-744f8bb595-cqkpq        1/1     Running   0          155m
project-deployment-85db7fdfb5-555l9   1/1     Running   0          53s
project-deployment-85db7fdfb5-mplf9   1/1     Running   0          53s
```

```
kubectl -n gitlab-runner describe pod project-deployment-85db7fdfb5-mplf9 | grep Node:
```

```
Node:             node004.cluster.local/192.168.56.11
```

