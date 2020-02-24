FROM olbat/debian:testing
MAINTAINER devel@olbat.net

#RUN add-apt-repository ppa:ubuntu-lxc/lxd-stable
# Install Packages (basic tools, cups, basic drivers, HP drivers)
RUN apt-get update -qq -y \
&& apt-get install -qq -y \
  sudo \
  whois \
  cups \
  cups-client \
  cups-bsd \
  cups-filters \
  foomatic-db-compressed-ppds \
  printer-driver-all \
  openprinting-ppds \
  hpijs-ppds \
  hp-ppd \
  hplip \
  python-cups \
  inotify-tools \
  libcups2 \
  libavahi-client3 \
  avahi-daemon \
  avahi-utils \
  libsnmp30 \
  build-essential \
  libcups2-dev \
  libavahi-client-dev \
  git \
  bzr \
&& apt-get -qq -y autoclean \
&& apt-get -qq -y autoremove \
&& apt-get -qq -y clean \
&& rm -rf /var/lib/apt/lists/*

ADD * /tmp/
RUN chmod +x /tmp/*.sh \
&& /tmp/install.sh 

# Create var/run/dbus, Disbale some cups backend that are unusable within a container, Clean install files
RUN mkdir -p /var/run/dbus \
&& mv -f /usr/lib/cups/backend/parallel /usr/lib/cups/backend-available/ || true \
&& mv -f /usr/lib/cups/backend/serial /usr/lib/cups/backend-available/ || true \
&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* || true

RUN systemctl enable avahi-daemon \
&& systemctl start avahi-daemon \
&& systemctl enable cupsd \
&& systemctl start cupsd
#########################################
##         EXPORTS AND VOLUMES         ##
#########################################
# Export volumes
VOLUME /config /etc/cups/ /var/log/cups /var/spool/cups /var/cache/cups /var/run/dbus
EXPOSE 631

# Add user and disable sudo password checking
RUN useradd \
  --system \
  --groups=sudo,lp,lpadmin,root \
  --no-create-home \
  --home-dir=/home/print \
  --shell=/bin/bash \
  --password=$(mkpasswd $CUPS_USER_PASSWORD) \
  $CUPS_USER_ADMIN \
&& sed -i '/%sudo[[:space:]]/ s/ALL[[:space:]]*$/NOPASSWD:ALL/' /etc/sudoers

# Configure the service's to be reachable
RUN /usr/sbin/cupsd \
  && while [ ! -f /var/run/cups/cupsd.pid ]; do sleep 1; done \
  && cupsctl --remote-admin --remote-any --share-printers \
  && kill $(cat /var/run/cups/cupsd.pid)

# Default shell
CMD ["/usr/sbin/cupsd", "-f"]
