ARG QMSTR_BRANCH="master"
ARG QMSTR_FORK="endocode"

# VERSIONS
ARG QMSTR_VERSION='0.5'
ARG SCANCODE_VERSION='2.2.1'
ARG GRPCIO_VERSION='1.20.0'

################################################################################
# STAGE 1: Build master binaries                                               #
################################################################################

FROM golang:1.12-buster as gobuilder

ARG QMSTR_BRANCH
ARG QMSTR_FORK
ENV GOPROXY="https://proxy.golang.org"

RUN set -e && \
  apt update && \
  apt install -y git protobuf-compiler && \
  cd /root/ && \
  git clone https://github.com/${QMSTR_FORK}/qmstr.git \
    --branch ${QMSTR_BRANCH} \
    --single-branch

WORKDIR /root/qmstr

RUN set -e && \
  make qmstr-master && \
  make scancode-analyzer && \
  make spdx-identifier-analyzer && \
  make test-analyzer && \
  make qmstr-reporter-html && \
  make console-reporter && \
  make test-reporter && \
  make package-manifest-reporter

################################################################################
# STAGE 2: Fetch & install python components                                   #
################################################################################

FROM python:3.6-buster as pybuilder

ARG QMSTR_BRANCH
ARG QMSTR_FORK

RUN set -e && \
  apt install -y git && \
  cd /root/ && \
  git clone https://github.com/${QMSTR_FORK}/qmstr.git \
    --branch ${QMSTR_BRANCH} \
    --single-branch

WORKDIR /root/qmstr

# Generate python files from protobuf specification

ARG GRPCIO_VERSION
RUN set -e && \
  pip install grpcio==${GRPCIO_VERSION} \
  grpcio-tools==${GRPCIO_VERSION} && \
  python -m grpc_tools.protoc -Iproto \
  --python_out=./lib/pyqmstr/qmstr/service \
  --grpc_python_out=./lib/pyqmstr/qmstr/service proto/*.proto && \
  sed -i -E 's/(^import.*_pb2)/from . \1/' ./lib/pyqmstr/qmstr/service/*.py

# Build pyqmstr wheels

ARG QMSTR_VERSION
RUN set -e && \
  GRPCIO_VERSION=${GRPCIO_VERSION} QMSTR_VERSION=${QMSTR_VERSION} pip wheel \
  -w ./wheels lib/pyqmstr

# Build pex file for spdx-analyzer

RUN pip install pex
RUN set -e && \
  QMSTR_VERSION=${QMSTR_VERSION} pex ./modules/analyzers/spdx-analyzer/ \
  pyqmstr -e spdxanalyzer.__main__:main --disable-cache -f ./wheels -o \
  ./out/analyzers/spdx-analyzer


################################################################################
# STAGE 3: Build master container image (deploy)                               #
################################################################################

FROM python:3.6-buster as deploy

# Required QMSTR directories
ENV QMSTR_DIRS="/var/qmstr/ /var/cache/qmstr/ /var/lib/qmstr/ \
/opt/scancode-toolkit/"

# Copy binaries from build stage
COPY --from=gobuilder /root/qmstr/out/* /usr/local/bin/
COPY --from=pybuilder /root/qmstr/out/analyzers/* /usr/local/bin/

# Get the scancode toolkit

ARG SCANCODE_VERSION
ENV SCANCODE_BASEURL=https://github.com/nexB/scancode-toolkit/releases/download/
ENV SCANCODE_RELEASE=scancode-toolkit-${SCANCODE_VERSION}
RUN set -e && \
  wget ${SCANCODE_BASEURL}v${SCANCODE_VERSION}/${SCANCODE_RELEASE}.zip && \
  unzip ${SCANCODE_RELEASE} -d /opt/ && \
  ln -sf -T /opt/${SCANCODE_RELEASE}/scancode /usr/local/bin/scancode && \
  addgroup qmstr && \
  adduser --system qmstr --ingroup qmstr && \
  mkdir -p ${QMSTR_DIRS} && \
  chown -R qmstr /opt/${SCANCODE_RELEASE} && \
  chown -R qmstr ${QMSTR_DIRS} && \
  DUMMY=$(mktemp -d) && \
  scancode -l -c --json $DUMMY/out.json $DUMMY && \
  rm -rf $DUMMY

WORKDIR /home/qmstr
USER qmstr

COPY --chown=qmstr entrypoint-test.sh .

EXPOSE 50051

VOLUME /home/qmstr/config
VOLUME /home/qmstr/buildroot

ENTRYPOINT ["./entrypoint-test.sh"]
#CMD qmstr-master --config ./config/qmstr.yaml
