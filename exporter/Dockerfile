FROM gcr.io/google-appengine/debian9

# Get the list of packages from the base image.

ENV EXPORTER_VERSION 1.1.0
ENV EXPORTER_SHA1 7261fba6fcb927b85de6fdc2576813b5a97f976f

# Install Dependencies
RUN set -x && \
    apt-get update -y && \
    apt-get install -y --no-install-recommends \
    curl

# Download and Extract binaries
RUN curl -L -o redis-exporter.tar.gz https://github.com/oliver006/redis_exporter/releases/download/v${EXPORTER_VERSION}/redis_exporter-v${EXPORTER_VERSION}.linux-amd64.tar.gz && \
    echo "${EXPORTER_SHA1} *redis-exporter.tar.gz" | sha1sum -c - && \
    tar -xvf redis-exporter.tar.gz -C / --strip-components=1 && \
    chmod +x /redis_exporter


ENTRYPOINT [ "/redis_exporter" ]