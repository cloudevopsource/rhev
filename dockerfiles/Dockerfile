FROM registry.access.redhat.com/rhel7/rhel
RUN subscription-manager register --username orasun@vip.qq.com --password Fastrise@666 --auto-attach
RUN subscription-manager list
RUN subscription-manager repos \
    --disable='*' \
    --enable=rhel-7-server-rpms \
    --enable=rhel-7-server-supplementary-rpms \
    --enable=rhel-7-server-rhv-4.3-manager-rpms \
    --enable=rhel-7-server-rhv-4-manager-tools-rpms \
    --enable=rhel-7-server-ansible-2-rpms \
    --enable=jb-eap-7.2-for-rhel-7-server-rpms
RUN yum install --downloadonly --downloaddir=/root/down \
    rhvm && \
    cd /root/down && rm -f systemd* && \
    tar zcvf /tmp/rhvm.tar.gz *.rpm && \
    rm -rf *
