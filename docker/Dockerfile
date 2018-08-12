FROM elixir:latest

ARG BLOCKSCOUT_VERSION=master
ENV DEBIAN_FRONTEND=noninteractive
ENV MIX_ENV=production

EXPOSE 4000

# Update Pkg-list and get build-reqs
RUN    apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get update \
    && apt-get install -yqq netcat libtool automake supervisor locales jq \
    && curl -sL https://deb.nodesource.com/setup_10.x | bash - \ 
    && apt install -yqq nodejs

# Create Application User 
RUN    mkdir -p /opt/app \
    && groupadd -r poausr \
    && useradd -r -m -d /home/poausr -g poausr poausr \
    && chown -R poausr: /opt/app

# Set working dir and App user
WORKDIR /opt/app
USER poausr

# Checkout defined version of blockscout to app dir
# and switch some hardcoded user info
RUN   git clone https://github.com/poanetwork/blockscout.git /opt/app \
   && git checkout ${BLOCKSCOUT_VERSION} \
   && grep -Ri ec2-user * | cut -f1 -d: | xargs -i{} sed -i 's/ec2-user/poausr/g' {}

# Install Node.js and Elixir dependencies
RUN   cd apps/explorer_web/assets && npm install; cd - \
   && cd apps/explorer && npm install; cd - 

# Build explorer
# Note: Alternatively if theres a build artifact from CI, we pull it from 
#       artifact management instead of building in Dockerfile.
RUN   /opt/app/bin/deployment/build 

# Switch back to root for system config tasks
USER root

# Set the locale
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    locale-gen
ENV LANG en_US.UTF-8  
ENV LANGUAGE en_US:en  
ENV LC_ALL en_US.UTF-8 

# Copy in Process Management Config
COPY supervisord.conf /etc/supervisor/conf.d/
COPY entrypoint.sh /opt/app/entrypoint.sh

# Set CMD Process
CMD ["/usr/bin/supervisord"]
