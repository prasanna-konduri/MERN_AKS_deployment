FROM node:18.3.0-alpine3.15
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE  3001
CMD ["node", "index.js"]