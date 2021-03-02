build:
	GOARCH=mips64le make build
	make docker-build DAPR_REGISTRY=loongson DAPR_TAG=0.6.0 TARGET_ARCH=mips64le
