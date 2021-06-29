FROM node:14.16.1-alpine3.13

WORKDIR /app

COPY package*.json ./ 

RUN npm install

# Aproveitar as camadas cacheadas (package-lock.json e package.json)
COPY . .

# Porta da aplicação
EXPOSE 8080

# CMD é executado no momento da criação do container
CMD ["node", "index.js"]