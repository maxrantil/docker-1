# docker-1

# Hive Docker project

### 1. Get the hello-world container from the Docker Hub, where it’s available.

```bash
docker pull hello-world
```

### 2. Launch the hello-world container, and make sure that it prints its welcome message, then leaves it.

```bash
docker run hello-world
```

### 3. Launch a nginx container, available on Docker Hub, as a background task. It should be named overlord, be able to restart on its own, and have its 80 port attached to the 5000 port of your machine. You can check that your container functions properly by visiting http://localhost:5000 on your web browser.

```bash
docker run -d --name overlord --restart always -p 5000:80 nginx
```

breakdown:

- **`d`** tells Docker to start the container in detached mode (i.e., in the background).
- **`-name overlord`** specifies the name of the container as "overlord".
- **`-restart always`** tells Docker to automatically restart the container if it fails for any reason.
- **`p 5000:80`** maps port 80 of the container to port 5000 on the local machine.
- **`nginx`** specifies the name of the image to use (in this case, the official nginx image from Docker Hub).

Once the container is running, you can check that it functions properly by visiting **`http://localhost:5000`** in your web browser. You should see the default nginx welcome page. If you see that, then your container is functioning properly.

### 4. Get the internal IP address of the overlord container without starting its shell and in one command.

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' overlord
```

This command uses the **`docker inspect`** command to retrieve detailed information about the **`overlord`** container, and the **`-f`** option allows you to format the output of the **`docker inspect`** command using a Go template.

The Go template **`{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}`** iterates over the network settings of the container and returns the IP address of the network interface associated with each network. Since the **`overlord`** container only has one network interface, this template will return the IP address of that interface.

Once you run the command, it will output the IP address of the **`overlord`** container's network interface. You can use this IP address to connect to services running inside the container from other containers or from your local machine.

### 5. Launch a shell from an alpine container, and make sure that you can interact directly with the container via your terminal, and that the container deletes itself once the shell’s execution is done.

```bash
docker run -it --rm alpine sh
```

### 6. From the shell of a debian container, install via the container’s package manager everything you need to compile C source code and push it onto a git repo (of course, make sure before that the package manager and the packages already in the container are updated). For this exercise, you should only specify the commands to be run directly in the container.

```bash
apt-get update
apt-get upgrade
apt-get install -y build-essential git
```

These commands update the package manager, upgrade all installed packages to their latest versions, and then install the following packages:

- **`build-essential`**: A package that includes various compilers and build tools, including **`gcc`**, **`g++`**, and **`make`**.
- **`git`**: The Git version control system.

With these packages installed, you should have everything you need to compile C source code and push it onto a Git repo. You can use the **`gcc`** compiler to compile your C code, and the **`git`** command to manage your Git repo. Note that you will need to set up your Git repo and configure your environment appropriately to use Git successfully.

### 7. Create a volume named hatchery.

```bash
docker volume create hatchery
```

This command uses the **`docker volume create`** command to create a new Docker volume with the name **`hatchery`**. Once the volume is created, you can use it to store data that can be shared between containers or persisted across container restarts. You can also use the **`docker volume ls`** command to list all available Docker volumes, including the **`hatchery`** volume you just created.

### 8. List all the Docker volumes created on the machine. Remember VOLUMES.

```bash
docker volume ls
```

By default, Docker uses the **`local`** driver to create volumes, which means that volumes are stored on the host filesystem in the **`/var/lib/docker/volumes`** directory (on Linux hosts). However, you can also use other drivers to create volumes that are stored in other locations or managed in different ways.

### 9. Launch a mysql container as a background task. It should be able to restart on its own in case of error, and the root password of the database should be Kerrigan. You will also make sure that the database is stored in the hatchery volume, that the container directly creates a database named zerglings, and that the container itself is named spawning-pool.

```bash
docker run -d \
  --name spawning-pool \
  -e MYSQL_ROOT_PASSWORD=Kerrigan \
  --restart=always \
  -v hatchery:/var/lib/mysql \
  -e MYSQL_DATABASE=zerglings \
  mysql:latest
