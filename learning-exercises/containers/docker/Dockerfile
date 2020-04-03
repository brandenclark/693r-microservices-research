# FROM is where we build off a base image
FROM node:10

# setting the working directory of the image
WORKDIR /usr/src/app

# copy files over to the image
COPY package*.json ./

# run command on the image to install dependencies
RUN npm install

# copy over the rest of the code
COPY . .

# set open network ports
EXPOSE 8080

 # start the server on the image
CMD ["node", "server.js"]

