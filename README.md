Remove all stopped containers
    docker container prune

-----------------
DOCKERIZE MONGODB
-----------------


Use official mongo image.

    docker run mongo
And this would spin up a container named mongodb based on this image.
    docker run --name mongodb --rm -d mongo


https://hub.docker.com/_/mongo

I wanna publish one of the ports made availableby this Mongo image.
Because as long as the node backend API is not dockerized, this API, this node API, still talks to the database here when it connects as if it would be running on my local host machine. Now if we spin up a Docker container containing the database,

    docker run --name mongodb --rm -d -p 27017:27017 mongo

Restart Backend (node):
   npm i
   node app.js 

The backend node app connects to Mongodb container rather than locally installed mongodb

    docker logs mongodb

See local nodejs application


------------------
DOCKERIZE NODE APP
------------------


There is no official Docker Hub image for our own custom application. So we need to write our own Docker file to build our own image for this project.

Remove all unused images I have on my system
    docker image prune -a


Build image <image-name>= goals-node
    docker build -t goals-node .

Run a container based on the newly created image:
    docker run --name goals-backend --rm goals-node

But it crashes because we have MONGODB running in a container exposing its port, but in the dockerized node backend application I am reaching out to localhost.
  'mongodb://localhost:27017/course-goals',
And that now, since this application is now inside of a container, means that I'm trying to access some other service on this port inside of that same backend container, 
not on my host machine. You'll learn about that in the networking module. And there you'll also learn how you can solve that. There is a special alternative domain or address,
which you should use here and that's host.docker.internal. That's a special address, a special identifier, which is translated to your real local host machine IP by Docker.

Rebuild image since the sourcecode changed.
    docker build -t goals-node .
Run container again
    docker run --name goals-backend --rm goals-node

CONNECTED TO MONGODB log message indicates tha node did connect.
So our backend is now dockerized. It is in it's container and able to talk
to mongodb.

Problem? The React frontend application will not be able to talk to this backend.
From devtools "connection refused" error. So while the backend is running in a container now,we're not publishing the ports which it exposes.

And therefore, the frontend trying to talk to this backend application
on specific ports fails.Now we do have this EXPOSE instruction in the Dockerfile
of the backend application,but as you learned, this alone doesn't do much since it relates to the image.
Instead, you need to publish the ports that should be available on your localhost machine when you run a container.

BACKEND: 
    docker run --name goals-backend --rm goals-node
    docker stop goals-backend
    docker run --name goals-backend --rm -d -p 80:80 goals-node
    docker run --name goals-backend  --add-host=host.docker.internal:172.17.0.1 --rm goals-node

And now the React application, which currently is not dockerized yet, will be able to talk to this backend. So now two of three parts are dockerized only the frontend is missing.

ADD DOCKER NETWORKS FOR EFFICIENT CROSS-CONTAINER COMMUNICATION

At root:
docker network create goals-net
docker network create <network-name>

Now start different containers in network goals-net.
 

Now, we no longer need to publish this port because if containers are in the same network they can communicate with each other anyways.
My localhost machine will not be able to talk to this container through its 
localhost address if the port is not published.
But that's no problem because I don't intend on doing that anyways.
It's enough for me if the three containers can talk to each other.
So, instead of publishing the port, I'll add --network and run this container
in the goals net network here.

At root:<container-name> = mongodb
docker run --name mongodb --rm -d --network goals-net mongo
docker run --name <db-container-name> --rm -d --network <network-name> <image>

The mongodb database container is now part of this network: goals-net

In backend folder: <container-name> = goals-backend
docker run --name goals-backend --rm -d --network goals-net goals-node


'mongodb://hosts.docker.internal:27017/course-goals'
'mongodb://<container-name>:27017/course-goals'
'mongodb://mongodb:27017/course-goals'

Host.docke.internal refers to our local host machine though
and there this port will no longer be available because we're not publishing it anymore, on the started mongodb container.

Instead here, we should use mongodb,the name of that container,
which is part of the same network.So, we need to adjust this code in our backend folder first and thereafter, we can run this container.

rebuild image
docker build -t goals-node .


docker run --name goals-backend --rm -d -p 80:80 goals-node
docker run --name goals-backend --rm -d --network goals-net goals-node
docker run --name goals-frontend --rm -p 3000:3000 goals-react


docker network ls
docker ps -a

--------
FRONTEND
-------
Rebuild image 
docker build -t goals-react .

docker run --name goals-frontend --rm -d -p 3000:3000 goals-react

To Troubleshoot run in attached mode:
docker run --name goals-frontend --rm -p 3000:3000 goals-react


Solution : -it
docker run --name goals-frontend --rm -p 3000:3000 -it goals-react

-------------------------

In all the places in the frontend sourcecode where we reach out to
localhost we need to be the name of my nodes application container:
<node-app-container> = goals-backend

Rebuild image 
docker build -t goals-react .

docker run --name goals-frontend --network goals-net --rm -p 3000:3000 -it goals-react


I still want to publish my port here because I still also want to interact with this app from my local host machine, so that we can test it in the browser here.
But I will also add --network and add this to goals net.
So that this container is also part of the same networkand is able to communicate with the other containers.


Error name not resolved?
and if you keep in mind and understand that all the front end react code here,
this JavaScript code, is actually running in the browser, not on some server.
Node is executed by the node runtime on the server in the container.
React always runs in the browser, it is not executed in a container.


And that means that our nice code here where we reach out to goals backend,
does not run in the container where docker would be able
to translate this, it runs in the browser
and the browser has no idea what goals-backend should be.


And with that changed back, we need to ensure
that on localhost, these end points can be reached.
And that simply means, that we still need to publish port 80
on the backend application, so that that application
is also still available on localhost,
because our front end application needs that access.

Because of the way react works,
and because of the fact that react applications
have browser site JavaScript code, and not JavaScript code
that runs inside of the docker container.
If that would be different,
if we had a second node application
that needs to talk to the first one,
then we could use the container name.

We need code the browser understands, not docker.

stop container
Rebuild image:
    docker build -t goals-react .
Run container
    docker run --name goals-frontend --rm -p 3000:3000 -it goals-react

Go to backend container. stop it. Restart it publishing port 80. 

docker stop goals-backend
docker run --name goals-backend --rm -d -p 80:80 --network goals-net goals-node

we still need the network here, because the node API still also talks to mongodb. And we wanna use a network here, so that we can use the mongodb name here in our node code. And here it works, because this code executes directly
in the docker container. So, here docker is able to help us.

But I also still want to publish port 80 on port 80 on the localhost machine,
so that our separate react application is able to talk to that.

3 containers up and running & talking to each other.
