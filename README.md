# Docker Deployment of full-stack App on both local and AWS ECS

This README provides instructions on setting up and running a Dockerized multi-container application. The application consists of a backend, frontend, and uses MongoDB Atlas as the database. The deployment instructions cover production on both Local and AWS ECS. Please note that the code for the app, including the JavaScript, JSON, HTML and CSS is not authored by the repository owner.

## Production Deployment - Local System

### Prerequisites

- Docker installed on your local machine.
- MongoDB Atlas account for the database (free tier is applicable).

### Getting Started

1. Clone the repository:

    ```bash
    git clone https://github.com/your-username/your-repo.git
    
    cd your-repo
     ```

2. Set a value on the .env file for the MONGODB_NAME variable so that a database is used for this development phase:

```
MONGODB_USERNAME='<your-username>'
MONGODB_PASSWORD='<your-password>'
MONGODB_URL='<URL>'
MONGODB_NAME='<goals-$dev>'
```

3. Navigate in your MongoDB Atlas account and in your cluster do the following:

- Click on connect
- Click on Drivers
- In the number 3 copy the url(string between @'<copy-this-string>'/)

4. Copy your MongoDB Atlas URL on the variable MONGODB_URL in the .env file:

```
MONGODB_USERNAME='<name>'
MONGODB_PASSWORD='<password>'
MONGODB_URL='<MongDB-ATLAS-URL>'
MONGODB_NAME='<goals-$dev>'
```

5. Navigate in your MongoDB Atlas account and allow access to your db:

- Click on Network access
- ADD IP ADDRESS
- You might as well select “allow from anywhere” if you are trying to send this app to production following the Deployment process on this project.

6. Navigate in your MongoDB Atlas account and create a new user to have read+write access to this db:

- Database access
- username:'<name>'
- password:'<password>'
- Give the user a role for read and write permission to any database

7. Copy your the credentials of the MongoDB ATLAS user that has read+write access to the database in the .env file for the variables 

    MONGODB_USERNAME and MONGODB_PASSWORD:
    MONGODB_USERNAME='<name>'
    MONGODB_PASSWORD='<password>'
    MONGODB_URL='<MongDB-ATLAS-URL>'
    MONGODB_NAME='<goals-$dev>'

8. Build and run the Docker containers:

    ```docker
    docker-compose up
    ```

9. Open your browser and visit http://localhost:3000 to access the application.

10. Delete the containers:

    ```docker
    docker-compose down
    ```

## Production Deployment - AWS ECS

### Prerequisites

- Docker installed locally.
- Dockerhub account.
- MongoDB Atlas account for a managed database (free tier is applicable).
- Database configured in MongoDB Atlas with a Networks access and a user with read and write access to the db.
- An AWS account.

### Getting Started

#### Configure the NodeJS backend container on AWS ECS using the mgmt console

1. Create Cluster

- Add a Task for the Backend Container
- Launch a Service for the Backend Container
- Cluster name: goals-app
- Check create VPC
- Keep the default networking configuration
- Click on Create Cluster

2. Add a Task for the Backend Container

- Under Tasks Definitions click Create new Task
- Choose Fargate
- Task definition name: goals
- Task role: choose the the one available (you should have one available. If you don’t have one delete the entire cluster again and create a new cluster and lunch the first wizard again)
- Task memory: choose the smallest one
- Task CPU: choose the smallest one
- Click Add Container
- Container name: goals-backend
- Image: xcocox/goals-node
- Port mappings: 80
- Ignore Healthcheck
- Under Environment > Command: node,app.js (this command will execute the app.js inside the container with the node runtime)
- Under Environment > environment variables: MONGODB_USERNAME=<name>; MONGODB_PASSWORD=<password>; MONGODB_URL=<MongDB-ATLAS-URL>; MONGODB_NAME=<goals-$prod> (these are the values set on your MongoDB Atlas db)
- Leave rest as default
- Click Add

3. Launch a Service for the Backend Container

- Under Cluster > Services > Click on Create
- Launch Type: FARGATE
- Task Definition: goals
- Revision: latest
- Service name: goals-service
- Number of tasks: 1
- Deployment type leave as default
- Click Next Step

4. Configure Network
	
- Cluster VPC: Select the default VPC created (a VPC must had been created when creating the cluster)
- Subnets: add both subnets available
- Auto-assign public ip must be enabled
- Load Balancer type: choose Application Load Balancer
- Click on EC2 prompt to create the load balancer

5. Load Balancer (the creation of this Load Balance will incur costs)
	
- Click on Create Application Load Balancer
- Name: ecs-lb
- Choose internet-facing
- Load balancer port: 80
- VPC: make sure to connect to the same VPC as the cluster
- Availability zones: check all boxes
- Leave rest as default
- Click Next
- Click on Next: Configure security group
- Select: Existing security group
- Make sure to select de default security group and the default one
- Click Next
- Under target group > Name: tg
- Under target group > Target type: choose IP
- Under Healthcheck > Health check path: /goals
- Click Register Target group
- Click Next Review
- Click Create

6. Configure Network
	
- Load balancer name: ecs-lb
- Container name : port: goals-backend:80:80 - Click Add to load balancer
- Target group name: tg
- Click next step
- Click next step
- Create Service

7. Locate the load balancer url
	
- Under EC2 dashboard > click Load Balancers
- DNS name: copy this url and store it

#### Configure the React frontend container on AWS ECS using the mgmt console

8.  Modify the App.js file for the connection and communication with the backend container
	
