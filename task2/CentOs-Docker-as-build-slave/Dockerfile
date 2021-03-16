FROM centos/systemd

RUN yum -y install httpd; yum clean all; systemctl enable httpd.service
RUN curl -fsSL https://get.docker.com/ | sh

EXPOSE 80

CMD ["/usr/sbin/init"]