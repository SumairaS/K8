# cat index.html
bash 
```
<!DOCTYPE html>
<html>
<head>
    <title>My Nginx Page</title>
</head>
<body>
    <h1>Hello from Nginx!</h1>
</body>
</html>
```
# cat deployment.yml
bash ```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80  # Add this line to expose port 80
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html-volume
        configMap:
          name: nginx-html-config 
          ```

# cat service.yml
bash ```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30000  # Optional, specify a port between 30000-32767
  selector:
    app: nginx
       ```
    
# cat configmap.yaml

Bash ```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-html-config
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>My Nginx Page</title>
    </head>
    <body>
        <h1>Hello from Nginx!</h1>
    </body>
    </html>
    ```

*********************************************************************************************************************************************************8
**# configmap.yaml  deployment.yml  index.html  service.yml
# so now we have Kubernetes manifests for a deployment setup in the nginx-html directory. If you want to apply these manifests to set up an NGINX deployment, follow these # # steps:**
1. Apply the Manifests
kubectl apply -f configmap.yaml
2. Apply the Deployment:
kubectl apply -f deployment.yml
Apply the Service:
kubectl apply -f service.yml
3. Verify the Deployment
kubectl get pods
4. Check Services:
kubectl get services
5. Check ConfigMaps:
kubectl get configmaps
6. Inspect the Logs (if necessary):
kubectl logs <pod-name>
This will deploy your NGINX application according to the configurations in your manifests. If you encounter any issues or need further customization, let me know!
