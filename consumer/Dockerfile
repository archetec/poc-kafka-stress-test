FROM confluentinc/cp-kafka:5.2.2

#VOLUME ["/var/lib/${COMPONENT}/data", "/etc/${COMPONENT}/secrets"]

COPY include/etc/ia/docker /etc/ia/docker

CMD ["/etc/ia/docker/run"]
