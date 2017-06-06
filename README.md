# remnde-docker

contains:
* alpine
* ....

## Installation and Configuration
```
cp docker-compose.yml-template docker-compose.yml
vi docker-compose.yml
./do up
./do logs
./do login    # jump in to the container
```
Point your browser to the redmine port an configure redmine.

## Create image:

     ./do build                # build the image

## Create and start a container from the image

     vi ./docker-compose.yml   # modify the path of your local source directory in docker-compose.yml
     ./do up                   # craete a container from the image an start it 

## Work with the conatiner:

     ./do stop
     ./do start
     ./do login                # jump in to the container
     ./do rm                   # remove the container

