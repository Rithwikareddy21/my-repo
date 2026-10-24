# DevOps Practical Exam — Command Cheat Sheet (Windows / PowerShell)

> Use this as a quick reference. Replace values in `<angle brackets>` with your actual names.
> Run commands one at a time. Do not type the angle brackets themselves.
> Keep Docker Desktop open. For Kubernetes with Minikube, make sure Docker Desktop is running too.

---

## 0. Check tools

```powershell
docker --version
docker ps
docker images
node --version
npm --version
kubectl version --client
minikube version
```

### Install/setup reminders
- Install Docker Desktop from the official Docker website; `docker --install` is not a valid install command.
- If WSL is missing, open **PowerShell as Administrator** and run `wsl --install`, then restart if requested.
- Install Node.js LTS for Node/Express tasks.
- Install Minikube and kubectl from their official instructions; ensure `kubectl.exe` is on PATH.
- Start Docker Desktop before `minikube start --driver=docker`.

---

# Git & GitHub — Q1 to Q6

> Run these commands in PowerShell or the VS Code terminal. Replace the repository URL with your own GitHub repository URL.

## Q1. Configure Git identity (once per computer)

```powershell
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global --list
```

## Q2. Create a local repository, add a file, commit, and push to GitHub

Create a new repository on GitHub first, then use its URL below:

```powershell
mkdir my-repo
cd my-repo
git init
echo "Hello Git" > local-file.txt
git add local-file.txt
git commit -m "Add local file"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
git push -u origin main
```

If the repository is already initialized or the `origin` remote already exists, don't repeat `git init` or `git remote add origin`. Check the remote with:
```powershell
git remote -v
```

## Q3. Pull changes from GitHub

If the GitHub repository has a file that is not in your local folder, run:

```powershell
git pull origin main
dir
```

To view a text file in PowerShell:
```powershell
type github-file.txt
```

## Q4. Check repository status and commit changes

```powershell
git status
git add .
git commit -m "Update files"
git push origin main
```

## Q5. Clone an existing GitHub repository

Use the repository URL provided in the exam:

```powershell
git clone https://github.com/YOUR-USERNAME/YOUR-REPO.git
cd YOUR-REPO
git status
```

## Q6. Common Git commands / branch workflow

```powershell
git status
git log --oneline
git branch
git switch -c feature-branch
git add .
git commit -m "Add changes"
git push -u origin feature-branch
git switch main
git pull origin main
```

If the exam specifically asks to merge the feature branch into main:
```powershell
git switch main
git merge feature-branch
git push origin main
```

### Git quick revision

- `git init` — initialize a local repository.
- `git clone URL` — download an existing repository.
- `git add file` — stage a file.
- `git commit -m "message"` — save a snapshot locally.
- `git push` — upload local commits to GitHub.
- `git pull` — download and integrate remote changes.
- `git status` — show changed/staged files.
- `git log --oneline` — show compact commit history.
- `git remote -v` — show connected remote repositories.

> Note: If the exam's exact Q1–Q6 prompts use different filenames or steps, follow the exam sheet's wording. Use `dir` and `type filename.txt` in Windows PowerShell rather than relying on Unix-only `ls`/`cat` behavior.

---

# A. Docker commands

## Pull an image and inspect images/containers

```powershell
docker pull nginx:latest
docker images
docker ps
docker ps -a
```

- `docker ps` shows running containers.
- `docker ps -a` shows all containers, including stopped ones.
- `docker images` lists downloaded/built images.

## Run, inspect, stop, start, remove a container

```powershell
docker run -d --name my-nginx -p 8080:80 nginx:latest
docker ps
docker logs my-nginx
curl.exe http://localhost:8080
docker stop my-nginx
docker start my-nginx
docker rm -f my-nginx
docker ps -a
```

If the container name is already in use, inspect with `docker ps -a` and remove the old one before re-running:
```powershell
docker rm -f my-nginx
```

## Run a container with an environment variable

Example: PostgreSQL

```powershell
docker pull postgres:latest
docker run -d --name MyPostgresContainer -e POSTGRES_PASSWORD=abcd1234 -p 8844:5432 postgres:latest
docker ps
docker logs MyPostgresContainer
docker exec -it MyPostgresContainer psql -U postgres
```