```

This command uses the **`docker run`** command to start a new container from the **`mysql`** image. The **`-d`** option tells Docker to start the container in detached mode (i.e., as a background task). The **`--name`** option specifies the name of the container as **`spawning-pool`**. The **`-e MYSQL_ROOT_PASSWORD`** option sets the root password of the MySQL database to **`Kerrigan`**. The **`--restart=always`** option tells Docker to automatically restart the container if it crashes or is stopped. The **`-v hatchery:/var/lib/mysql`** option creates a new volume mount, linking the **`hatchery`** volume to the **`/var/lib/mysql`** directory inside the container, which is where MySQL stores its data. Finally, the **`-e MYSQL_DATABASE`** option specifies the name of the new database that should be created inside the container as **`zerglings`**.

Once the container is started, you can connect to the MySQL database running inside the container by using the **`mysql`** command line tool or any other MySQL client. To access the MySQL database from outside the container, you will need to expose the appropriate ports and configure the appropriate network settings.

### 10. Print the environment variables of the spawning-pool container in one command, to be sure that you have configured your container properly.

```bash
docker exec spawning-pool env
```

This command uses the **`docker exec`** command to run the **`env`** command inside the **`spawning-pool`**
 container. The **`env`** command prints a list of all the environment variables that are currently set inside the container. By running this command, you can verify that the container has been configured properly and that the required environment variables (such as **`MYSQL_ROOT_PASSWORD`** and **`MYSQL_DATABASE`**) have been set correctly.

### 11. Launch a wordpress container as a background task, just for fun. The container should be named lair, its 80 port should be bound to the 8080 port of the virtual machine, and it should be able to use the spawning-pool container as a database service. You can try to access lair on your machine via a web browser, with the IP address of the virtual machine as a URL. Congratulations, you just deployed a functional Wordpress website in two commands!

```bash
docker run -d \
  --name lair \
  -p 8080:80 \
  --restart=always \
  --link spawning-pool:mysql \
  wordpress:latest
```

This command uses the **`docker run`** command to start a new container from the **`wordpress`** image. The **`-d`** option tells Docker to start the container in detached mode (i.e., as a background task). The **`--name`** option specifies the name of the container as **`lair`**. The **`-p 8080:80`** option binds the container's port **`80`** to the host machine's port **`8080`**, so that you can access the Wordpress site via **`http://<host-machine-ip>:8080`** in a web browser. The **`--restart=always`** option tells Docker to automatically restart the container if it crashes or is stopped. The **`--link spawning-pool:mysql`** option creates a link between the **`lair`** container and the **`spawning-pool`** container, allowing the Wordpress container to access the MySQL database running inside the **`spawning-pool`** container.

Once the container is started, you can access the Wordpress site by visiting **`http://<host-machine-ip>:8080`** in a web browser. The site should be fully functional and ready to use!

### 12. Launch a phpmyadmin container as a background task. It should be named roach-warden, its 80 port should be bound to the 8081 port of the virtual machine and it should be able to explore the database stored in the spawning-pool container.

```bash
docker run -d \
  --name roach-warden \
  -p 8081:80 \
  --link spawning-pool:mysql \
  phpmyadmin/phpmyadmin:latest
```

This command uses the **`docker run`** command to start a new container from the **`phpmyadmin/phpmyadmin`** image. The **`-d`** option tells Docker to start the container in detached mode (i.e., as a background task). The **`--name`** option specifies the name of the container as **`roach-warden`**. The **`-p 8081:80`** option binds the container's port **`80`** to the host machine's port **`8081`**, so that you can access phpMyAdmin via **`http://<host-machine-ip>:8081`** in a web browser. The **`--link spawning-pool:mysql`** option creates a link between the **`roach-warden`** container and the **`spawning-pool`** container, allowing phpMyAdmin to access the MySQL database running inside the **`spawning-pool`** container.

