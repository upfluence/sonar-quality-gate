FROM florianupf/sonar-quality-gate:0.0.11

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

WORKDIR /srv
ENTRYPOINT ["/entrypoint.sh"]
