# Django-Docker

```
# For more information, please refer to https://aka.ms/vscode-docker-python
FROM python:3.8-slim-buster
# Keeps Python from generating .pyc files in the container
ENV PYTHONDONTWRITEBYTECODE=1
# Turns off buffering for easier container logging
ENV PYTHONUNBUFFERED=1

ARG DATABASE_URL
ARG SECRET_KEY
ARG EMAIL_HOST
ARG EMAIL_PORT
ARG EMAIL_HOST_USER
ARG EMAIL_HOST_PASSWORD
ARG EMAIL_API_KEY

ENV DATABASE_URL=$DATABASE_URL
ENV DJANGO_SECRET=$SECRET_KEY
ENV EMAIL_HOST=$EMAIL_HOST
ENV EMAIL_PORT=$EMAIL_PORT
ENV EMAIL_HOST_USER=$EMAIL_HOST_USER
ENV EMAIL_HOST_PASSWORD=$EMAIL_HOST_PASSWORD
ENV EMAIL_API_KEY=$EMAIL_API_KEY

# Install pip requirements
ADD requirements.txt .
# ssh
ENV SSH_PASSWD "root:Docker!"
#install dependencies
RUN echo "Acquire::Check-Valid-Until \"false\";\nAcquire::Check-Date \"false\";" | cat > /etc/apt/apt.conf.d/10no--check-valid-until
RUN apt-get update && apt-get install -y python-pip python-dev && apt-get clean \
	&& apt-get install -y --no-install-recommends openssh-server \
    && apt-get clean \
	&& echo "$SSH_PASSWD" | chpasswd 

#copy file to code directory
WORKDIR /app
ADD . /app
RUN python -m pip install -r requirements.txt

COPY sshd_config /etc/ssh/
COPY init.sh /usr/local/bin/

RUN chmod u+x /usr/local/bin/init.sh
# Switching to a non-root user, please refer to https://aka.ms/vscode-docker-python-user-rights
RUN useradd appuser && chown -R appuser /app
USER appuser

RUN python manage.py makemigrations && python manage.py migrate
RUN python manage.py collectstatic --noinput
RUN python manage.py compress --force

EXPOSE 8000
EXPOSE 2222
# During debugging, this entry point will be overridden. For more information, please refer to https://aka.ms/vscode-docker-python-debug
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myApp.wsgi:application"]

```
