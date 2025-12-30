FROM node:18-alpine AS build

WORKDIR /app

COPY package.json .
COPY package-lock.json .

RUN npm ci

COPY . .

RUN NODE_OPTIONS=--openssl-legacy-provider npm run build

# For Production Stage
FROM nginx:stable-alpine AS production

COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]