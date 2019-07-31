FROM iagdrmd01acr01.azurecr.io/kafka-mirrormaker:latest

#VOLUME ["/var/lib/${COMPONENT}/data", "/etc/${COMPONENT}/secrets"]

COPY include/etc/ia/docker /etc/ia/docker

CMD ["/etc/ia/docker/run"]
