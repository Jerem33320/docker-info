 Commandes DOCKER:

- docker
- docker —version
- docker ps   (afficher info et s’assurer docker daemon is running)

IMAGES ET CONTAINER:
Image is a template for creating env of your choice- Snapshot - has everything need to run your Apps ( OS, Software, AppCode)
Container is running instance of an image

———hub.docker.com——— (registre: place where you can download liste d’images)

- docker pull nginx   (ça download l’image nginx)
- docker images   (affiche la liste d’images)

- docker run nginx:nom_du_tag   (run a container depuis l’image).  (Ctrl + C pour sortir du container) (on peut le laisser tourner en laissant ce terminal ouvert)
- docker container ls   (infos tels que le PORTS …)   (docker ps est plus rapide)
- docker run -d nginx:nom_du_tag   (detached mode)


- docker ps -a (voir container)

- docker rm id


Les ‘hello world’ ne se voit plus car ils se stoppent auto, il faut pas oublier de les rm ou de —rm en les lançant.

- Le "-it" se mettre dans le container, "exit" pour sortir.

EXPOSING HOST - FROM THE HOST TO THE CONTAINER exemple localhost:8080 mapped to port TCP 80 du container nginx
- docker stop docker_id
- docker run -d -p 8080:80 nginx:latest   (pas obligé de mettre latest, il y est par défault)
- docker ps   (mtnt le port affiche l’ost mapped sur le port TCP du container)
localhost:8080 sur notre machine affiche mtnt la page nginx!   (bien sûr on peut mettre le port host que l’on veut)

EXPOSING MULTI PORT
- docker run -d -p 8080:80 -p 3000:80 nginx:latest
- docker ps

MANAGING CONTAINER
- docker stop docker_name   (works with id and name)
- docker start docker_name
- docker ps —help
- docker ps -a   (all)
- docker rm docker_name (delete container)
- docker ps -a   (see its no longer in the list)
- docker ps -aq   (all and quiet : all the id)
- docker rm $(docker ps -aq)     (DELETE ALL the containers)   (on ne peut pas remove si un container is running)


NAMING CONTAINERS
- docker run —name website -d -p 3000:80 nginx:latest
- docker ps   (u can see the name: website and not a random one)
- docker stop website

DOCKER PS —FORMAT
- docker run —name website  -d -p 3000:80 nginx:latest
- docker ps —format=‘ID\t{{.ID}}\nNAME\t{{.Names}}\nIMAGE\t{{.Image}}\nPORTS\t{{.Ports}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.CreatedAt}}\nSTATUS\t{{.Status}}\n’
- export FORMAT=‘ID\t{{.ID}}\nNAME\t{{.Names}}\nIMAGE\t{{.Image}}\nPORTS\t{{.Ports}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.CreatedAt}}\nSTATUS\t{{.Status}}\n’   (on créé variable FORMAT)
- docker ps
- docker ps —format=$FORMAT   (on assigne le format à la variable FORMAT)


VOLUMES
Nous permet d’échanger la donnée. (files or folders) entre host et container et aussi entre les containers
Sur le site hub.docker.com on va sur la page nginx et on scroll down jusqu’à « Hosting some simple static content »
-v  : volume
ro : read only

On ouvre vscode et on créé un fichier index.html avec <h1> HELLO WORLD </h1> dans un dossier website sur le desktop.
- cd into the website folder
- docker run —name website -v $(pwd):/usr/share/nginx/html:ro -d -p 8080:80 nginx:latest   (on mount le dossier website sur nginx container)
- localhost:8080 affichera alors l’intérieur de la balise h1 du fichier index.html
- docker exec -it website bash   (execute en dynamiq a l’intérieur du container)
- mtnt qu’on est à l’intérieur du container (root@id) on peut:
- cd /usr/share/nginx/html/
- ls -al   (on voit bien notre fichier index.html)
on peut faire:
- touch about.html   (on ne peut pas car on est dans un container read only)
- exit
- docker rm -f website (supprime le container mm sil est en train de tourner)

- docker run —name website -v $(pwd):/usr/share/nginx/html -d -p 8080:80 nginx:latest   (on enleve le ro read only pour pouvoir créer à l’intérieur du container)
on peut alors créer un fichier about.html dans le dossier website sur le desktop

NEW SITE
on peut alors aller sur startbootrap, dézipper un site depuis le github de leur site, puis copier coller dans notre dossier website ( après avoir enlevé le fichier index.html et about.html existant)


