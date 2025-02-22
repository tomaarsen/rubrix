FROM python:3.10.12-slim AS builder

# Copying argilla distribution files
COPY dist/*.whl /packages/
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN apt-get update && \
  apt-get install -y python-dev-is-python3 libpq-dev gcc && \
  pip install uvicorn[standard] && \
  for wheel in /packages/*.whl; do pip install "$wheel"[server,postgresql]; done && \
  apt-get remove -y python-dev-is-python3 libpq-dev gcc && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /packages

FROM python:3.10.12-slim

# Environment Variables
ENV ARGILLA_HOME_PATH=/var/lib/argilla
ENV DEFAULT_USER_ENABLED=true
ENV DEFAULT_USER_PASSWORD=1234
ENV DEFAULT_USER_API_KEY=argilla.apikey
ENV USERS_DB=/config/.users.yml
ENV UVICORN_PORT=6900

RUN useradd -ms /bin/bash argilla

# Create argilla volume
RUN mkdir -p "$ARGILLA_HOME_PATH" && \
  chown argilla:argilla "$ARGILLA_HOME_PATH"
VOLUME $ARGILLA_HOME_PATH

COPY scripts/start_argilla_server.sh /home/argilla
# Destination folder must be the same as the builder one. Otherwise installed script won't work (since the installation fixes the path inside the script)
COPY --chown=argilla:argilla --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

WORKDIR /home/argilla
RUN chmod +x start_argilla_server.sh

USER argilla

# Exposing ports
EXPOSE 6900

CMD ["/bin/bash", "start_argilla_server.sh"]
