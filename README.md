# Coworking Space Service Extension
The Coworking Space Service is a set of APIs that enables users to request one-time tokens and administrators to authorize access to a coworking space. This service follows a microservice pattern and the APIs are split into distinct services that can be deployed and managed independently of one another.

For this project, you are a DevOps engineer who will be collaborating with a team that is building an API for business analysts. The API provides business analysts basic analytics data on user activity in the service. The application they provide you functions as expected locally and you are expected to help build a pipeline to deploy it in Kubernetes.

## Getting Started

### Dependencies

#### Local Environment
1. Python Environment - run Python 3.6+ applications and install Python dependencies via `pip`
2. Docker CLI - build and run Docker images locally
3. `kubectl` - run commands against a Kubernetes cluster

#### Remote Resources
1. AWS CodeBuild - build Docker images remotely
2. AWS ECR - host Docker images
3. Kubernetes Environment with AWS EKS - run applications in k8s
4. AWS CloudWatch - monitor activity and logs in EKS
5. GitHub - pull and clone code

### Setup
#### 1. Configure a Database
Set up a Postgres database through YAML configurations. The YAML files are `pvc.yaml`, `pv.yaml`, and `postgresql-deployment.yaml`. Apply each one of them subsequently.

##### Test Database Connection
The database is accessible within the cluster. This means that when you will have some issues connecting to it via your local environment. You can either connect to a pod that has access to the cluster _or_ connect remotely via [`Port Forwarding`](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

* Connecting Via Port Forwarding
```bash
kubectl port-forward --namespace default svc/<SERVICE_NAME>-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432
```

* Connecting Via pods
```bash
kubectl exec -it <POD_NAME> bash
PGPASSWORD="<PASSWORD HERE>" psql postgres://postgres@<SERVICE_NAME>:5432/postgres -c <COMMAND_HERE>
```

##### Run Seed Files
Run the seed files in `db/` in order to create the tables and populate them with data.

```bash
kubectl port-forward --namespace default svc/<SERVICE_NAME>-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < <FILE_NAME.sql>
```

Or, you can colletively put those commands in a script like below:

```bash
# After preparing the environment variables

# Initialise the db
cd db
chmod +x db_initialisation.sh
./db_initialisation.sh
```

### Run the Analytics Application Locally
In the `analytics/` directory:

1. Install dependencies
```bash
pip install -r requirements.txt
```
2. Run the application
```bash
# Prepare the environment variables
source ./export_vars.sh
python app.py
```

3. Verify The Application

* Generate report for check-ins grouped by dates
`curl <BASE_URL>/api/reports/daily_usage`

* Generate report for check-ins grouped by users
`curl <BASE_URL>/api/reports/user_visits`

### Deployment yaml files
The yaml files are available in the `./deployment` directory.

## Project Instructions
1. Set up a Postgres database
2. Create a `Dockerfile` for the Python application. I use `python:3.8-slim-bookworm` as the base image, following the workspace's Python version.
3. Write a simple build pipeline with AWS CodeBuild to build and push a Docker image into AWS ECR
4. Create a service and deployment using Kubernetes configuration files to deploy the application
5. Check AWS CloudWatch for application logs

### Deliverables
1. Deployment file - in `./deployment` directory
2. `Dockerfile` - in `./analytics` directory
3. Screenshots - in `./screenshots` directory:
    1. Screenshot of AWS CodeBuild pipeline
    2. Screenshot of AWS ECR repository for the application's repository
    3. Screenshot of `kubectl get svc`
    4. Screenshot of `kubectl get pods`
    5. Screenshot of `kubectl describe svc <DATABASE_SERVICE_NAME>`
    6. Screenshot of `kubectl describe deployment <SERVICE_NAME>`
    7. All Kubernetes config files used for deployment (ie YAML files)
    8. Screenshot of AWS CloudWatch logs for the application
4. `README.md` file,
