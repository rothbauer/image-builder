# SPDX-FileCopyrightText: © 2025 VEXXHOST, Inc.
# SPDX-License-Identifier: GPL-3.0-or-later
# Atmosphere-Rebuild-Time: 2024-06-25T22:49:25Z

FROM openstack-venv-builder AS build
# renovate: name=openstack/horizon repo=https://github.com/openstack/horizon.git branch=stable/2025.2
ARG HORIZON_GIT_REF=dc63b9fb234f528acc8be7bde27c5e54c01cfb4a
ADD --keep-git-dir=true https://github.com/openstack/horizon.git#${HORIZON_GIT_REF} /src/horizon
RUN git -C /src/horizon fetch --unshallow
# renovate: name=openstack/designate-dashboard repo=https://github.com/openstack/designate-dashboard.git branch=stable/2025.2
ARG DESIGNATE_DASHBOARD_GIT_REF=9f1e7e9855dc58aa8d3a2ef24eea758a93963b90
ADD --keep-git-dir=true https://github.com/openstack/designate-dashboard.git#${DESIGNATE_DASHBOARD_GIT_REF} /src/designate-dashboard
RUN git -C /src/designate-dashboard fetch --unshallow
# renovate: name=openstack/heat-dashboard repo=https://github.com/openstack/heat-dashboard.git branch=stable/2025.2
ARG HEAT_DASHBOARD_GIT_REF=7ee87d7504e97e0538697ba6d7f58884994db7ac
ADD --keep-git-dir=true https://github.com/openstack/heat-dashboard.git#${HEAT_DASHBOARD_GIT_REF} /src/heat-dashboard
RUN git -C /src/heat-dashboard fetch --unshallow
# renovate: name=openstack/ironic-ui repo=https://github.com/openstack/ironic-ui.git branch=stable/2025.2
ARG IRONIC_UI_REF=c537b9562addf2f61fcbb324959460d9fb5b885d
ADD --keep-git-dir=true https://github.com/openstack/ironic-ui.git#${IRONIC_UI_REF} /src/ironic-ui
RUN git -C /src/ironic-ui fetch --unshallow
# renovate: name=openstack/magnum-ui repo=https://github.com/openstack/magnum-ui.git branch=stable/2025.2
ARG MAGNUM_UI_REF=d0bc945bddcddfddb6402ec52a85422a5605760c
ADD --keep-git-dir=true https://github.com/openstack/magnum-ui.git#${MAGNUM_UI_REF} /src/magnum-ui
RUN git -C /src/magnum-ui fetch --unshallow
# renovate: name=openstack/manila-ui repo=https://github.com/openstack/manila-ui.git branch=stable/2025.2
ARG MANILA_UI_REF=a2563fc220962c982dba1986972f3ddf5d132175
ADD --keep-git-dir=true https://github.com/openstack/manila-ui.git#${MANILA_UI_REF} /src/manila-ui
RUN git -C /src/manila-ui fetch --unshallow
# renovate: name=openstack/neutron-vpnaas-dashboard repo=https://github.com/openstack/neutron-vpnaas-dashboard.git branch=stable/2025.2
ARG NEUTRON_VPNAAS_DASHBOARD_REF=9bf310507187b7e0d4be71a56e47ce1d103b7f23
ADD --keep-git-dir=true https://github.com/openstack/neutron-vpnaas-dashboard.git#${NEUTRON_VPNAAS_DASHBOARD_REF} /src/neutron-vpnaas-dashboard
RUN git -C /src/neutron-vpnaas-dashboard fetch --unshallow
# renovate: name=openstack/octavia-dashboard repo=https://github.com/openstack/octavia-dashboard.git branch=stable/2025.2
ARG OCTAVIA_DASHBOARD_REF=a67d1cd7b9217dfafd8456af8f62ad02ed72663e
ADD --keep-git-dir=true https://github.com/openstack/octavia-dashboard.git#${OCTAVIA_DASHBOARD_REF} /src/octavia-dashboard
RUN git -C /src/octavia-dashboard fetch --unshallow
COPY patches/magnum-ui /patches/magnum-ui
RUN git -C /src/magnum-ui apply --verbose /patches/magnum-ui/*
RUN --mount=type=cache,target=/root/.cache/uv <<EOF bash -xe
uv pip install \
    --constraint /upper-constraints.txt \
        /src/designate-dashboard \
        /src/heat-dashboard \
        /src/horizon \
        /src/ironic-ui \
        /src/magnum-ui \
        /src/manila-ui \
        /src/neutron-vpnaas-dashboard \
        /src/octavia-dashboard \
        pymemcache
EOF

FROM openstack-python-runtime
RUN <<EOF bash -xe
apt-get update -qq
apt-get install -qq -y --no-install-recommends \
    apache2 gettext libapache2-mod-wsgi-py3
apt-get clean
rm -rf /var/lib/apt/lists/*
EOF
COPY --from=build --link /var/lib/openstack /var/lib/openstack
