#Etapa 1
#Utilizo node con alpine para la construcción de la app React
FROM node:18-alpine AS build
#Me posiciono en el directorio /app y lo establezco como dir de trabajo
WORKDIR /app
#Copio el archivo de dependencias primero por si cambian las dependencias (buena practica - orden de las operaciones)
COPY package*.json ./
#Instalo las dependencias de la aplicación React utilizando npm
RUN npm install
#Copio el resto de los archivos - Luego de las dependencias, buena práctica
COPY . .
#Construyo la aplicación React - Genera los archivos estáticos en /app/build
RUN npm run build

#Etapa 2
#Utilizando la estrategia Multistage Building...
#Utilizo una segunda imagen, ahora basada en Nginx con alpine para el servidor del front
FROM nginx:1.23-alpine
#Copio los archivos estáticos construidos con npm al directorio de nginx
COPY --from=build /app/build /usr/share/nginx/html
#Expongo el puerto 80 que utilizará nginx
EXPOSE 80
#Comando para ejecutar Nginx en modo foreground y evitar que se detenga
CMD ["nginx", "-g", "daemon off;"]