# /usr/local/bin/start.sh will start the service

FROM openshifttools/oso-centos7-ops-base:latest

# Pause indefinitely if asked to do so.
ARG OO_PAUSE_ON_BUILD
RUN test "$OO_PAUSE_ON_BUILD" = "true" && while sleep 10; do true; done || :

# Add root folder
ADD root/ /root/

ADD scripts/ /usr/local/bin/

# Local install of python2-clamd, which is a prerequisite for scanpod-inmem
# Then install golang and the in-memory scanner
RUN yum install -y python2-boto3 \
		   /root/python2-clamd-1.0.2-4.el7.noarch.rpm \
                   golang \
                   git \
                   python-requests \
                   openscap-scanner \
                   openshift-tools-scripts-scanpod \
                   python2-botocore && \
    yum clean all

ADD clam-scanner /go/src/github.com/openshift/clam-scanner

ADD image-inspector /go/src/github.com/openshift/image-inspector

ENV GOBIN=/bin \
    CGO_ENABLED=0 \
    GOPATH=/go

# Make mount point for host filesystem and compile scanning utilities
RUN go get github.com/golang/glog github.com/spf13/cobra github.com/spf13/pflag && \
    mkdir -p /host /go/src/golang.org/x/net /run/clamd.scan /tmp/scan-data /var/lib/image-inspector && \
    cd /go/src/golang.org/x && \
    git clone https://github.com/golang/net.git --depth 1 && \
    go install -a -installsuffix cgo /go/src/github.com/openshift/image-inspector/cmd/image-inspector.go && \
    cd  /go/src/github.com/openshift/clam-scanner && \
    go install && \
    cd && \
    rm -rf /go

EXPOSE 8080

# run as root user
USER 0

# Start processes
CMD /usr/local/bin/start.sh
