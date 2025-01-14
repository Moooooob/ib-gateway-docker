
#
# Setup Stage: install apps
#
# This is a dedicated stage so that donwload archives don't end up on 
# production image and consume unnecessary space.
#

FROM ubuntu:22.04 as setup

ENV IB_GATEWAY_VERSION=10.12.2u
ENV IBC_VERSION=3.14.0

# Prepare system
RUN apt-get update -y
RUN apt-get install --no-install-recommends --yes \
  curl \
  ca-certificates \
  unzip

WORKDIR /tmp/setup

# Install IB Gateway
# Use this instead of "RUN curl .." to install a local file:
#COPY ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh .
RUN curl -sSL https://storage.googleapis.com/ib-dependencies/ibgateway-10.12.2u-standalone-linux-x64.sh \
  --output ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh
# RUN curl -sSL https://github.com/UnusualAlpha/ib-gateway-docker/raw/gh-pages/ibgateway-releases/ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh.sha256 \
#   --output ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh.sha256
# RUN sha256sum --check ./ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh.sha256
RUN chmod a+x ./ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh
RUN ./ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh -q -dir /root/Jts/ibgateway/${IB_GATEWAY_VERSION}
COPY ./stable/config/ibgateway/jts.ini /root/Jts/jts.ini

# Install IBC
RUN curl -sSL https://storage.googleapis.com/ib-dependencies/IBCLinux-3.14.0.zip --output IBCLinux-${IBC_VERSION}.zip
RUN mkdir /root/ibc
RUN unzip ./IBCLinux-${IBC_VERSION}.zip -d /root/ibc
RUN chmod -R u+x /root/ibc/*.sh 
RUN chmod -R u+x /root/ibc/scripts/*.sh
COPY ./stable/config/ibc/config.ini.tmpl /root/ibc/config.ini.tmpl

# Copy scripts
COPY ./stable/scripts /root/scripts

#
# Build Stage: build production image
#

FROM ubuntu:22.04

ENV IB_GATEWAY_VERSION=10.12.2u

WORKDIR /root

# Prepare system
RUN apt-get update -y
RUN apt-get install --no-install-recommends --yes \
  gettext \
  xvfb \
  libxslt-dev \
  libxrender1 \
  libxtst6 \
  libxi6 \
  libgtk2.0-bin \
  socat \
  x11vnc

# Copy files
COPY --from=setup /root/ .
RUN chmod a+x /root/scripts/*.sh
COPY --from=setup /usr/local/i4j_jres/ /usr/local/i4j_jres

# IBC env vars
ENV TWS_MAJOR_VRSN ${IB_GATEWAY_VERSION}
ENV TWS_PATH /root/Jts
ENV IBC_PATH /root/ibc
ENV IBC_INI /root/ibc/config.ini
ENV TWOFA_TIMEOUT_ACTION exit

# Start run script
CMD ["/root/scripts/run.sh"]