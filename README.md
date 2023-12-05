# ntr-k8s
##  Project directory tree
```
├── ntr-backend
│   ├── src
│   ├── Dockerfile
│   └── requirements.txt
├── ntr-frontend
│   ├── Dockerfile
│   └── package.json
├── ntr-order-handler
│   ├── src
│   ├── Dockerfile
│   └── requirements.txt
├── ntr-payment-handler
│   ├── src
│   ├── Dockerfile
│   └── requirements.txt
├── ntr-inventory-handler
│   ├── src
│   ├── Dockerfile
│   └── requirements.txt
├── ntr-delivery-handler
│   ├── src
│   ├── Dockerfile
│   └── requirements.txt
├── ntr-tests
│   └── main.py
├── ntr-k8s
│   └── *.yaml
```
The above is the project directory tree if you clone all of our repositories. Fortunately, you don't need to. Because all images were pushed to `ghcr.io`

The only repo you need to clone is `ntr-k8s` which contains all necessary deployment files and run the code below.

## Kubernetes Setup
Before running the following code make sure to install `helm`, `Signoz`,  and `kubectl` 

To port forwarding Signoz in Kubernetes after installing it please run:
```bash
export POD_NAME=$(kubectl get pods --namespace platform -l "app.kubernetes.io/name=signoz,app.kubernetes.io/instance=my-release,app.kubernetes.io/component=frontend" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:3301 to use your application"
  kubectl --namespace platform port-forward $POD_NAME 3301:3301
```

To set up kubernetes, run (in toktik-k8s):
```bash
helm upgrade --install traefik traefik/traefik
kubectl create secret docker-registry ghcr-login-secret \
 --docker-server=https://ghcr.io \
 --docker-username={username} \ 
 --docker-password={personal_access_token}
kubectl apply -f .
```
The code above install trafik proxy using helm then create a k8s secret for pulling image from ghcr.io then finally apply all the services and deployment yaml files

wait until all pods are up and running, then you can access our toktik at http://localhost:80/
## Backend
Our backend (`ntr-backend` was implemented using FastAPI.) 
- For endpoints/functionality, in `routers/`, there are the following files
	- `users.py` -- user creation, management, and authentication stuff
	- `orders.py` -- order creation and update
- For database stuff
	- `models.py` contain the models for each table in our database
	- `db_services.py` contains all methods which involve interaction with our database.

## Order Handler
This handler is special since it acts as the execute controller in our SAGA design
It is another FastAPI app. The other handlers talk to it when the order status needs to be updated

## Payment, Inventory, Delivery Handler
Act as services pipeline to service the order from user. All handlers are connected using Redis