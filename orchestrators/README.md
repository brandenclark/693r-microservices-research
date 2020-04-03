# Instructions

```bash
# turn local computer into a swarm cluster
docker swarm init
# build our image
docker-compose build
# create a stack to deploy the app using our compose file
docker stack deploy nodeapp -c docker-compose.yml
# scale service to run 4 replicas of our app
docker service scale nodeapp_web=4
# stop nodeapp in the swarm
docker stack rm nodeapp
# leave our local swarm
docker swarm leave --force
```
