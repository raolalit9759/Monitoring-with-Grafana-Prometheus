# Monitoring-with-Grafana-Prometheus
Monitoring with Grafana and Prometheus
Project Overview
This project demonstrates the integration of Prometheus and Grafana for monitoring a Node.js application. It includes:

Prometheus: For collecting and querying metrics.
Grafana: For visualizing the metrics collected by Prometheus.
Node.js Backend: A sample application that exposes metrics to be scraped by Prometheus.
Prerequisites
Docker and Docker Compose installed.
Node.js and npm installed.
A GitHub repository to store the project.
Steps to Set Up the Project
1. Clone the Repository
Clone this repository to your local machine:

$ git clone https://github.com/AbhayShukla1907/Monitoring-with-Grafana-Prometheus.git
$ cd Monitoring-with-Grafana-Prometheus
2. Create a Node.js Backend
2.1 Initialize a Node.js Application
$ mkdir backend && cd backend
$ npm init -y
2.2 Install Required Packages
$ npm install express prom-client
2.3 Create the Backend Code
Create a file named server.js:

const express = require('express');
const client = require('prom-client');

const app = express();
const register = client.register;

// Custom metrics
const httpRequestDurationMicroseconds = new client.Histogram({
  name: 'http_request_duration_microseconds',
  help: 'Duration of HTTP requests in ms',
  labelNames: ['method', 'route', 'status'],
});

app.get('/', (req, res) => {
  res.send('Hello, Monitoring!');
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
Start the backend:

$ node server.js
3. Set Up Prometheus
3.1 Create Prometheus Configuration
In the project root, create a file named prometheus.yml:

scrape_configs:
  - job_name: 'node_backend'
    static_configs:
      - targets: ['host.docker.internal:3000']
Screenshot 2025-01-02 191000

4. Set Up Grafana
Grafana will connect to Prometheus and visualize the metrics.

Screenshot 2025-01-02 191157

5. Dockerize the Setup
5.1 Create a Dockerfile for Node.js
Inside the backend directory, create a Dockerfile:

FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
5.2 Create a docker-compose.yml File
In the project root, create the following file:

version: '3.8'
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - '9090:9090'

  grafana:
    image: grafana/grafana
    ports:
      - '3001:3000'

  backend:
    build:
      context: ./backend
    ports:
      - '3000:3000'
6. Run the Services
Start all services using Docker Compose:

$ docker-compose up --build
7. Visualize Metrics in Grafana
Access Grafana at http://localhost:3001.
Log in with default credentials (admin/admin).
Add Prometheus as a data source:
URL: http://prometheus:9090
Create a dashboard and add panels to visualize metrics such as http_request_duration_microseconds.
Deployment Steps to GitHub
Initialize Git in the Project Directory:

$ git init
$ git remote add origin https://github.com/AbhayShukla1907/Monitoring-with-Grafana-Prometheus.git
Add Files and Commit:

$ git add .
$ git commit -m "Initial commit"
Push to GitHub:

$ git push -u origin main --force
Notes
Avoid adding large binaries like prometheus executables directly to the repo. Use .gitignore or Git LFS for such files.
Regularly check logs for troubleshooting (docker-compose logs).
