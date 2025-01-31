#Utilizo Python 3.10 versión slim
FROM python:3.10-slim
WORKDIR /app
#Instalo dependencias del sistema necesarias para compilar `psycopg2` (cliente PostgreSQL)
RUN apt-get update && apt-get install -y libpq-dev gcc && rm -rf /var/lib/apt/lists/*
#Copio el archivo de dependencias
COPY requirements.txt .
#Instalo dependencias necesarias para la aplicación
RUN pip install --no-cache-dir --root-user-action=ignore -r requirements.txt
#Copio el resto de los archivos
COPY . .
#Configuración final del PATH
ENV PATH="/usr/local/bin:$PATH"
#Expongo el puerto a utilizar donde escuchará el backend
EXPOSE 8000
#Comando para iniciar el servidor con Gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "core.wsgi:application"]