FROM gcr.io/google-appengine/debian9


# Get the list of packages from the base image.
ENV EXPORTER_VERSION 1.3.1
ENV EXPORTER_SHA1 d4459037d64b07a6ca7d316904e246b729930a69822301d0c7a018913c9c1fd1

ENV GOLANG_VERSION 1.13
ENV GOLANG_SHA256 68a2297eb099d1a76097905a2ce334e3155004ec08cdea85f24527be3c48e856

# Install Dependencies
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Download and Extract go binaries
RUN curl -o go.tar.gz https://dl.google.com/go/go${GOLANG_VERSION}.linux-amd64.tar.gz \
    && set -eux \
    && echo "${GOLANG_SHA256} go.tar.gz" | sha256sum -c \
    && tar -C /usr/local -zxvf go.tar.gz

# Download source and build exporter
WORKDIR /redis_exporter
RUN git clone https://github.com/oliver006/redis_exporter --branch=v${EXPORTER_VERSION} /redis_exporter \
    && export PATH=$PATH:/usr/local/go/bin \
    && go build \
    && chmod +x redis_exporter

ENTRYPOINT [ "/redis_exporter/redis_exporter" ]
