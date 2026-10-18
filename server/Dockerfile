# install node js
# copy the package file
# install the dependencies
# copy all the file
# now create a build file 
# run the server


FROM  node:22-alpine
WORKDIR /server-app
# two dot with space (. . meaning copy everything to the current dir)
COPY . .
RUN npm i 
EXPOSE 3000
CMD [ "node" , "index.js" ]