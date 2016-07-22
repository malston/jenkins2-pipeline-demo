#Jenkins 2.0 Cloud Foundry Pipeline Demo

Demonstrates a Jenkins 2.0 Pipeline to build a Java project and deploy the project to a running instance of Pivotal Cloud Foundry.

## To Run this Demo

* Clone this project
  ```
  git clone https://github.com/pivotalservices/jenkins2-pipeline-demo
  cd jenkins2-pipeline-demo
  ```

* Run the following commands to build and run the Docker container
  ```
  docker-machine create jenkins2-pipeline-demo --driver virtualbox --virtualbox-memory "11000" --virtualbox-disk-size "100000"
  eval "$(docker-machine env jenkins2-pipeline-demo)"
  docker build -t pivotal/jenkins2-cf-pipeline-demo - < Dockerfile
  docker run -i -t -p 8080:8080 -p 50000:50000 --name=jenkins-pipeline-demo -v /var/jenkins_home pivotal/jenkins2-cf-pipeline-demo
  ```

* The default admin password will appear in the console as logs from Jenkins. This password is required to run through the initial setup of Jenkins. You can also retrieve the Jenkins Admin password by executing the following: `docker exec -it $(docker ps -l -q) cat /var/jenkins_home/secrets/initialAdminPassword` where ``docker ps -l -q`` will retrieve the last container ID.

* Set `JENKINS_URL` environment variable
  ```
  eval "$(docker-machine env jenkins2-pipeline-demo)"
  export JENKINS_URL=$(echo \"$(echo $DOCKER_HOST)\"|
              \sed 's/tcp:\/\//http:\/\//g'|
              \sed 's/[0-9]\{4,\}/8080/g'|
              \sed 's/\"//g')
  ```

* Create the demo pipeline job using the CLI:
  ```
  curl -O http://192.168.99.100:8080/jnlpJars/jenkins-cli.jar
  java -jar jenkins-cli.jar create-job sample-spring-cloud-svc-ci < config.xml --username admin --password <REPLACE-WITH-PASSWORD-ABOVE>
  ```

* Navigate to the IP of the Docker VM to access Jenkins.
  ```
  open $(echo \"$(echo $DOCKER_HOST)\"|
              \sed 's/tcp:\/\//http:\/\//g'|
              \sed 's/[0-9]\{4,\}/8080/g'|
              \sed 's/\"//g')
  ```
The pipeline created using the above script should be located at `$JENKINS_URL/job/sample-spring-cloud-svc-ci`

## Docker Compose

To run this demo with Sonar and Nexus you can startup all the containers as once using Docker Compose

  `docker-compose up -d`

If everything is configured correctly, each of the container images we built earlier will be launched within their own VM container on Docker and networked for automatic service discovery. You will see a flurry of log output from each of the services as they begin their startup sequence. This might take a few minutes to complete, depending on the performance of the machine you’re running this demo on.

To see the log output from the cluster, you can run the following command.

  `docker-compose logs`

Once the startup sequence is completed, you can navigate to Jenkins using the `$JENKINS_URL` or copy and paste the following command into the terminal where Docker can be accessed using the $DOCKER_HOST environment variable.
```
$ open $(echo \"$(echo $DOCKER_HOST)\"|
            \sed 's/tcp:\/\//http:\/\//g'|
            \sed 's/[0-9]\{4,\}/8080/g'|
            \sed 's/\"//g')
```

## To Build this project

This project builds a custom Docker container with Jenkins plugins automatically installed, and the CF CLI automatically installed and configured so it can be used by Pipelines.

  `docker build -t pivotal/jenkins2-cf-pipeline-demo .`

## To Build Cloudbees version of this project

  `docker build -f Dockerfile_cloudbees -t pivotal/cb-cf-pipeline-demo .`

### Initilization scripts (Groovy init scripts that run on Jenkins startup)
- `init_02_pull_remote_pipeline_global_libs.groovy`: Pulls Pipeline Global Library from a GitHub repo into the Jenkins Pipeline repo, making the shared Pipeline Libraries available immediately after initial Jenkins startup
- `init_03_add_sa_credentials.groovy`: Sets up global credentials, retrieving sensitive data from environmental variables; includes credential to connect to GitHub Enterprise
- `init_04_add_ghe_server.groovy`: Configures GitHub Enterprise API endpoint to be used with the GitHub Organization Folder plugin
- `init_05_mail_server_config.groovy`: Configures mail server to allow sending email from Jenkins
- `init_08_audit_trail_config.groovy`: Configures Audit Trail plugin to send data to Logstash via syslog; and on to Elasticsearch to be displayed in custom CJOC analytics dashboard
- `init_20_top-level-folders.groovy`: Creates a folder with a custom icon from the custom [Simple Build plugin](https://github.com/kmadel/simple-build-for-pipeline-plugin), used to hold example job(s)
- `init_21_simple_build_pipeline.groovy`: Creates an example Pipeline job that uses DSL from the [Simple Build plugin](https://github.com/kmadel/simple-build-for-pipeline-plugin)
- `init_99_save.groovy`: Ensures any previous configuration changes are saved  


## References

* Docker Machine Installation Instructions: https://docs.docker.com/machine/install-machine/
* Jenkins Docker Image README:  https://github.com/jenkinsci/docker/blob/master/README.md
