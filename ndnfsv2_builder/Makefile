PLUGIN_NAME=nexenta/nexentaedge-nfs-plugin
PLUGIN_TAG=stable


all: clean docker rootfs create enable clean


clean:
	@echo "### docker rm -f ndnfs_tmp"
	@docker rm -f ndnfs_tmp || true
	@echo "### rm ./plugin"
	@rm -rf ./plugin
	@echo "### rm ./ndnfs"
	@rm -rf ./ndnfs

docker:
	@echo "### docker build: builder image"
	@docker build -q -t builder -f Dockerfile.dev .
	@echo "### extract ndnfs"
	@docker create --name ndnfs_tmp builder
	@docker start -i ndnfs_tmp
	@docker cp ndnfs_tmp:/go/bin/ndnfs .
	@docker rm -vf ndnfs_tmp
	@docker rmi builder
	@echo "### docker build: rootfs image with ndnfs"
	@docker build -q -t ${PLUGIN_NAME}:rootfs .

rootfs:
	@echo "### create rootfs directory in ./plugin/rootfs"
	@mkdir -p ./plugin/rootfs
	@docker create --name ndnfs_tmp ${PLUGIN_NAME}:rootfs
	@docker export ndnfs_tmp | tar -x -C ./plugin/rootfs
	@echo "### copy config.json to ./plugin/"
	@cp config.json ./plugin/
	@docker rm -vf ndnfs_tmp

create:
	@echo "### remove existing plugin ${PLUGIN_NAME}:${PLUGIN_TAG} if exists"
	@docker plugin rm -f ${PLUGIN_NAME}:${PLUGIN_TAG} || true
	@echo "### mkdir /var/lib/ndnfs"
	@mkdir -p /var/lib/ndnfs
	@echo "### create new plugin ${PLUGIN_NAME}:${PLUGIN_TAG} from ./plugin"
	@docker plugin create ${PLUGIN_NAME}:${PLUGIN_TAG} ./plugin

enable:
	@echo "### enable plugin ${PLUGIN_NAME}:${PLUGIN_TAG}"
	@docker plugin enable ${PLUGIN_NAME}:${PLUGIN_TAG}

push:  clean docker rootfs create enable
	@echo "### push plugin ${PLUGIN_NAME}:${PLUGIN_TAG}"
	@docker plugin push ${PLUGIN_NAME}:${PLUGIN_TAG}
