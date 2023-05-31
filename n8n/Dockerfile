ARG N8N_VERSION=$N8N_VERSION
FROM docker.n8n.io/n8nio/n8n:$N8N_VERSION

RUN apk add --no-cache --update python3 py3-pip

RUN pip install fire -y

# Install n8n-nodes-python module
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-python