FROM haproxy:lts-alpine
COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg
USER root
RUN mkdir /var/run/haproxy
RUN chown -Rf haproxy:haproxy /var/run/haproxy
USER haproxy