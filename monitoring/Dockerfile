FROM kadalu/kadalu-base

RUN yum -y install python3-pyxattr glusterfs-server
RUN rpm -qa | grep gluster | tee /gluster-rpm-versions.txt

COPY lib/kadalulib.py                 /kadalu/
COPY server/server.py                 /kadalu/
COPY server/glusterfsd.py             /kadalu/
COPY server/kadalu_quotad/quotad.py   /kadalu/
COPY server/shd.py                    /kadalu/
COPY lib/startup.sh                   /kadalu/
COPY server/stop-server.sh            /kadalu/

COPY server/mount-glustervol          /usr/bin/

RUN mkdir -p /kadalu/templates /kadalu/volfiles

# Copy Volfile templates
COPY templates/Replica1.client.vol.j2 /kadalu/templates/
COPY templates/Replica3.client.vol.j2 /kadalu/templates/
COPY templates/Replica1.brick0.vol.j2 /kadalu/templates/
COPY templates/Replica3.brick0.vol.j2 /kadalu/templates/
COPY templates/Replica3.brick1.vol.j2 /kadalu/templates/
COPY templates/Replica3.brick2.vol.j2 /kadalu/templates/
COPY templates/Replica3.shd.vol.j2    /kadalu/templates/

RUN chmod +x /usr/bin/mount-glustervol
RUN chmod +x /kadalu/startup.sh
RUN chmod +x /kadalu/stop-server.sh

ARG version="(unknown)"
# Container build time (date -u '+%Y-%m-%dT%H:%M:%S.%NZ')
ARG builddate="(unknown)"

LABEL build-date="${builddate}"
LABEL io.k8s.description="KaDalu container(glusterfsd or glustershd)"
LABEL name="kadalu-server"
LABEL Summary="KaDalu Server"
LABEL vcs-type="git"
LABEL vcs-url="https://github.com/kadalu/kadalu"
LABEL vendor="org.kadalu.gluster"
LABEL version="${version}"

ENTRYPOINT ["/kadalu/startup.sh", "/usr/bin/python3", "/kadalu/server.py"]

# Debugging, Comment the above line and
# uncomment below line
# ENTRYPOINT ["tail", "-f", "/dev/null"]