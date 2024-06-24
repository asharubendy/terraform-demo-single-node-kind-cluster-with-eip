## Hello apprentices

Your job for today is to:

Ensure ASAP that you have a:
a cloud based mongodb instance

https://www.mongodb.com/docs/atlas/tutorial/deploy-free-tier-cluster/
move your data into it (mongodb is a particular pain to set up on a kind cluster)

AND:

Set of docker containers and test they run fine together

They must: 
Be able to change url by using a envronment variable. 
I.e if its local, it should be able to take localhost, if its hosted, it should be able to take a url. 

the frontend / backend should have customisable endpoint urls via env variables too

https://www.thisdot.co/blog/typescript-integration-with-env-variables - for ts 

https://www.cherryservers.com/blog/set-docker-environment-variables

this means we can override them when we put them into kubernetes and use the built in dns services. 

heres a sample Dockerfile for front end: 
```
FROM node:22

WORKDIR /app

COPY package.json ./package.json

RUN npm install

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

and api example: (thanks again nial im glad i dont have to type these out <3)
```
FROM node:22

# Set the working directory in the container
WORKDIR /app
# Copy the package.json and package-lock.json (if available)
COPY package*.json ./

# Install the dependencies
RUN npm install

# Copy the rest of the application files
COPY . .

# Expose the port the app runs on
EXPOSE 3101

# Command to run the application
CMD ["npm", "run", "dev"]
```

Obviously you will have to change these when creating yours, take note of the ports and stuff and change them accordingly. 

---

## Then its time to build your docker image:

Heres nials which is a great example of building a container using gh actions:

General steps are 
- checkout your code
- set up docker buildx
- build and push docker image to ghcr

```
  build-mui:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./social-api-frontend/.
          platforms: linux/amd64,linux/arm64
          file: social-api-frontend/Dockerfile
          push: true
          tags: ghcr.io/nmcc1212/mui-frontend:latest
          cache-from: type=gha
          cache-to: type=gha
```

https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions


https://dev.to/willvelida/pushing-container-images-to-github-container-registry-with-github-actions-1m6b

from there i will set you up a repo and a jenkins project, which should be basically the same across all of you:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-typescript-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-typescript-app
  template:
    metadata:
      labels:
        app: my-typescript-app
    spec:
      containers:
      - name: my-typescript-app
        image: my-typescript-app:latest
        ports:
        - containerPort: 8080
```
and the portforwarding / svc 
```
apiVersion: v1
kind: Service
metadata:
  name: my-typescript-app-service
spec:
  selector:
    app: my-typescript-app
  ports:
    - protocol: TCP
      port: your app port
      targetPort: 80
  type: ClusterIP
  ````

