# DevPlanet
Github Repository for the Dev Planet Publication.


kubectl apply -f postgres-deployment.yaml
kubectl apply -f backend-deployment.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f adminer-deployment.yaml
kubectl apply -f postgres-pvc.yaml  # If using PVC

-------------------------

sudo mkdir -p /mnt/data/postgres-all nodes create it 
sudo chown -R 1000:1000 /mnt/data/postgres


-----------------------------------------------
# deploy.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/postgres  # Path on the host node

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres  # Replace with your custom image if necessary
          env:
            - name: POSTGRES_USER
              value: "john"
            - name: POSTGRES_PASSWORD
              value: "example"
            - name: POSTGRES_DB
              value: "endgame"
          ports:
            - containerPort: 5432
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgres-storage
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  type: ClusterIP
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    app: postgres

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: roshandocker11/endgamebackend:v1  # Replace with your Docker Hub image
          env:
            - name: DB_USER
              value: "john"
            - name: DB_PASS
              value: "example"
            - name: DB
              value: "endgame"
            - name: DB_HOST
              value: "postgres-service"
          ports:
            - containerPort: 5000

---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  ports:
    - port: 5000
      targetPort: 5000
  selector:
    app: backend

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: roshandocker11/endgamefrontend:v1  # Replace with your Docker Hub image
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: frontend




