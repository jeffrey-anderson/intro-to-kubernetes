# Deploy a complex app:

Deploy the API and front end website from [Spring Web Essentials](https://leanpub.com/springwebessentials).

[![Cover of Spring Web Essentials](https://d2sofvawe08yqg.cloudfront.net/springwebessentials/s_hero?1620626184)](https://leanpub.com/springwebessentials)

## Give it a unique namespace:

Apply [swe-ns.yaml](./swe-ns.yaml): 
```
kubectl apply -f swe-ns.yaml
```

## Create a secret for the database and API to use:

Apply [postgres-db-password-secret.yaml](postgres-db-password-secret.yaml): 
```
kubectl apply -f postgres-db-password-secret.yaml
```

**NOTE:** While the password may look encrypted, it is only encoded and easily be decoded. DO NOT check passwords and other secrets into version control. 

```
echo VXNlLWEtQmV0dGVyLVBhc3N3MHJk | base64 --decode
```

## Deploy a PostgreSQL StatefulSet for the API to use:

Apply [postgres-sts.yaml](postgres-sts.yaml): 
```
kubectl apply -f postgres-sts.yaml
minikube service list
```


## Deploy an API that uses the database:

Apply [swe-app-api-deploy.yaml](swe-app-api-deploy.yaml): 
```
kubectl apply -f swe-app-api-deploy.yaml
```

## Create a service so clients can connect:

Apply [swe-app-api-svc.yaml](./swe-app-api-svc.yaml): 
```
kubectl apply -f swe-app-api-svc.yaml
```

**NOTE:** Run `minikube tunnel` in a second terminal window and provide your password when prompted.

Finding the service IP address:

```
minikube service list
curl 192.168.49.2:30680/api/authors
kubectl get services -n swe
curl 10.108.120.195:3000/api/authors
```

## Deploy the front end single page application:

Apply [swe-vue-app-deploy.yaml](./swe-vue-app-deploy.yaml):
```
kubectl apply -f swe-vue-app-deploy.yaml
kubectl get all -n swe
```

## Expose it via a service

Apply [swe-vue-app-svc.yaml](swe-vue-app-svc.yaml)

```
kubectl apply -f swe-vue-app-svc.yaml
kubectl get all -n swe
```

Explore:
* Open the browser and point at the external IP address for the `swe-vue-app-svc` service
* Use the dev tools and inspect the network activity
* Create a DNS entry in `/etc/hosts` with external IP address for `swe-api-service` and a hostname of `swe-api-service`
* Refresh the page and notice the change in the network stats
* Check the console for errors
* Create a DNS entry in `/etc/hosts` with external IP address for `swe-vue-app-svc` and a hostname of `swe-blog`
* Point your browser at `http://swe-blog/` and check the network and console
* Use DBeaver to connect to the database and populate some sample data:
  ```
  INSERT INTO AUTHOR (first_name, last_name, EMAIL_ADDRESS) VALUES
    ('Ned', 'Flanders', 'ned.flanders@springwebessentials.com'),
    ('Homer', 'Simpson', 'homer.simpson@springwebessentials.com');

  INSERT INTO BLOG_POST (category, content, date_posted, title, author_id) VALUES
    ('Healthy and Delicious', 'Bacon ipsum dolor amet cow turducken ball tip fatback filet mignon. T-bone bresaola capicola andouille beef ribs. Hambur
  ger doner meatball spare ribs tail picanha. Meatloaf chicken ribeye sausage short ribs bacon tail. Porchetta fatback pork belly corned beef meatloaf.
  Pig boudin frankfurter strip steak turkey biltong drumstick. Tongue hamburger kielbasa, venison frankfurter short loin meatball ribeye tri-tip ham j
  owl jerky.', now(), 'Chicken Gyro with Beer Cheese Soup', 1 ),
  ('Delicious Desserts', 'Bacon ipsum dolor amet chislic tenderloin ground round, meatball ham hock fatback cupim beef ribs kevin pig ball tip filet
  mignon leberkas picanha pork chop. Alcatra swine short ribs, burgdoggen capicola prosciutto tenderloin brisket porchetta kielbasa cow spare ribs cupi
  m. Rump leberkas ground round tongue short loin ham hock venison shoulder pig meatball chuck pork loin picanha doner. Flank tongue shank, strip steak
  ribeye pork cow bacon tail. Frankfurter hamburger bresaola andouille t-bone buffalo pancetta cow chuck pork pastrami tail prosciutto. Filet mignon l
  andjaeger flank frankfurter bacon.', now(), 'Gooey Chocolate Crumble with Espresso', 2 );
  ```
* Refresh the page

## Clean up:

```
kubectl get all,pv,pvc,secrets -n swe

kubectl delete service/swe-vue-app-svc service/swe-api-service service/postgres -n swe
kubectl delete deployment.apps/swe-vue-app deployment.apps/swe-app-api -n swe
kubectl delete statefulset.apps/postgres -n swe
kubectl delete persistentvolumeclaim/postgres-pv-claim secret/postgres-db-password -n swe

kubectl get all,pv,pvc,secrets -n swe
```