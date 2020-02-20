include repo.env
include versions.env

TAG := $(shell if [ ! -z "${BUILD_NUMBER}" ]; then echo ":${BUILD_NUMBER}";fi)

masterimage:
	DOCKER_BUILDKIT=1 docker build \
	--build-arg QMSTR_VERSION="${QMSTR_VERSION}" \
	--build-arg SCANCODE_VERSION="${SCANCODE_VERSION}" \
	--build-arg GRPCIO_VERSION="${GRPCIO_VERSION}" \
	--build-arg QMSTR_BRANCH="${QMSTR_BRANCH}" \
	--build-arg QMSTR_FORK="${QMSTR_FORK}" \
	-t qmstr/master$(TAG) .
