# jenkins-docker-timeout-issue
Project to describe how to reproduce the lock of all docker provision queue

## Install your mock server
Copy the content of the mockserver folder on a UNIX server with java installed.

The server mock the docker api `/containers/json` (list container) and `/version` (get version).

You can change or enchance the init.json file wich describe the api. For example you can change the response delay to test the timeout.

For more information about how to configure mockserver, see [the Official Documentation](https://www.mock-server.com/).

Run mockserver by launching command `java -jar mockserver-netty-5.11.1-jar-with-dependencies.jar -serverPort 1099`

## Configure your Jenkins to point on mockserver
Install docker-plugin if it is not installed.

Go to System Configuration -> Nodes -> configure cloud

Add a docker cloud, give it the name docker-mock and point it to your mockserver adress like "tcp://yourMockServerAdress:1099".

You don't need any credentials.

Click to test connection. If `Version = 19.03.12, API Version = 1.40` is displayed, your configuration is good.

Enable the docker cloud, and add a new "Docker Agent Template", put "docker-mock" as label, name and docker image.

Save the configuration.

## Configure logger
Go to Configuration -> System Logs -> add new logger

- Name : docker-plugin
- Logger : com.nirima.jenkins.plugins.docker; Level : ALL
- Logger : io.jenkins.docker; Level : ALL

Save the configuration.

## Create a Job
Create a new freestyle project "jenkins-docker-timeout-issue".

Configure it to run on "docker-mock" node only.

Add a script shell builder with "ls" for example just to have something to be run in the job.

## Get a timeout
If you let the timeout of the docker cloud to 60s and the response delay of `/containers/json` api to 90s, you should get :

`java.net.SocketTimeoutException: Read timed out: No data received within 60000ms.  Perhaps the docker API`

Now stop the job still waiting for node provision, and wait 5 mins then the docker cloud will be enabled again. You can enable it manually if you don't want to wait.

## Block docker-mock cloud provision
Launch the job again but now stop mockserver before the `SocketTimeoutException`. 

Now your job will wait indefinitly and you will not able to provision other node on the docker-mock cloud.

## Block all docker node provisions
Go to Configuration -> Tools and Actions -> Reload configuration from disk

**Now no more docker node can be provisioned on any cloud, you must stop Jenkins JVM and restart to do more tests**








