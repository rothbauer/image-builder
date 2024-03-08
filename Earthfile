VERSION --use-copy-link --try 0.8

build.collection:
  FROM registry.gitlab.com/pipeline-components/ansible-lint:latest
  COPY . /src
  RUN ansible-galaxy collection build /src
  SAVE ARTIFACT /code/*.tar.gz AS LOCAL dist/

go.build:
  FROM golang:1.21
  WORKDIR /src
  ARG GOOS=linux
  ARG GOARCH=amd64
  ARG VARIANT
  COPY --dir go.mod go.sum ./
  RUN go mod download

libvirt-tls-sidecar.build:
  FROM +go.build
  ARG GOOS=linux
  ARG GOARCH=amd64
  ARG VARIANT
  #COPY --dir cmd internal ./
  #RUN GOARM=${VARIANT#"v"} go build -o main cmd/libvirt-tls-sidecar/main.go
  SAVE ARTIFACT ./main

libvirt-tls-sidecar.platform-image:
  ARG TARGETPLATFORM
  ARG TARGETARCH
  ARG TARGETVARIANT
  FROM --platform=$TARGETPLATFORM ./images/base+image
  COPY \
    --platform=linux/amd64 \
    (+libvirt-tls-sidecar.build/main --GOARCH=$TARGETARCH --VARIANT=$TARGETVARIANT) /usr/bin/libvirt-tls-sidecar
  ENTRYPOINT ["/usr/bin/libvirt-tls-sidecar"]
  ARG REGISTRY=core.harbor.cloud.prz/openstack-helm
  SAVE IMAGE --push ${REGISTRY}/libvirt-tls-sidecar:latest

libvirt-tls-sidecar.image:
    BUILD --platform=linux/amd64 +libvirt-tls-sidecar.platform-image

build.wheels:
  FROM ./images/builder+image
  COPY pyproject.toml poetry.lock ./
  ARG --required only
  RUN poetry export --only=${only} -f requirements.txt --without-hashes > requirements.txt
  RUN pip wheel -r requirements.txt --wheel-dir=/wheels
  SAVE ARTIFACT requirements.txt
  SAVE ARTIFACT /wheels
  SAVE IMAGE --cache-hint

build.venv:
  ARG --required only
  FROM +build.wheels --only ${only}
  RUN python3 -m venv /venv
  ENV PATH=/venv/bin:$PATH
  RUN pip install -r requirements.txt
  SAVE IMAGE --cache-hint

build.venv.dev:
  FROM +build.venv --only main,dev
  SAVE ARTIFACT /venv

build.venv.runtime:
  FROM +build.venv --only main
  SAVE ARTIFACT /venv

image:
  ARG RELEASE=2023.1
  FROM ./images/cloud-archive-base+image --RELEASE ${RELEASE}
  ENV ANSIBLE_PIPELINING=True
  DO ./images+APT_INSTALL --PACKAGES "rsync openssh-client"
  COPY +build.venv.runtime/venv /venv
  ENV PATH=/venv/bin:$PATH
  COPY +build.collections/ /usr/share/ansible
  ARG tag=latest
  ARG REGISTRY=core.harbor.cloud.prz/openstack-helm
  SAVE IMAGE --push ${REGISTRY}:${tag}

images:
  ARG REGISTRY=core.harbor.cloud.prz/openstack-helm
  BUILD +libvirt-tls-sidecar.image --REGISTRY=${REGISTRY}
  BUILD ./images/barbican+image --REGISTRY=${REGISTRY}
  BUILD ./images/cinder+image --REGISTRY=${REGISTRY}
  BUILD ./images/cluster-api-provider-openstack+image --REGISTRY=${REGISTRY}
  BUILD ./images/designate+image --REGISTRY=${REGISTRY}
  BUILD ./images/glance+image --REGISTRY=${REGISTRY}
  BUILD ./images/heat+image --REGISTRY=${REGISTRY}
  BUILD ./images/horizon+image --REGISTRY=${REGISTRY}
  BUILD ./images/ironic+image --REGISTRY=${REGISTRY}
  BUILD ./images/keystone+image --REGISTRY=${REGISTRY}
  #BUILD ./images/kubernetes-entrypoint+image --REGISTRY=${REGISTRY}
  BUILD ./images/libvirtd+image --REGISTRY=${REGISTRY}
  BUILD ./images/magnum+image --REGISTRY=${REGISTRY}
  BUILD ./images/manila+image --REGISTRY=${REGISTRY}
  BUILD ./images/netoffload+image --REGISTRY=${REGISTRY}
  BUILD ./images/neutron+image --REGISTRY=${REGISTRY}
  BUILD ./images/nova-ssh+image --REGISTRY=${REGISTRY}
  BUILD ./images/nova+image --REGISTRY=${REGISTRY}
  BUILD ./images/octavia+image --REGISTRY=${REGISTRY}
  BUILD ./images/openvswitch+image --REGISTRY=${REGISTRY}
  BUILD ./images/ovn+images --REGISTRY=${REGISTRY}
  BUILD ./images/placement+image --REGISTRY=${REGISTRY}
  BUILD ./images/senlin+image --REGISTRY=${REGISTRY}
  BUILD ./images/staffeln+image --REGISTRY=${REGISTRY}
  BUILD ./images/tempest+image --REGISTRY=${REGISTRY}

