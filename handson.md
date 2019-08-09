# create the namespace
kubectl create namespace dev
kubectl create namespace test

# Create the httpd deployment in the dev namespace:
kubectl create deployment httpd-dev --image=httpd -n dev
# Create the tomcat deployment in the dev namespace:
kubectl create deployment tomcat-dev --image=tomcat -n dev
# Create the mongo deployment in the dev namespace:
kubectl create deployment mongo-dev --image=mongo -n dev
