# Copyright 2015 The Kubernetes Authors. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

ARG BASE_IMAGE

FROM ${BASE_IMAGE}

ARG VERSION

LABEL org.opencontainers.image.title="NGINX Ingress Controller for Kubernetes"
LABEL org.opencontainers.image.documentation="https://kubernetes.github.io/ingress-nginx/"
LABEL org.opencontainers.image.source="https://github.com/kubernetes/ingress-nginx"
LABEL org.opencontainers.image.vendor="The Kubernetes Authors"
LABEL org.opencontainers.image.licenses="Apache-2.0"
LABEL org.opencontainers.image.version="${VERSION}"

WORKDIR  /etc/nginx

RUN apk add -U --no-cache \
  diffutils \
  libcap

COPY --chown=www-data:www-data . /

# Fix permission during the build to avoid issues at runtime
# with volumes (custom templates)
RUN bash -xeu -c ' \
  writeDirs=( \
    /etc/ingress-controller \
    /etc/ingress-controller/ssl \
    /etc/ingress-controller/auth \
    /var/log \
    /var/log/nginx \
  ); \
  for dir in "${writeDirs[@]}"; do \
    mkdir -p ${dir}; \
    chown -R www-data.www-data ${dir}; \
  done'

RUN  setcap    cap_net_bind_service=+ep /nginx-ingress-controller \
  && setcap -v cap_net_bind_service=+ep /nginx-ingress-controller

RUN  setcap    cap_net_bind_service=+ep /usr/local/nginx/sbin/nginx \
  && setcap -v cap_net_bind_service=+ep /usr/local/nginx/sbin/nginx

USER www-data

# Create symlinks to redirect nginx logs to stdout and stderr docker log collector
RUN  ln -sf /dev/stdout /var/log/nginx/access.log \
  && ln -sf /dev/stderr /var/log/nginx/error.log

ENTRYPOINT ["/usr/bin/dumb-init", "--"]

CMD ["/nginx-ingress-controller"]
