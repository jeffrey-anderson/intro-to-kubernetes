# Pulling Private Images

References:

* [Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

Procedure:

* Make sure you are logged out:

  ```
  docker logout
  ```

* Create a new token at https://hub.docker.com/settings/security

* Using the the token displayed as the password, login to Docker:

  ```
  docker login -u datadaddy
  ```

* Create a Docker Config based secret

  ```
  kubectl create secret generic regcred --from-file=.dockerconfigjson=$HOME/.docker/config.json --type=kubernetes.io/dockerconfigjson

  kubectl get secret regcred --output=yaml
  ```

* Use the secret to create a pod based on a private image by applying [img-pull-secret-demo.yaml](img-pull-secret-demo.yaml):

  ```
  kubectl apply -f img-pull-secret-demo.yaml; watch kubectl get all,secrets

  minikube service list
  ```

* Clean up:

  ```
  kubectl delete secret/regcred service/img-pull-secret-demo-svc pod/img-pull-secret-demo
  ```
  * Delete your access token
  * Logout of Docker: `docker logout`