- Build the Frontend Docker image and push it to your Dockerhub repository
- Add a Task for the Frontend Container
- Create a Load Balancer for the Frontend Container
- Launch a Service for the Frontend Container
- Access the application

9. Modify the App.js file for the connection and communication with the backend container

- Locate the copied url of the load balancer for the backend container
- Navigate to the documentation of this repository.
- Locate the frontend documentation or directory.
- Open the App.js file within the frontend directory.
- In the App.js file, find line 7, where the backend URL is specified.
- Copy the backend URL and replace '<copy-here-the-url>' on line 10 in App.js with the actual URL.

Example:
// Replace '<copy-here-the-url>' with the actual backend URL
const backendUrl = 'http://your-backend-url';

- Save the changes to App.js.

10. Build the Frontend Docker image and push it to your Dockerhub repository
	
- Create a new repository on your dockerhub profile > name: <your-dockerhub-repo-name>
- Open a terminal
- Navigate to the root directory of the project
- Use the following command to build the frontend Docker image

    ```docker    
    docker build -f frontend/Dockerfile.prod -t <your-dockerhub-repo-name>/<your-react-frontend-image-name> ./frontend
	```

- Push the image to your dockerhub repo

    ```docker
    docker push <your-dockerhub-repo-name>/<your-react-frontend-image-name>
    ```
Replace '<your-dockerhub-repo-name>' with your Docker Hub repository name.

11. Add a Task for the Frontend Container
	
- Under Tasks Definitions click Create new Task
- Choose Fargate
- Task definition name: goals-react
- Task role: choose the the one available (use the same task role as for the backend)
- Task memory: choose the smallest one
- Task CPU: choose the smallest one
- Click Add Container
- Container name: goals-backend
- Image: <your-dockerhub-repo-name>/<your-react-frontend-image-name>
- Port mappings: 80
- Leave rest as default
- Click Add
- Click Create

12.  Create a Load Balancer for the Frontend Container
	
- Under EC2 dashboard > click Load Balancers
- Click Create Load Balancer
- Name: goals-react-lb
- Choose internet-facing
- Load balancer port: 80
- VPC: make sure to connect to the same VPC as the cluster
- Availability zones: check all boxes
- Leave rest as default
- Click Next
- Click on Next: Configure security group
- Select: both existing security groups
- Make sure to select de default security group and the default one
- Click Next
- Under target group > Name: react-tg
- Under target group > Target type: choose IP
- Under Healthcheck > Health check path: /
- Click Register Target group
- Click Next Review
- Click Create

**The url of this Load Balancer is the url that will be used to reach this React app.**

13. Launch a Service for the Frontend Container
	
- Under Cluster > Services > Click on Create
- Launch Type: FARGATE
- Task Definition: goals-react
- Revision: latest
- Cluster: goals-app
- Service name: goals-react
- Number of tasks: 1
- Use rolling update
- Click Next Step

#### Configure Network

14. Configure the Network

- Cluster VPC: Select the default VPC created (a VPC must had been created when creating the cluster)
- Subnets: add both subnets available
- Security groups>click on edit>select an existing security group>select goals--856>click Save
- Auto-assign public ip must be enabled
- Load Balancer type: choose goals-react-lb
- Click Add to load balancer
- target group name: react-tg
- Leave rest as default
- Click Next step

15. Configure Network Service

- Leave as default
- Click next step
- Click next step
- Create Service

### Access the application

Access de deployed app following these steps:

1. Under EC2 dashboard > click Load Balancers
2. Select goals-react-lb
3. Copy the DNS name
4. Copy the DNS name on your browser

### Deleting AWS ECS Cluster - Manual Steps

1. Delete Load Balancers:
		
- Open the AWS Management Console.
- Navigate to the Load Balancers page:
- Go to the Load Balancers Dashboard.
- Find and select the backend load balancer:
- Click on the backend load balancer name.
- In the load balancer details, navigate to the "Listeners" tab.
- Delete the listener associated with the backend service.
- Click the Actions dropdown and select Delete Load Balancer.
- Confirm the deletion.
- Repeat the above steps for the frontend load balancer.
- Repeat the process for the frontend load balancer.

2. Delete ECS Cluster:

- Open the AWS Management Console.
- Navigate to the ECS Cluster page:
- Go to the ECS Dashboard.
- Select the cluster you want to delete.
- Click the Delete Cluster button.
- Confirm the deletion.


#### Disclaimer Regarding AWS ECS (Fargate) Costs

Running services on AWS ECS (Fargate) may incur charges based on your usage, AWS pricing, and resource configurations.

- Cost Considerations:
  * Users deploying services on AWS ECS (Fargate) are responsible for any costs incurred, including but not limited to container runtime network usage, load balancers, and associated AWS services.

- Cost Estimation:
  * It is recommended to estimate potential costs by considering factors such as the number of tasks, CPU and memory allocations, load balancer usage, data transfer, and any additional AWS services utilized within the ECS environment.

- Liability Waiver:
  * The author(s) of this project cannot be held responsible for any charges that may occur on your AWS account as a result of following these instructions or due to misconfiguration of ECS settings.

- Cost-Reducing Strategies:
  * To mitigate costs, optimize resource allocation by scaling based on actual demand, monitoring resource utilization, and considering AWS cost-saving features or services.

- Deletion of Resources:
  * Remember to promptly delete ECS clusters, load balancers, and associated resources after completing the project to prevent ongoing charges.

By proceeding with the deployment instructions provided in this documentation, you acknowledge and agree that you understand the potential financial implications and release the author(s) of this project from any liability related to AWS charges incurred during or after following these instructions.

## License

This project is licensed under the MIT License - see the LICENSE file for details.
