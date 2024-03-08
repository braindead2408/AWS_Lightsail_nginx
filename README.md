# AWS_Lightsail_nginx
AWS Lightsail guide to set up Reverse proxy Server using NGINX.

Install Docker
```sudo yum install docker```
``` sudo systemctl start docker``` Start Docker Daemon.
``` sudo systemctl status docker``` Check Status to see if it's started or not prooperly.

Install Docker Compose with executable permission
```
sudo curl -SL https://github.com/docker/compose/releases/download/v2.24.6/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

Install Amazon CLI with Lightsail plugin

```curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"```
```  unzip awscliv2.zip```
  ```sudo ./aws/install```

Lightsail Plugin installation
```curl "https://s3.us-west-2.amazonaws.com/lightsailctl/latest/linux-amd64/lightsailctl" -o "/usr/local/bin/lightsailctl"```
```sudo chmod +x /usr/local/bin/lightsailctl```

Clone Repo
```git clone https://github.com/awsgeek/lightsail-containers-nginx.git```

Change directory to Project Directory
```cd lightsail-containers-nginx```

Add line to app.py in flask directory
	```if __name__ == "__main__":
	   app.run(host="0.0.0.0", port=5000)```
	   
Add port expose to DockerFile in Flask and jinja2 update command too.
	```EXPOSE 5000/tcp
	RUN pip install --upgrade Flask Jinja2 #after pip install requirements.txt```
	
Build the flask container
```docker build -t flask-container ./flask```

Run this Container
```docker run -p 5000:5000 flask-container```

Curl to localhost:5000 to check if its working.
```curl localhost:5000```
Don't worry if it's not showing any output here it'll show after Docker-compose.

cd to nginx directory
```cd /nginx```
```docker build -t nginx-container ./nginx```

Docker-compose the nginx file
```docker-compose up –-build```

Curl and test localhost at 80
```curl localhost```

DO AWS Configure by configuring by access key before connectiong lightsail to aws account 

Create container service in lightsail 
```aws lightsail create-container-service --service-name sample-service --power small --scale 1```

monitor to see if it's ready yet
```aws lightsail get-container-services --service-name sample-service```

Give adminaccess role through IAM roles if there is not yet make one and give below step will work.

Push flask application to container service(note deployment number)
```aws lightsail push-container-image --service-name sample-service --label flask-container --image flask-container```

Push nginx container to container service(note deployment number)
```aws lightsail push-container-image --service-name sample-service --label nginx-container --image nginx-container```

Create new containers.json file 

```{
   "sample-nginx": {
      "image": ":sample-service.nginx-container.Y",
      "command": [],
      "ports": {
            "80": "HTTP"
      },
      "environment": {
            "NGINX_ENVSUBST_OUTPUT_DIR": "/etc/nginx",
            "FLASK_HOST": "localhost",
            "FLASK_PORT": "5000"
      }
   },
   "sample-flask": {
      "image": ":sample-service.flask-container.X",
      "ports": {
            "5000": "HTTP"
      }
   }
  ``` 
Create new file public-endpoint.json
```{
    "containerName": "sample-nginx",
    "containerPort": 80
}
```

Deploy the containers to container service

```aws lightsail create-container-service-deployment --service-name sample-service --containers file://containers.json --public-endpoint file://public-endpoint.json```

Monitor the container service
```aws lightsail get-container-services --service-name sample-service```

Done when you see running. Task is completed.

Cleanup::

```aws lightsail delete-container-service --service-name sample-service```

Wait until deletion and done.

Thank You..!😁😁







