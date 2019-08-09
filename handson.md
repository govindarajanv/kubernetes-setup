##### create the namespace
kubectl create namespace dev
kubectl create namespace test

##### Create the httpd deployment in the dev namespace:
kubectl create deployment httpd-dev --image=httpd -n dev
##### Create the tomcat deployment in the dev namespace:
kubectl create deployment tomcat-dev --image=tomcat -n dev
##### Create the mongo deployment in the dev namespace:
kubectl create deployment mongo-dev --image=mongo -n dev
##### sock shop
kubectl create namespace sock-shop
kubectl apply -f https://raw.githubusercontent.com/microservices-demo/microservices-demo/master/deploy/kubernetes/complete-demo.yaml
kubectl get deployments -n sock-shop
kubectl get pods -n sock-shop
kubectl scale deployments carts catalogue --replicas=3 -n sock-shop
kubectl get deployments -n sock-shop
kubectl get hpa -n sock-shop
