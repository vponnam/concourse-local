#### Instructions to add a darwin worker to tsa running inside docker container.

**Step 1.** Install the stable from docker compose [here](https://docs.docker.com/docker-for-mac/install/#what-to-know-before-you-install)

Make sure it's installed properly by typing `docker-compose -v`

		Venkatas-MacBook-Pro:docker ponnam$ docker-compose -v
		docker-compose version 1.14.0, build c7bdf9e
		
**Step 2** Get the docker-compose manifest file

		git clone https://github.com/vponnam/concourse-local.git
		cd concourse-local/install/docker/
		
###### Note: Please make sure to change the default creds that were specifed in **docker-compose.yml**

**Step 3** Generate the reuired keys as below for TLS

		mkdir -p keys/web keys/worker

		ssh-keygen -t rsa -f ./keys/web/tsa_host_key -N ''
		ssh-keygen -t rsa -f ./keys/web/session_signing_key -N ''
		ssh-keygen -t rsa -f ./keys/worker/worker_key -N ''

		cp ./keys/worker/worker_key.pub ./keys/web/authorized_worker_keys
		cp ./keys/web/tsa_host_key.pub ./keys/worker
		

**Step 4** Generate key pair Darwin worker and add it to authorized_keys

		ssh-keygen -t rsa -f ./keys/worker/drw_worker_key -N ''
		cat ./keys/worker/drw_worker_key.pub >> ./keys/web/authorized_worker_keys

**Step 5** Time to Fireup the docker containers (tsa_web, postgre/db and a linux worker)

		export CONCOURSE_EXTERNAL_URL=http://192.168.99.100:8080
		docker-compose up
		
**Step 6** Verify that you can access web_api through localhost:8080 from your browser

Login page should appear like below: 
![Login Page](https://raw.github.com/vponnam/concourse-local/master/install/images/UI_ScreenShot.png "Logo Title Text 1")

Make sure that you can login to web using the creds specified in docker-compose.yml

**Adding Darwin worker**
		
**Step 7** From a different terminal session start the worker using below command
		
		cd concourse-local/install/docker/keys/worker
		
		sudo concourse worker \
		--work-dir /opt/concourse/worker \
		--tsa-host 127.0.0.1 \
		--tsa-port 8222 \
		--tsa-public-key ./keys/worker/tsa_host_key.pub \
		--tsa-worker-private-key ./keys/worker/drw_worker_key

**Testing** Test the setup. From a different terminal session run the below commands

		fly -t my-local login -c http://localhost:8080
		Use the creds from docker-compose.yml
		
		fly sp -t my-local -p test-pipeline -c test.yml

**Verifying the workers** `fly -t my-local workers` command should result the below 2 workes
	
		containers  platform  tags  team  state
		0           darwin    none  none  running
		0           linux     none  none  running
	
Alright! we're all set, just choose the required worker and then fly!!

##### Tips

*Cheking staled containers*
	
			docker ps
			
			CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                            NAMES
			f6292424ae27        concourse/concourse   "/usr/local/bin/du..."   17 minutes ago      Up 6 minutes                                                         github_concourse-worker_1
			18314c3b1d85        concourse/concourse   "/usr/local/bin/du..."   17 minutes ago      Up 6 minutes        0.0.0.0:8080->8080/tcp, 0.0.0.0:8222->2222/tcp   github_concourse-web_1
			8a914a1c084b        postgres:9.5          "docker-entrypoint..."   17 minutes ago      Up 6 minutes        5432/tcp                                         github_concourse-db_1


			docker stop f6292424ae27 18314c3b1d85  8a914a1c084b
			 
*When not in use, docker deamon can be stopped by clicking on `Quit Docker`*
![Stop Docker](https://raw.github.com/vponnam/concourse-local/master/install/images/Docker_ScreenShot.png "Logo Title Text 1")