Inside PostgreSQL, exit with:
```sql
\q
```

Remove it when finished:
```powershell
docker rm -f MyPostgresContainer
```

## Whoami containers / port mapping

```powershell
docker pull traefik/whoami:latest
docker run -d --name whoami1 -p 8686:80 traefik/whoami:latest
docker run -d --name whoami2 -p 8787:80 traefik/whoami:latest
docker ps
curl.exe http://localhost:8686
curl.exe http://localhost:8787
```

Optional third container to test port 8687:
```powershell
docker run -d --name whoami3 -p 8687:80 traefik/whoami:latest
curl.exe http://localhost:8687
```

## Docker network

```powershell
docker network create my-network
docker network ls
docker network inspect my-network
docker network rm my-network
```

## Build an image from a simple Nginx HTML page

Create `index.html` in a folder, then create a file named exactly `Dockerfile` in the same folder:

```dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Build and run:
```powershell
docker build -t my-web:1.0 .
docker run -d --name my-web-container -p 8080:80 my-web:1.0
curl.exe http://localhost:8080
docker logs my-web-container
```

Cleanup:
```powershell
docker rm -f my-web-container
```

## Docker Compose

Run these from the folder containing the Compose YAML file. Replace the filename with the exact file provided in the exam.

```powershell
docker compose -f .\docker-compose.yml up -d
docker compose -f .\docker-compose.yml ps
docker compose -f .\docker-compose.yml logs
docker compose -f .\docker-compose.yml down
```

Example for the provided Simple API file:
```powershell
docker pull jkaninda/simple-api:latest
docker compose -f .\jkaninda-simple-api-docker-compose.yml up -d
docker compose -f .\jkaninda-simple-api-docker-compose.yml ps
docker compose -f .\jkaninda-simple-api-docker-compose.yml logs
docker compose -f .\jkaninda-simple-api-docker-compose.yml down
```

## Pull the four images from the practice sheet

```powershell
docker pull postgres:latest
docker pull traefik/whoami:latest
docker pull jkaninda/simple-api:latest
docker pull ssatyab/studentmarkservice:latest
docker images
```

---

# B. Node.js Student CRUD API (Q11)

## 1. Create the project

```powershell
mkdir student
cd student
npm init -y
npm install express
```

Create `server.js` with the code below.

## 2. `server.js`

```javascript
const express = require('express');
const app = express();
app.use(express.json());

let students = [];

app.get('/students', (req, res) => {
  res.json(students);
});

app.get('/students/:id', (req, res) => {
  const student = students.find(s => s.id == req.params.id);
  if (student) return res.json(student);
  res.status(404).send('Not found');
});

app.post('/students', (req, res) => {
  students.push(req.body);
  res.status(201).json(req.body);
});

app.put('/students/:id', (req, res) => {
  const index = students.findIndex(s => s.id == req.params.id);
  if (index === -1) return res.status(404).send('Not found');
  students[index] = req.body;
  res.json(students[index]);
});

app.delete('/students/:id', (req, res) => {
  const before = students.length;
  students = students.filter(s => s.id != req.params.id);
  if (students.length === before) return res.status(404).send('Not found');
  res.sendStatus(204);
});

app.listen(3000, () => console.log('Student service running on port 3000'));
```

Run and test locally:
```powershell
node server.js
```

Keep that terminal open. In another terminal:
```powershell
curl.exe http://localhost:3000/students
```

To stop the local server, press `Ctrl+C`.

## 3. Create `Dockerfile` in the same `student` folder

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --omit=dev
COPY server.js .
EXPOSE 3000
CMD ["node", "server.js"]
```

Optional `.dockerignore` file:
```text
node_modules
```

Build the image:
```powershell
docker build -t student:1.0 .
docker images
```

Test the container (make sure the local `node server.js` process is stopped first):
```powershell
docker run -d --name student-api-container -p 3000:3000 student:1.0
docker ps
curl.exe http://localhost:3000/students
docker logs student-api-container
```

Cleanup if needed:
```powershell
docker rm -f student-api-container
```

## 4. Kubernetes YAML files for Q11

