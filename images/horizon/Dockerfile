# SPDX-FileCopyrightText: © 2025 VEXXHOST, Inc.
# SPDX-License-Identifier: GPL-3.0-or-later
# Atmosphere-Rebuild-Time: 2024-06-25T22:49:25Z

FROM openstack-venv-builder AS build
# renovate: name=openstack/horizon repo=https://github.com/openstack/horizon.git branch=stable/2025.1
ARG HORIZON_GIT_REF=19488fb4ae5da3fd57096734f276da102f8aaff3
ADD --keep-git-dir=true https://github.com/openstack/horizon.git#${HORIZON_GIT_REF} /src/horizon
RUN git -C /src/horizon fetch --unshallow
# renovate: name=openstack/designate-dashboard repo=https://github.com/openstack/designate-dashboard.git branch=stable/2025.1
ARG DESIGNATE_DASHBOARD_GIT_REF=349f34baca5813aebde7097318cde160f60aa061
ADD --keep-git-dir=true https://github.com/openstack/designate-dashboard.git#${DESIGNATE_DASHBOARD_GIT_REF} /src/designate-dashboard
RUN git -C /src/designate-dashboard fetch --unshallow
# renovate: name=openstack/heat-dashboard repo=https://github.com/openstack/heat-dashboard.git branch=stable/2025.1
ARG HEAT_DASHBOARD_GIT_REF=04a8d01a0c02737c1990e7187db5f0f86e8cac93
ADD --keep-git-dir=true https://github.com/openstack/heat-dashboard.git#${HEAT_DASHBOARD_GIT_REF} /src/heat-dashboard
RUN git -C /src/heat-dashboard fetch --unshallow
# renovate: name=openstack/ironic-ui repo=https://github.com/openstack/ironic-ui.git branch=stable/2025.1
ARG IRONIC_UI_REF=8d03cdb78e800e74399744984bfccb7710557671
ADD --keep-git-dir=true https://github.com/openstack/ironic-ui.git#${IRONIC_UI_REF} /src/ironic-ui
RUN git -C /src/ironic-ui fetch --unshallow
# renovate: name=openstack/magnum-ui repo=https://github.com/openstack/magnum-ui.git branch=stable/2025.1
ARG MAGNUM_UI_REF=37e79129e069a609ed766294438b54f777196949
ADD --keep-git-dir=true https://github.com/openstack/magnum-ui.git#${MAGNUM_UI_REF} /src/magnum-ui
RUN git -C /src/magnum-ui fetch --unshallow
# renovate: name=openstack/manila-ui repo=https://github.com/openstack/manila-ui.git branch=stable/2025.1
ARG MANILA_UI_REF=f27cabee486adb30d86b8d7aa9c24171ab916b13
ADD --keep-git-dir=true https://github.com/openstack/manila-ui.git#${MANILA_UI_REF} /src/manila-ui
RUN git -C /src/manila-ui fetch --unshallow
# renovate: name=openstack/neutron-vpnaas-dashboard repo=https://github.com/openstack/neutron-vpnaas-dashboard.git branch=stable/2025.1
ARG NEUTRON_VPNAAS_DASHBOARD_REF=d77ceeb44c741aac4d42a2152e18487557736752
ADD --keep-git-dir=true https://github.com/openstack/neutron-vpnaas-dashboard.git#${NEUTRON_VPNAAS_DASHBOARD_REF} /src/neutron-vpnaas-dashboard
RUN git -C /src/neutron-vpnaas-dashboard fetch --unshallow
# renovate: name=openstack/octavia-dashboard repo=https://github.com/openstack/octavia-dashboard.git branch=stable/2025.1
ARG OCTAVIA_DASHBOARD_REF=d95b42dc10dbc32888c49ad43f39223a59f436bc
ADD --keep-git-dir=true https://github.com/openstack/octavia-dashboard.git#${OCTAVIA_DASHBOARD_REF} /src/octavia-dashboard
RUN git -C /src/octavia-dashboard fetch --unshallow
COPY patches/magnum-ui /patches/magnum-ui
RUN git -C /src/magnum-ui apply --verbose /patches/magnum-ui/*
# use workaround (xstatic needs an older version of setuptools)
#RUN uv pip install "setuptools<70"
#RUN uv pip show setuptools
#ENV UV_EXTRA_BUILD_DEPENDENCIES='{"xstatic-angular-schema-form": ["setuptools<82"]}'
RUN --mount=type=cache,target=/root/.cache/uv <<EOF bash -xe
uv pip install \
    --constraint /upper-constraints.txt \
    --extra-build-dependency "xstatic-angular-schema-form=setuptools<82" \
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
