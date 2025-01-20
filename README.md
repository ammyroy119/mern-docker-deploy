# mern-docker-deploy
# Deploying a MERN App on EC2 with Docker

This guide will walk you through the steps to deploy a MERN (MongoDB, Express, React, Node.js) application on an EC2 instance using Docker.

---

## Steps to Launch an EC2 Instance:

1. **Launch a New Instance:**
   - Go to the AWS Management Console and click on "Launch Instance."
   - ![Launch New Instance Screenshot](URL_TO_SCREENSHOT)

2. **Select AMI and Instance Type:**
   - Choose an Amazon Machine Image (AMI) such as Ubuntu (preferred).
   - Select an instance type, such as a free-tier eligible instance.
   - ![Select AMI Screenshot](URL_TO_SCREENSHOT)

3. **Set Up Key Pair:**
   - Create a new key pair or use an existing one. This key pair will be used to connect to your instance.

4. **Configure Network Settings:**
   - Create a security group and set inbound rules for the required ports:
     - Backend: `5050`
     - Frontend: `5173`
     - Database: Default MongoDB port `27017`
   - For security, restrict access to specific IPs wherever possible. For example:
     - Allow `0.0.0.0/0` for development (not recommended for production).
     - Use your public IP with `/32` to restrict access to your machine only.
   - Configure outbound rules to allow all traffic for outgoing connections.
   - ![Network Settings Screenshot](URL_TO_SCREENSHOT)

5. **Configure Storage and Launch:**
   - Set up storage as required for your application.
   - Click on "Launch Instance."
   - ![Storage Configuration Screenshot](URL_TO_SCREENSHOT)

---

## Connecting to Your EC2 Instance

To connect to your EC2 instance, navigate to the directory containing your `.pem` file and run the following command:

```bash
ssh -i "<your-key-pair>.pem" ubuntu@<EC2-Instance-Public-IP>
```

Replace `<your-key-pair>` with the name of your key pair file and `<EC2-Instance-Public-IP>` with the public IP of your EC2 instance.

---

## Installing Git and Docker on EC2

Run the following commands to install Git and Docker on your EC2 instance:

```bash
# Update the package list
sudo apt update

# Install Git
sudo apt install git -y

# Install Docker
sudo apt install docker.io -y

# Add your user to the Docker group
sudo usermod -aG docker $USER
```

Log out and log back in to apply the Docker group changes.

---

## Application Services

The MERN application consists of two main services:
- **Backend**
- **Frontend**

Each service has its own Dockerfile for creating a Docker image.

### Backend Dockerfile
```dockerfile
FROM node:18.20.4

WORKDIR /app

COPY package*.json .

RUN npm install

COPY . .

EXPOSE 5050

CMD ["npm", "start"]
```

### Frontend Dockerfile
```dockerfile
FROM node:18.20.4

WORKDIR /app

COPY package*.json .

RUN npm install

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev"]
```

**Note:** Ensure that the frontend application points to the correct backend URL. Update the environment variable or API URL in your frontend code to match the backend's IP or domain, such as:

```javascript
const BASE_URL = "http://<EC2-Instance-Public-IP>:5050";
```

---

## Deployment Options

### 1. Without Docker Compose

#### Steps:

1. **Connect to Your EC2 Instance:**
   - Use SSH and your key pair to connect.

2. **Install Required Tools:**
   - Install Git and Docker (instructions above).

3. **Clone the Repository:**
   - Clone your application repository from GitHub:
     ```bash
     git clone <repository-url>
     ```

4. **Build the Backend and Frontend Services:**
   - Navigate to the respective folders and build Docker images for both services:
     ```bash
     docker build -t backend .
     docker build -t frontend .
     ```

5. **Set Up Docker Network and Volumes:**
   - Create a Docker bridge network:
     ```bash
     docker network create app-network
     ```
     [Learn more about Docker networks](https://docs.docker.com/network/)

   - Create a Docker volume for MongoDB:
     ```bash
     docker volume create mongo-data
     ```
     [Learn more about Docker volumes](https://docs.docker.com/storage/volumes/)

6. **Run Containers:**
   - Start a MongoDB container:
     ```bash
     docker run -d --name mongo --network app-network -v mongo-data:/data/db mongo
     ```

   - Start the backend container:
     ```bash
     docker run -d --name backend --network app-network -p 5050:5050 backend
     ```

   - Start the frontend container:
     ```bash
     docker run -d --name frontend --network app-network -p 5173:5173 frontend
     ```

### 2. With Docker Compose (Optional)
   - Create a `docker-compose.yml` file to manage all services easily.

---

## Accessing the Application

- Frontend: `http://<EC2-Instance-Public-IP>:5173`
- Backend: `http://<EC2-Instance-Public-IP>:5050`

---

Feel free to expand this guide with additional explanations or troubleshooting steps as needed!

