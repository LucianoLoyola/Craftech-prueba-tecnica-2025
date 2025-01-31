#Utilizo python 3.10 slim para la compilación del backend junto con sus dependencias.
FROM python:3.10-slim AS build
#Me posiciono en /app
WORKDIR /app
#Instalo las dependencias necesarias para la construcción...
RUN apt-get update && apt-get install -y libpq-dev && rm -rf /var/lib/apt/lists/*
#Instalo las dependencias necesarias para la aplicación React
COPY requirements.txt .
RUN pip install --no-cache-dir --root-user-action=ignore -r requirements.txt
#Copio el resto de los archivos
COPY . .



#Utilizando la estrategia Multistage Building...
#Ahora debo generar la imagen de producción
FROM python:3.10-slim
#Me posiciono en /app
WORKDIR /app
#Copio las dependencias instaladas en la etapa de construcción
COPY --from=build /usr/local /usr/local
COPY --from=build /app /app
# Instalo solo las bibliotecas necesarias en producción
RUN apt-get update && apt-get install -y libpq5 && rm -rf /var/lib/apt/lists/*
#Establezco variables de entorno
ENV PATH="/usr/local/bin:$PATH"
#Expongo el puerto de la aplicación
EXPOSE 8000
#Ejecuto la aplicación
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "core.wsgi:application"]