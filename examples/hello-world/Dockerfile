FROM node:22.2.0-alpine3.18
WORKDIR /app
COPY ./examples/hello-world/index.js .
COPY ./package.json .
RUN npm install
CMD ["node", "index.js"]
EXPOSE 3000
