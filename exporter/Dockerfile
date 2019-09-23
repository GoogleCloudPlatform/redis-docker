FROM gcr.io/google-appengine/debian9


# Get the list of packages from the base image.
ENV EXPORTER_VERSION 1.2.0
ENV EXPORTER_SHA1 3de72cd115b1f93ccff01458ba2806c177539f78

ENV NOTICES_SHA256 71e6cf75de2d43c4477212c0f9feddec0754e5708fbe1b506caffcda9801c8aa

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


# Copy licenses and check them
COPY LICENSE /usr/share/doc
COPY NOTICES /usr/share/doc
RUN set -eux \
    && echo "${NOTICES_SHA256}" /usr/share/doc/NOTICES | sha256sum -c

ENTRYPOINT [ "/redis_exporter" ]