VOLUMES BETWEEN CONTAINERS (copier site)
- docker run —name website-copy —volumes-from website -d -p 8081:80 nginx
- docker ps —format=$FORMAT   (on voit bien les 2 containers running)
on voit bien que localhost:8080 et localhost:8081 affichent le mm site donc échange de data réussie



DOCKERFILE
build our own images
- docker run —name website -v $(pwd):/usr/share/nginx/html -d -p 8080:80 nginx
- ll
- docker image ls
- docker ps —format=$FORMAT 
on a une image et un container
on ouvre sur vscode le folder website
on créer Dockerfile à la racine et on écrit:

FROM nginx:latest
ADD .  /usr/share/nginx/html

(simple dockerfile qui produit une image… CTRL + S !! )

DOCKER BUILD
toujours dans le dossier website depuis le terminal
- docker build —tag website:latest .
on vient de build une image 2 STEP de notre dockerfile
- docker image ls
on peut voir nos 2 images nginx et website
- docker ps —format=$FORMAT 
- docker rm -f website
on supprime le container
- docker ps -a
tout est vide
- docker image ls
on voit bien nos 2 images
- docker run —name website -p 8080:80 -d website:latest
on voit bien que notre image website run
- docker ps —format=$FORMAT 
on voit bien que website:latest is running

localhost:8080 is working!



NODE  ET EXPRESS
revenir sur le desktop
- mkdir user-service-api
- cd user-service-api
- npm init
- npm i express
vscode
touch index.js

paste: 
const express = require('express')
const app = express()

app.get('/', function (req, res) => res.json([{
  	name: ‘Bob’,
	email: ‘bob@gmail.com’
}]))

app.listen(3000)

- node index.js


DOCKERFILE FOR OUR API
on créé Dockerfile à la racine de user-service-api

FROM node:latest              (limage sur hub.docker)
WORKDIR /app
ADD .  .        (on ajoute dans l’app directory créé)
RUN npm install
CMD node index.js

dans le terminal, user-service-api:
- docker build -t user-service-api:latest .
…build the 5 STEPS

RUNNING CONTAINERS FOR OUR API
- docker image ls
- docker run —name user-api -d -p 3000:3000 user-service-api:latest
- docker ps —format=$FORMAT 
on peut voir 2 containers, le website et l’api

localhost:3000 (containeriser une app express avec docker, done!)


.DOCKERFILEIGNORE
à la racine de user-service-api
touch .dockerignore
y écrire:
node_modules
Dockerfile
.git

dans le terminal
- docker build -t user-service-api:latest .


CACHING AND LAYERS
- npm i react webpack gulp grunt (for example)
- docker build -t user-service-api:latest .
construction des 5 steps
some steps are using cache, longer cause we have installed dependancies
- docker rm -f user-api
- docker run —name user-api -d -p 3000:3000 user-service-api:latest
on peut ajouter par exemple d’autres user dans index.js puis refaire
- docker build -t user-service-api:latest .
on peut voir que ça reconstruit les 5 steps et que c’est toujours aussi long(réinstallation des dépendances…). on peut faire mieux en profitant du cache!
on va donc modifier le Dockerfile:

FROM node:latest
WORKDIR /app
ADD package*.json ./
RUN npm install
ADD . .
CMD node.index.js

- docker build -t user-service-api:latest .
construction des 6 STEPS
cette fois si si on ajoute d’autres user et qu’oin relance la commande, ça ira beaucoup plus vite grâce à l’utilisation du cache!
- docker ps (pour voir les images)
- docker rm -f user-api (stop et rm)
- docker run —name user-api -d -p 3000:3000 user-service-api:latest
localhost:3000 affiche les users de index.js


ALPINE (alpine linux distribution. alpine.org)
improve image size with docker
- docker image ls
on voit que user-service-api est presque à 1Gigabits
sur docker hub pull alpine image en faisant:
- docker pull node:lts-alpine     (lts for latest)  ( ça c’est pour diminuer l’image node)
- docker image ls     (on peut voir le TAG lts-alpine et surtout SIZE qui a diminué)
- docker pull nginx:alpine     ( ça c’est pour diminuer l’image nginx)
- docker image ls
on voit 2 images nginx une dont la SIZE est plus plus grosse que alpine.

on peut également changer la SIZE grâce à alpine pour user-service-api et website
on modifie le Dockerfile de user-service-api

