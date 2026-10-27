FROM node:20-alpine 

WORKDIR /app

COPY . .

RUN npm install ejs

CMD ["node", "server.js"]

EXPOSE 3000
