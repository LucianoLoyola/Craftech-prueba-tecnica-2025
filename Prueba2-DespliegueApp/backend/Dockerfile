#Utilizo Python 3.10 slim
FROM python:3.10-slim
WORKDIR /app
#Instalo dependencias del sistema necesarias para compilar `psycopg2`
RUN apt-get update && apt-get install -y libpq-dev gcc && rm -rf /var/lib/apt/lists/*
#Copio el archivo de dependencias
COPY requirements.txt .
# Instalo dependencias de Python
RUN pip install --no-cache-dir --root-user-action=ignore -r requirements.txt
# Copio el resto de los archivos
COPY . .
# Configuración final
ENV PATH="/usr/local/bin:$PATH"
EXPOSE 8000
# Comando para iniciar el servidor
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "core.wsgi:application"]