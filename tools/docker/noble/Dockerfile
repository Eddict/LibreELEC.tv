FROM ubuntu:noble

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
 && apt-get dist-upgrade -y \
 && apt-get install -y locales sudo

RUN locale-gen en_US.UTF-8 \
 && update-locale LANG=en_US.UTF-8 LANGUAGE=en_US:en
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=en_US.UTF-8

ARG USER_UID
ENV USER_UID=${USER_UID:-1234}
# make sure any user/group in image do not use the UUID/GUID of the host docker user/group
RUN <<EOF
# https://www.kevinguay.com/posts/next-uid-gid/
newid=$(cat /etc/group /etc/passwd | cut -d ":" -f 3 | grep "^1...$" | sort -n | tail -n 1 | awk '{ print $1+1 }')
echo ${newid}
# do we have a group with same GID
if groupname=$(getent group ${USER_UID}); then
  echo ${groupname}
  groupmod --gid ${newid} ${groupname}
fi
# do we have a user with same UID
if username=$(getent passwd ${USER_UID}); then
  echo ${username}
  usermod --uid ${newid} ${username}
fi
#groupadd --gid ${USER_UID} docker
#useradd docker --uid ${USER_UID} --no-user-group --groups docker,sudo --create-home --shell /bin/bash
useradd docker --uid ${USER_UID} --user-group --groups sudo --create-home --shell /bin/bash
echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
EOF

RUN apt-get update

RUN apt-get install -y \
    curl bash bc gcc-14 cpp-14 sed patch patchutils tar bzip2 gzip xz-utils zstd perl gawk gperf zip \
      unzip diffutils lzop make file g++-14 xfonts-utils xsltproc default-jre-headless python3 \
      libc6-dev libncurses5-dev libjson-perl libxml-parser-perl libparse-yapp-perl rdfind \
      golang-1.23-go git openssh-client rsync upx-ucl \
    --no-install-recommends \
    && ln -s /usr/lib/go-1.23 /usr/lib/go \
    && ln -s /usr/lib/go-1.23/bin/go /usr/bin/go \
    && ln -s /usr/lib/go-1.23/bin/gofmt /usr/bin/gofmt

RUN if [ "$(uname -m)" = "aarch64" ]; then \
  apt-get install -y libc6-amd64-cross qemu-user-binfmt --no-install-recommends; \
 fi

RUN rm -rf /var/lib/apt/lists/*

RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-14 100 \
    --slave /usr/bin/cpp cpp /usr/bin/cpp-14 \
    --slave /usr/bin/g++ g++ /usr/bin/g++-14 \
    --slave /usr/bin/gcov gcov /usr/bin/gcov-14
RUN update-alternatives --config gcc

USER docker
