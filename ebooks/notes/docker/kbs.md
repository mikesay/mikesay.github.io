## Dockerfile for Go
```docker
# 1.14-alpine
# pull from the exact digest for security purpose to make sure it is the exact image you want
#
FROM golang@sha256:e484434a085a28801e81089cc8bcec65bc990dd25a070e3dd6e04b19ceafaced AS builder

# Install git for fetching go modules
#
RUN apk update && \
    apk add --no-cache git tzdata

# Setup non root user with limit option
#
RUN adduser \
    --disabled-password \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid 80008 \
    mike

WORKDIR $GOPATH/src/main/hello/
COPY . .

RUN go mod download
RUN go mod verify

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-w -s" -o /go/bin/hello

##
## Build real docker image
##
FROM scratch
COPY --from=builder /go/bin/hello /go/bin/hello

# Copy zoneinfo
#
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo

# Copy running user
#
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /etc/group /etc/group

# User non root user with limited privelege
#
USER mike:mike

# Expose the service by port. Try to use port higher 
# than 1024, so that no need root priveledge to bind
#
EXPOSE 3000

# Export directory
#
VOLUME /tmp

# Set default environment value
#
ENV MESSAGES=Mike

ENTRYPOINT ["/go/bin/hello"]
```

## Dockerfile for Python
```docker
## Create python runtime
#
FROM python:3.8-slim-buster as builder
# Use PYPI_REPO to pass customized PyPI registry
# docker build --timeout 120 -t dap-cn-sddc:latest --build-arg PYPI_REPO=-i\ xxx
# i.e -i https://<username>:<password>@xxxx/artifactory/api/pypi/xxxx-pypi-virtual/simple
ARG PYPI_REPO

WORKDIR /opt/app

# Copy python code
COPY app.py /opt/app/
COPY requirements.txt /opt/app/

# Install dependencies
RUN pip install --timeout 120 ${PYPI_REPO} -r requirements.txt

## Build real docker image
##
FROM python:3.8-slim-buster

# Setup non root user with limit option
RUN adduser \
    --gecos \
    --disabled-password \
    --disabled-login \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid 80008 \
    apprunner

WORKDIR /opt/app

# Copy python code
COPY --from=builder /opt/app/ /opt/app/

# Copy dependencies
COPY --from=builder /usr/local/lib/python3.8/site-packages/ /usr/local/lib/python3.8/site-packages/

# Define volume for config
VOLUME /opt/app/config

# Change ownership of working directory
RUN chown -R apprunner:apprunner /opt/app

# User non root user with limited privelege
USER apprunner:appruner

ENTRYPOINT ["/usr/local/bin/python", "/opt/app/app.py"]
```

## Dockerfile for Angular
```docker
FROM node:10.20.1 AS builder
WORKDIR /opt/dashboard
COPY . .
RUN npm install && \
    ./node_modules/@angular/cli/bin/ng build --prod --base-href /dashboard/ --deploy-url /dashboard/

FROM nginx:1.15.9
RUN /bin/bash -c 'mkdir -p /opt/dashboard/www/dashboard'
WORKDIR /opt/dashboard
COPY --from=builder /opt/dashboard/build/dashboard /opt/dashboard/www/dashboard/
COPY docker/start.sh /opt/dashboard/
COPY docker/dashboard.conf /etc/nginx/conf.d/

CMD ["/bin/bash", "/opt/cw-dashboard/start.sh"]
```

**dashboard.conf**
```nginx
server {
    listen 8888;
    server_name _;

    root /opt/dashboard/www;
    index index.html index.htm index.nginx-debian.html;

    location /dashboard {
        try_files $uri $uri/ /dashboard/index.html =404;
    }
}
```

**start.sh**
```bash
onfig_file="/opt/dashboard/www/dashboard/assets/data/app-config.json"

if [ ! -z "${CONTEXT_NAME}" ] && [ "${CONTEXT_NAME}" != "dashboard" ]
then
	sed -i "s/\/dashboard\//\/${CONTEXT_NAME}\//g" /opt/dashboard/www/dashboard/index.html
	sed -i "s/\/dashboard/\/${CONTEXT_NAME}/g" /etc/nginx/conf.d/dashboard.conf
	mv /opt/dashboard/www/dashboard /opt/dashboard/www/${CONTEXT_NAME}
	config_file="/opt/dashboard/www/${CONTEXT_NAME}/assets/data/app-config.json"
fi

if [ ! -z "${BACKEND_HOST}" ]
then
	sed -ibak.1 -r "s/(.*ServicesBaseUrl.*?:.*http:\/\/).*(\/api[\'|\"].*$)/\1${BACKEND_HOST}\2/g" ${config_file}
fi

nginx -g 'daemon off;'
```