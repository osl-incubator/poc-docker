# ref: https://github.com/mamba-org/micromamba-docker/blob/main/Dockerfile

FROM condaforge/mambaforge:latest

LABEL maintainer="Ivan Ogasawara <ivan.ogasawara@gmail.com>"
LABEL org.opencontainers.image.title="OpenScienceLabs"
LABEL org.opencontainers.image.authors="OpenScienceLabs Team"
LABEL org.opencontainers.image.source="https://github.com/osl-incubator/poc-docker"
LABEL org.opencontainers.image.version="latest"
LABEL org.opencontainers.image.description="OpenScienceLabs"
LABEL org.thegraphnetwork.osl.version="latest"

# it is the default, but using it here to have it explicitly
USER root

SHELL ["/bin/bash", "-c"]
# Use bash in Dockerfile RUN commands and make sure bashrc is sourced when
# executing commands with /bin/bash -c
# Needed to have the micromamba activate command configured etc.

ENV ENV_NAME=pocdocker
ENV DEBIAN_FRONTEND=noninteractive
ARG UID=1000
ARG GID=1000

RUN apt-get update -y \
  && apt-get install -y \
    build-essential \
    curl \
    tini \
  && rm -rf /var/lib/apt/lists/* \
    /var/cache/apt/archives \
    /tmp/*

RUN addgroup --gid ${GID} osl \
  && useradd --uid ${UID} --gid ${GID} -ms /bin/bash osl \
  && mkdir -p /opt/services \
  && chmod -R a+rwx /opt/conda /opt/services \
  && export ENV_NAME="$ENV_NAME" \
  && chown osl:osl /opt/services


USER osl

WORKDIR /opt/services/pocdocker

ENV PATH /opt/conda/envs/$ENV_NAME/bin:$PATH
ENV PYTHONPATH='/opt/services/pocdocker'

COPY --chown=osl:osl ./conda/ /tmp/conda

RUN mamba env create -n $ENV_NAME --file /tmp/conda/dev.yaml \
  && conda clean --all \
  && find /opt/conda/ -type f,l -name '*.a' -delete \
  && find /opt/conda/ -type f,l -name '*.pyc' -delete \
  && find /opt/conda/ -type f,l -name '*.js.map' -delete \
  && rm -rf /opt/conda/pkgs /tmp/*

COPY --chown=osl:osl ./ /opt/services/pocdocker/

COPY --chown=osl:osl containers/entrypoint.sh /opt/entrypoint.sh

RUN chmod +x /opt/entrypoint.sh \
  && echo "source /opt/entrypoint.sh" > ~/.bashrc

ENTRYPOINT ["tini", "--", "/opt/entrypoint.sh"]
CMD ["python -m http.server"]
