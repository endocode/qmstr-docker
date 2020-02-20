include versions.env

TAG := $(shell if [ ! -z "${BUILD_NUMBER}" ]; then echo ":${BUILD_NUMBER}";fi)

masterimage:
	DOCKER_BUILDKIT=1 docker build \
	--build-arg SCANCODE_VERSION=${SCANCODE_VERSION} \
	--build-arg GRPCIO_VERSION=${GRPCIO_VERSION} \
	--build-arg QMSTR_VERSION=${QMSTR_VERSION} \
	--build-arg QMSTR_BRANCH="master" \
	--build-arg QMSTR_FORK="endocode" \
	-t qmstr/master$(TAG) .