Once the container is started, you can access phpMyAdmin by visiting **`http://<host-machine-ip>:8081`** in a web browser. You should be able to log in to phpMyAdmin with the same credentials as you used for the MySQL database (i.e., username: **`root`**, password: **`Kerrigan`**). From there, you can explore the **`zerglings`** database that was created inside the **`spawning-pool`** container earlier.

### 13. Look up the spawning-pool container’s logs in real-time without running its shell.

```bash
docker logs -f spawning-pool
```

This command uses the **`docker logs`** command with the **`-f`** option to "follow" the logs of the **`spawning-pool`** container in real time. The container's name (**`spawning-pool`**) is specified at the end of the command. This will display the container's logs on your terminal in real-time, without starting the container's shell. To stop following the logs, you can press **`Ctrl+C`** on your keyboard.

### 14. Display all the currently active containers on your machine

```bash
docker ps
```

This command lists all running containers on your machine, along with some basic information such as the container ID, name, image used, and status. If you want to see all containers (not just running ones), you can use the **`-a`** or **`--all`**

### 15. Relaunch the overlord container.

```bash
docker start overlord
```

This command uses the **`docker start`** command with the container name (**`overlord`**) as an argument. This will start the container in the background, with the same settings it had when it was last stopped or exited. If you want to attach to the container's shell after starting it, you can use the **`docker attach`**
 command:

```bash
docker attach overlord
```

This will attach your terminal to the container's shell, allowing you to interact with it directly. To exit the shell and detach from the container, you can use the **`exit`** command.

### 16. Launch a container name Abathur. It will be a Python container, 2-slim version, its /root folder will be bound to a HOME folder on your host, and its 3000 port will be bound to the 3000 port of your virtual machine. You will personalize this container so that you can use the Flask micro-framework in its latest version. You will make sure that an html page displaying "Hello World" with <h1> tags can be served by Flask. You will test that your container is properly set up by accessing, via curl or a web browser, the IP address of your virtual machine on the 3000 port. You will also list all the necessary commands in your repository.

1. Create a file called **`app.py`** in the `$HOME` folder on your host. This file will contain a simple Flask app that serves an HTML page with the text "Hello World" in an **`<h1>`** tag:

```bash
cat > $HOME/app.py <<'EOF'
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return '<h1>Hello World</h1>'

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=3000)
EOF
```

4. Start the interactive `python` container in detached mode `-dit`, named **`Abathur`** with the bind mount and port mappings:

```bash
docker run -dit \
	--name Abathur \
	-v $HOME:/root \
	-p 3000:3000 \
	python:2-slim. Inside the container's shell, install Flask and any necessary dependencies
```

1. Install and start the Flask app inside the container by running the following command:

```bash
docker exec Abathur pip install Flask
```

```bash
docker exec -e FLASK_APP=/root/app.py Abathur flask run --host=0.0.0.0 --port 3000
```

1. This command runs the Flask application in the Abathur container, specifically running the **`flask run`** command with the **`--host`** and **`--port`** options set to bind the application to all available network interfaces on port 3000. The **`-e`** option sets an environment variable **`FLASK_APP`** to specify the path to the root file of the Flask application.

2. Test that the app is working by visiting **`http://<your-vm-ip>:3000`** in a web browser or running the following command in a terminal:

```bash
curl http://localhost:3000
```

You should see the "Hello World" text displayed in an **`<h1>`** tag. To stop and remove the container, you can use the **`docker stop`** and **`docker rm`** commands:

```bash
rm $HOME/app.py
docker stop Abathur
docker rm Abathur
```

### 17. Create a local swarm, your local machine (the school iMac) should be its manager.

```bash
docker swarm init
```

1. This will create a new swarm and make your local machine the manager.
2. If successful, the command will output a message that includes the command to use to join other nodes to the swarm. The command will look something like this:
