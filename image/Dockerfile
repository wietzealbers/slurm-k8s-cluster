FROM rockylinux:9

LABEL org.opencontainers.image.source="https://github.com/stackhpc/slurm-docker-cluster" \
      org.opencontainers.image.title="slurm-docker-cluster" \
      org.opencontainers.image.description="Slurm Docker cluster on Rocky Linux 9" \
      org.label-schema.docker.cmd="docker-compose up -d" \
      maintainer="StackHPC"

ARG SLURM_TAG=slurm-23.11
ARG GOSU_VERSION=1.11

COPY kubernetes.repo /etc/yum.repos.d/kubernetes.repo

RUN set -ex \
    && yum makecache \
    && yum -y update \
    && yum -y install dnf-plugins-core epel-release \
    && yum -y install dnf-plugins-core \
    && yum config-manager --set-enabled crb \
    && yum -y module enable ruby:3.1 nodejs:18 \
#    && yum -y install https://yum.osc.edu/ondemand/2.0/ondemand-release-web-2.0-1.noarch.rpm \
    && yum -y install https://yum.osc.edu/ondemand/3.1/ondemand-release-web-3.1-1.el9.noarch.rpm \
#    && yum -y install https://yum.osc.edu/ondemand/3.1/ondemand-release-compute-3.1-1.el9.noarch.rpm \
    && yum -y module install ruby nodejs \
    && yum -y install \
       wget \
       bzip2 \
       perl \
       gcc \
       gcc-c++\
       git \
       gnupg \
       make \
       munge \
       munge-devel \
       python3-devel \
       python3-pip \
       python3 \
       mariadb-server \
       mariadb-devel \
       psmisc \
       bash-completion \
       vim-enhanced \
       http-parser-devel \
       json-c-devel \
       mpitests-openmpi \
       pmix-devel \
       hwloc \
       hwloc-devel \
       dbus-devel \
       openssh-server \
       apptainer \
       ondemand \
       kubectl \
    && yum clean all \
    && rm -rf /var/cache/yum

##RUN alternatives --set python /usr/bin/python3

RUN pip3 install Cython nose

RUN set -ex \
    && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-amd64" \
    && wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-amd64.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
    && rm -rf "${GNUPGHOME}" /usr/local/bin/gosu.asc \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true

RUN set -x \
    && git clone -b ${SLURM_TAG} --single-branch --depth=1 https://github.com/SchedMD/slurm.git \
    && pushd slurm \
    && ./configure --enable-debug --prefix=/usr --sysconfdir=/etc/slurm \
        --with-mysql_config=/usr/bin  --libdir=/usr/lib64 \
    && make install \
    && install -D -m644 contribs/slurm_completion_help/slurm_completion.sh /etc/profile.d/slurm_completion.sh \
    && popd \
    && rm -rf slurm

RUN mkdir /etc/sysconfig/slurm \
        /var/spool/slurmd \
        /var/run/slurmd \
        /var/run/slurmdbd \
        /var/lib/slurmd \
        /var/log/slurm \
        /data \
        /etc/slurm \
    && touch /var/lib/slurmd/node_state \
        /var/lib/slurmd/front_end_state \
        /var/lib/slurmd/job_state \
        /var/lib/slurmd/resv_state \
        /var/lib/slurmd/trigger_state \
        /var/lib/slurmd/assoc_mgr_state \
        /var/lib/slurmd/assoc_usage \
        /var/lib/slurmd/qos_usage \
        /var/lib/slurmd/fed_mgr_state \
    && useradd -r --uid=990 slurm \
    && chown -R slurm:slurm /var/*/slurm* \
    && useradd -u 1000 rocky \
    && usermod -p '*' rocky # unlocks account but sets no password

VOLUME /etc/slurm
COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
