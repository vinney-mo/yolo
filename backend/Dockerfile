FROM node:16-alpine3.16

RUN mkdir /backend

WORKDIR /backend

COPY . .

RUN npm install

EXPOSE 5000

CMD ["npm","start"]