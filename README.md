# Apuntes Docker [Documentación Docker](https://docs.docker.com/)  

Use the docker container run command to run a container with the ubuntu image, using the top command. The -t flag allocates a pseudo-TTY, which you need for the top to work correctly.  
  
  __$ docker container run -it ubuntu top__  
  
Inspect the container with:  

  __$docker container exec.__  

Check your running containers with:  

  __$ docker container ls__  
  
  __$ docker container exec -it b3ad2a23fab3 bash__  
  
Run an Nginx server.  

  __$ docker container run --detach --publish 8080:80 --name nginx nginx__ 
  
Run a mongoDB server.  

  __$ docker container run --detach --publish 8081:27017 --name mongo mongo:3.4__  
  
Run docker container stop [container id] for each container in the list:  

  __$ docker container stop d67 ead af5__  
  
Remove the stopped containers.
docker system prune is a really handy command to clean up your system. It will remove any stopped containers, unused volumes and networks, and dangling images.  

  __$ docker system prune__  
  
Create the Dockerfile with the following contents:  

    FROM python:3.6.1-alpine  
    RUN pip install flask  
    CMD ["python","app.py"]  
    COPY app.py /app.py  
  
Build the Docker image:  

  __$ docker image build -t python-hello-world .__  
  
Run docker image ls to verify that your image shows up in your image list.  

  __$ docker image ls__  
  
Run the Docker image.  

  __$ docker run -p 5001:5000 -d python-hello-world__  
  
Check the log output of the container.  

  __$ docker container logs [container id]__  
  
## Push to a central registry  
Navigate to Docker Hub and create a free account, if you haven’t already.
For this lab, you will use the Docker Hub as your central registry. Docker hub is a free service to publicly store available images, or you can pay to store private images.  

Most organizations that heavily use Docker will set up their own registry internally. To simplify things, we will use the Docker Hub, but the following concepts apply to any registry.  

Log in to the Docker registry account by typing docker login on your terminal.  
  __$ docker login__  
  
The Docker Hub naming convention is to tag your image with [dockerhub username]/[image name]. To do this, tag your previously created image python-hello-world to fit that format.  

  __$ docker tag python-hello-world [dockerhub username]/python-hello-world__  
  
Once you have a properly tagged image, use the docker push command to push your image to the Docker Hub registry.  

  __$ docker push [dockerhub username]/python-hello-world__  
The push refers to a repository [docker.io/jzaccone/python-hello-world]

Check out your image on docker hub in your browser.
Navigate to Docker Hub, and go to your profile to see your newly uploaded image.

## Deploy a change  
Update app.py by replacing the string “Hello World” with “Hello Beautiful World!” in app.py.
Your file should have the following contents:

    from flask import Flask

    app = Flask(__name__)

    @app.route("/")
    def hello():
        return "Hello Beautiful World!"

    if __name__ == "__main__":
        app.run(host='0.0.0.0')  
    
Now that your app is updated, you need to rebuild your app and push it to the Docker Hub registry.
First rebuild, this time use your Docker Hub username in the build command.  

  __$  docker image build -t [dockerhub username]/python-hello-world .__  

Notice the “Using cache” for Steps 1-3. These layers of the Docker image have already been built, and docker image build will use these layers from the cache, instead of rebuilding them.  

  __$ docker push [dockerhub username]/python-hello-world__  

There is a caching mechanism in place for pushing layers too. Docker Hub already has all but one of the layers from an earlier push, so it only pushes the one layer that has changed.  

When you change a layer, every layer built on top of that will have to be rebuilt. Each line in a Dockerfile builds a new layer that is built on the layer created from the lines before it. This is why the order of the lines in your Dockerfile is important. We optimized our Dockerfile so that the layer that is most likely to change (COPY app.py /app.py) is the last line of the Dockerfile. Generally for an application, your code changes at the most frequent rate. This optimization is particularly important for CI/CD processes, where you want your automation to run as fast as possible.  

--

To look more closely at layers, you can use the docker image history command of the python image we created.  

  __$ docker image history python-hello-world__  
  
 
## Create your first swarm  
In this step, we will create our first swarm using play-with-docker.  

Navigate to Play-with-Docker.
The first swarm cluster will have three nodes, so click add new instance on the left side three times to create three nodes.
Initialize the swarm on node 1.
$ docker swarm init --advertise-addr eth0
Swarm initialized: current node (vq7xx5j4dpe04rgwwm5ur63ce) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-50qba7hmo5exuapkmrj6jki8knfvinceo68xjmh322y7c8f0pj-87mjqjho30uue43oqbhhthjui \
    10.0.120.3:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

You can think of docker swarm as a special “mode” that is activated by the command: docker swarm init. The --advertise-addr specifies the address in which the other nodes will use to join the swarm.

This docker swarm init command generates a join token. The token makes sure that no malicious nodes join our swarm. We will need to use this token to join the other nodes to the swarm. For convenience, the output includes the full command docker swarm join, which you can just copy/paste to the other nodes.

On both node2 and node3, copy and run the docker swarm join command that was outputted to your console by the last command.
You now have a three-node swarm!

Back on node1, run docker node ls to verify your three-node cluster.
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
7x9s8baa79l29zdsx95i1tfjp     node3               Ready               Active
x223z25t7y7o4np3uq45d49br     node2               Ready               Active
zdqbsoxa6x1bubg3jyjdmrnrn *   node1               Ready               Active              Leader

This command outputs the three nodes in our swarm. The asterisk (*) next to the ID of the node represents the node that handled that specific command (docker node ls in this case).

Our node consists of one manager node and two workers nodes. Managers handle commands and manage the state of the swarm. Workers cannot handle commands and are simply used to run containers at scale. By default, managers are also used to run containers.

All docker service commands for the rest of this lab need to be executed on the manager node (Node1).

Note: While we will control the Swarm directly from the node in which its running, you can control a docker swarm remotely by connecting to the docker engine of the manager by using the remote API or by activating a remote host from your local docker installation (using the $DOCKER_HOST and $DOCKER_CERT_PATH environment variables). This will become useful when you want to remotely control production applications, instead of ssh-ing directly into production servers.
