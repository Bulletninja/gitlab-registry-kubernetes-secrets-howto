# Description
Copy-Paste Ready™ how-to for using private [Gitlab Container Registry](https://docs.gitlab.com/ce/user/project/container_registry.html) with [Kubernetes](https://kubernetes.io).

#### Create env file with credentials

```bash
cat <<EOF >dockercfg.env
export DOCKER_REGISTRY_SERVER=<registry_url>
export DOCKER_USER=<username>
export DOCKER_PASSWORD=<password>
export DOCKER_EMAIL=<email>
export DOCKER_IMAGE_PATH=<image_path_with_tag>
EOF
```

#### Edit accordingly
See [dockercfg.env.example](https://github.com/jugatsu/gitlab-registry-kubernetes-secrets-howto/blob/master/dockercfg.env.example)

#### Export variables from dockercfg.env
```bash
eval $(cat dockercfg.env)
```

#### Export additional variables
```bash
export REGISTRY_NAME=`echo $DOCKER_REGISTRY_SERVER | sed -e 's/^http:\/\///g' -e 's/^https:\/\///g'`
export DOCKER_IMAGE_FULL_PATH=$REGISTRY_NAME/$DOCKER_IMAGE_PATH
```

#### Create secret in the Kubernetes Cluster
```bash
kubectl create secret docker-registry gitlab-registry \
  --docker-server=$DOCKER_REGISTRY_SERVER \
  --docker-username=$DOCKER_USER \
  --docker-password=$DOCKER_PASSWORD \
  --docker-email=$DOCKER_EMAIL
```

#### Test
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: foo
spec:
  containers:
    - name: foo
      image: $DOCKER_IMAGE_FULL_PATH
      imagePullPolicy: Always
  imagePullSecrets:
    - name: gitlab-registry
EOF
```

#### Check if the image was pulled successfully
```bash
kubectl describe po/foo | grep -i pull
```

### Cleanup
#### Remove pod
```bash
kubectl delete po/foo
```
#### Remove file with environment variables
```bash
rm dockercfg.env
```
#### Unset environment variables
```bash
unset DOCKER_REGISTRY_SERVER
unset DOCKER_USER
unset DOCKER_EMAIL
unset DOCKER_PASSWORD
unset DOCKER_IMAGE_PATH
unset REGISTRY_NAME
unset DOCKER_IMAGE_FULL_PATH
```
