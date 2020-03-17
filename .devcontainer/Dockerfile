# docker run --name common-container -v /Share_Root:/share -v /docker-share:/root/docker-share -p 22222:22 common-image:latest
FROM ubuntu:16.04
COPY ./docker-cache/apt-source/ali.sources.list /etc/apt/
RUN cp /etc/apt/sources.list /etc/apt/bak.sources.list \
    && rm /etc/apt/apt.conf.d/docker-clean

# SSH Server
# 替换服务器的rsa公钥并修改配置文件, 私钥的权限不对会导致SSHD起不来
RUN	cat /etc/apt/ali.sources.list > /etc/apt/sources.list && apt-get update \
    && apt-get install -y openssh-server
COPY ./docker-cache/.ssh /root/.ssh
RUN mkdir /var/run/sshd \
    && mv /root/.ssh/id_rsa_dockerServer /etc/ssh/ssh_host_rsa_key \
    && chmod u=rw,g=,o= /etc/ssh/ssh_host_rsa_key \
    && mv /root/.ssh/id_rsa_dockerServer.pub /etc/ssh/ssh_host_rsa_key.pub \
    && chmod u=rw,g=r,o=r /etc/ssh/ssh_host_rsa_key.pub \
    && sed -i 's@^PermitRootLogin\>.*$@PermitRootLogin yes@ \
    ; s@^#PasswordAuthentication\>.*$@PasswordAuthentication yes@ \
    ; s@^#AuthorizedKeysFile\>.*$@AuthorizedKeysFile /root/.ssh/authorized_keys@' /etc/ssh/sshd_config

# Install CMake, build-essential(g++, gcc, make, ...), gdb, git
COPY ./docker-cache/cmake-3.15.5-Linux-x86_64.sh /var/docker-cache/
RUN chmod +x /var/docker-cache/cmake-3.15.5-Linux-x86_64.sh \
    && /var/docker-cache/cmake-3.15.5-Linux-x86_64.sh --skip-license --prefix=/usr
RUN cat /etc/apt/ali.sources.list > /etc/apt/sources.list && apt-get update \
	&& apt-get install -y build-essential gdb git

# ENV
RUN printf "\n\
    export myname=common\n\
    export DISPLAY=192.168.31.31:0.0\n\
    alias cls='clear'\n\
    alias sshrestart='/etc/init.d/ssh restart'\n\
    PS1=\"\\[\\e[7;33m\\]\\w\\[\\e[0m\\]🤕\\$ \"\n\
    " >> /root/.bashrc

# All is OK, do some clean last
WORKDIR /root/
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

CMD [ "/usr/sbin/sshd", "-D" ]