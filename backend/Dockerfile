FROM node:18-alpine

WORKDIR /app

COPY package.json .
COPY package-lock.json .

RUN npm ci --omit=dev

COPY . .

EXPOSE 5000

ENV PORT=5000

CMD ["npm", "start"]