FROM node:alpine
WORKDIR /app
ADD package*.json ./
RUN npm install
ADD . .
CMD node.index.js

et dans le terminal:
- docker image ls
- docker image rm id_image_node
- docker pull node:alpine
- docker image ls
on voit l’image node avec SIZE petite
- docker build -t user-service-api:latest .
(même manip dans website)


TAGS AND VERSIONS
allows you to control image version
avoid breaking changes
safe

imaginons nous executons la commande docker pull node:alpine (avec node v8) et qlq jours plus tard nous refaisons docker pull node:alpine (avec node v12)
ça peut avoir bcp de breaking changes. la meilleur façon de faire est d’avoir full control par exemple: spécifier la version: docker pull node: 8-alpine
on trouve les versions dans docker hub

par exemple dans website, on modifie le Dockerfile en spécifiant la v de nginx:

FROM nginx:1.17.2-alpine
ADD . /usr/share/nginx/html


RUNNING CONTAINERS AND TAGS
- docker build -t website:latest .
on peut voir la version nginx
on fait le mm build pour user-service-api
- docker image ls
- docker ps -a
- docker rm -f user-api
- docker run —name user-api -d -p 3000:3000 user-service-api:latest     (on run le container)


TAGGING OVERRIDE
- docker image ls
on peut voir des images <none>
a chaq fois qu’on fait des changement avec le mm tag on doit override

- docker build -t amigoscode-website:latest .
- docker tag amigoscode-website:latest amigoscode-website:1
- docker imge ls
on peut voir la version 1 en TAG
on peut faire des modif dans le website html
- docker build -t amigoscode-website:latest .
- docker tag amigoscode-website:latest amigoscode-website:2
on peut voir 2 version 
on peut run 2 containers v1 et v2

RUNNING CONTAINER WITH DIFFERENT TAG
- docker ps
- docker rm -f website
- docker rm -f user-api
no container running

- docker run —name amigoscode-latest -p 8080:80 -d amigoscode-website:latest
- docker run —name amigoscode-2 -p 8081:80 -d amigoscode-website:2
- docker run —name amigoscode-1-p 8082:80 -d amigoscode-website:1

on peut checker les localhost mtnt


DOCKER REGISTRY
highly scalable server side app that stores and lets you distribute Docker images
used in your CD/CI Pipeline
run your app

on a des images sur notre host et on les envois en push sur le docker registry

Private/public: docker hub, quay.io et amazon ecr

on va utiliser docker hub (il faut être sign in)
dans la nav on click sur repositories, « create repository »    (presque comme sur github)

on va prendre les 3 images amigoscode-website (latest, v1 et v2) et les push sur notre repo docker

			FROM               TO the repo docker
- docker tag amigoscode-website:1 amigoscode/website:1
- docker tag amigoscode-website:2 amigoscode/website:2
- docker tag amigoscode-website:latest amigoscode/website:latest

- docker login  (on écrit username et mdp)
mtnt on peut docker push
- docker push amigoscode/website:1
its pushing! (same for v2 et latest)
si on va voir dans notre docker hub on voit bien nos 3 images


PULLING OWN IMAGES
dans docker hub search your repo
- docker image ls
- docker rmi amigoscode/website:1    (2 et latest)   (rmi remove image)
- docker pull amigoscode/website
- docker image ls
on voit bien qu’il à pull la dernière image de notre repo sur docker hub
- docker run —name website -p 9000:80 -d amigoscode-website
localhost:9000 ça marche!


DOCKER INSPECT
- docker ps —format=$FORMAT
on peut voir 2 containers running
- docker inspect container_id

DOCKER LOGS
- docker logs container_id_user_api (par exemple) on voit le console.log du fichier index.js
pr le website, on voit les requetes
on peut ajouter -f pour follow les logs lorqu’on rafraichit la page sur localhost

DOCKER EXEC
- docker exec -it container_id /bin/bash
ça fail si quand on fait inspect on va ds CMD et on voit qu’on n’est pas en bash
on peut changer le bash en sh et on se retrouve ds le container

	
COURS DOCKER----------------
docker ps -a (voir container)
docker rm id

Les ‘hello world’ ne se voit plus car ils se stoppent auto, il faut pas oublier de les rm ou de —rm en les lançant.

- docker container run --name matter01 --hostname matter01 -p 85:8065 -d --restart always mattermost/platform
Tricky localhost:TCP