Create `deployment.yml` in the project folder:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: student
spec:
  replicas: 1
  selector:
    matchLabels:
      app: student
  template:
    metadata:
      labels:
        app: student
    spec:
      containers:
        - name: student
          image: student:1.0
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
```

Create `service.yml` in the same folder:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: student
spec:
  selector:
    app: student
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

## 5. Start Kubernetes / deploy

For Minikube using Docker driver:
```powershell
minikube start --driver=docker
minikube image load student:1.0
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl get deployments
kubectl get pods
kubectl get services
kubectl get all
kubectl logs deploy/student
```

Forward the service to localhost; keep this terminal open:
```powershell
kubectl port-forward svc/student 8080:80
```

Now test at `http://localhost:8080` (use `/students` for the student API). To stop port-forwarding, press `Ctrl+C`.

Delete the Kubernetes objects:
```powershell
kubectl delete -f service.yml
kubectl delete -f deployment.yml
minikube stop
```

> If you use a different Kubernetes cluster, it may not have your local image. You may need to push the image to a registry and update the `image:` value.

---

# C. Minikube / kubectl quick commands

```powershell
minikube version
kubectl version --client
minikube start --driver=docker
kubectl get nodes
minikube status
```

## Create a deployment with 3 replicas

```powershell
kubectl create deployment my-deploy --image=nginx --replicas=3
kubectl get deployments
kubectl get pods
```

Expose it as a ClusterIP service:
```powershell
kubectl expose deployment my-deploy --name=my-svc --port=80 --target-port=80 --type=ClusterIP
kubectl get services
```

Delete:
```powershell
kubectl delete service my-svc
kubectl delete deployment my-deploy
```

## Create a pod

```powershell
kubectl run my-pod --image=nginx
kubectl get pods
kubectl delete pod my-pod
```

## Create a namespace

```powershell
kubectl create namespace studentmarkservice-ns
kubectl get namespaces
```

Delete the namespace when no longer needed:
```powershell
kubectl delete namespace studentmarkservice-ns
```

## Generate YAML from commands

```powershell
kubectl run my-pod --image=nginx --dry-run=client -o yaml > pod.yml
kubectl create deployment my-deploy --image=nginx --dry-run=client -o yaml > deployment.yml
kubectl create service clusterip my-svc --tcp=80:80 --dry-run=client -o yaml > service.yml
```

---

# D. Q9B — Student Mark Service Kubernetes files

Use the exact `namespace.yml`, `deployment.yml`, and `service.yml` supplied by your exam. Run them in this order from the folder containing those files:

```powershell
docker pull ssatyab/studentmarkservice:latest
kubectl apply -f namespace.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl get namespaces
kubectl get pods -A
kubectl get services -A
```

The namespace must be created before applying the deployment and service. Read the namespace and names from the provided YAML files.

If the Service is `NodePort`, inspect its port:
```powershell
kubectl get services -A
```

If it is not exposed externally, port-forward it using the actual service name, namespace, and service port from `service.yml`:
```powershell
kubectl port-forward svc/<service-name> 8080:<service-port> -n <namespace-name>
```

Test using the documented endpoint from the exam/API instructions. Do not guess API paths if they are not specified.

---

# E. HTTP methods — quick memory aid

- `GET` = read
- `POST` = create
- `PUT` = replace/update
- `DELETE` = delete

For the Student API:
- `GET http://localhost:3000/students`
- `POST http://localhost:3000/students`
- `PUT http://localhost:3000/students/1`
- `DELETE http://localhost:3000/students/1`

For POST and PUT, send JSON with header `Content-Type: application/json`, for example:
```json
{
  "id": 1,
  "studentName": "Asha",
  "totalMarks": 450
}
```

---

## Common troubleshooting

- **`docker` not recognized / cannot connect:** Start Docker Desktop; check `docker --version` and `docker ps`.
- **`kubectl` not recognized:** Install kubectl and add its folder to Windows PATH; reopen VS Code after changing PATH.
- **Minikube fails to start:** Start Docker Desktop first, then retry `minikube start --driver=docker`.
- **Port already in use:** Stop the other server/container or choose another host port, e.g. `-p 8081:3000`.
- **Container name already in use:** Check `docker ps -a`; remove the old container with `docker rm -f <container-name>`.
- **PowerShell curl:** Use `curl.exe`, not just `curl`, to ensure the real curl executable runs.
- **YAML apply errors:** Run commands from the directory containing the YAML files and check file names/indentation.
