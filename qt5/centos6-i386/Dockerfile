# CentOS6 i386 + Qt5
#
# VERSION               0.0.1
# vi: set ft=dockerfile :

# FROM centos:centos6
FROM toopher/centos-i386:centos6
MAINTAINER Dmitry Svintsov <root@uralbash.ru>

ENV HOME_USER vagrant
ENV HOME_PASS 123

WORKDIR /tmp

## Add USER
RUN useradd -m -s /bin/bash ${HOME_USER} -u 1000 && \
    echo "${HOME_USER}:${HOME_PASS}"|chpasswd && \
    groupadd sudo && \
    gpasswd -a ${HOME_USER} sudo && \
    echo ${HOME_USER}' ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers

## Install an SSH keys
RUN yum -y install openssh-server openssh-clients

RUN rm -f /etc/ssh/ssh_host_rsa_key
RUN rm -f /etc/ssh/ssh_host_dsa_key
RUN rm -f /etc/ssh/ssh_host_ecdsa_key
RUN ssh-keygen -q -N "" -t rsa -f /etc/ssh/ssh_host_rsa_key
RUN ssh-keygen -q -N "" -t dsa -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -q -N "" -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key

RUN mkdir -p /home/vagrant/.ssh
RUN chmod 0700 /home/vagrant/.ssh
RUN echo "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key" > /home/vagrant/.ssh/authorized_keys
RUN chmod 0600 /home/vagrant/.ssh/authorized_keys
RUN chown -R vagrant: /home/vagrant/.ssh
RUN mkdir -p /etc/sudoers.d
RUN install -b -m 0440 /dev/null /etc/sudoers.d/vagrant
RUN echo 'vagrant ALL=NOPASSWD: ALL' >> /etc/sudoers.d/vagrant
RUN sed -i 's/Defaults    requiretty/#Defaults    requiretty/g' /etc/sudoers

## Minimal requirements
RUN yum install bzip2 tar wget -yyy

## Downloads
RUN wget http://mirror.tochlab.net/pub/gnu/gcc/gcc-4.8.2/gcc-4.8.2.tar.gz && tar xvzf gcc-4.8.2.tar.gz && rm gcc-4.8.2.tar.gz
RUN wget http://download.qt.io/official_releases/qt/5.7/5.7.0/single/qt-everywhere-opensource-src-5.7.0.tar.gz

## C++11
RUN yum groupinstall "Development Tools" -yyy
RUN yum install glibc-devel.i686 glibc-devel ibstdc++-devel.i686 -yyy
RUN yum install gcc gcc.i686 -yyy
RUN yum install gcc gcc.i386 -yyy
RUN yum install glibc-devel.i386 glibc-devel ibstdc++-devel.i386 -yyy
RUN cd /tmp/gcc-4.8.2 && ./contrib/download_prerequisites
RUN mkdir -p /tmp/objdir
WORKDIR /tmp/objdir
RUN /tmp/gcc-4.8.2/configure --prefix=/opt/gcc-4.8.2 --build=i686-pc-linux-gnu "CFLAGS=-m32" "CXXFLAGS=-m32" "LDFLAGS=-m32"
RUN make -j4
RUN make install

RUN mv /usr/lib/libstdc++.so.6 /usr/lib/libstdc++.so.6.backup
RUN cp -P /opt/gcc-4.8.2/lib/libstdc++.so.6 /usr/lib
RUN cp /opt/gcc-4.8.2/lib/libstdc++.so.6.0.18 /usr/lib/

## Xorg
RUN yum -y install libX11-devel
RUN yum -y groupinstall "X Window System"

## libxcb
RUN yum -y install libxcb libxcb-devel xcb-util xcb-util-devel

## Fonts
RUN yum -y install freetype-devel
RUN yum -y install fontconfig-devel

## Qt5
WORKDIR /tmp
RUN tar xvzf qt-everywhere-opensource-src-5.7.0.tar.gz
WORKDIR /tmp/qt-everywhere-opensource-src-5.7.0

RUN PATH=/opt/gcc-4.8.2/bin/:$PATH \
    ./configure \
    --prefix=/opt/Qt-5.7/ \
    -static \
    -release \
    -opensource \
    -confirm-license \
    -nomake examples \
    -nomake tests \
    -no-compile-examples \
    -skip qtgamepad \
    -skip qtvirtualkeyboard \
    -skip declarative \
    -qt-xcb \
    -fontconfig

RUN PATH=/opt/gcc-4.8.2/bin/:$PATH \
    make -j4
RUN make install
RUN make clean

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]