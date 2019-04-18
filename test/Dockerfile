FROM ubuntu:18.04
MAINTAINER pbrian
LABEL Name="pbriandev" Version=0.0.1

### Constants
ENV WKDIR /staging
ENV USERHOME /home/pbrian
RUN mkdir $WKDIR
## {{ py3 }}

###### Install with apt
RUN apt-get update && \
    apt-get install -y python3.7 \
                       python3.7-dev \
                       python3-distutils \
                       python3-distlib \
                       python3-pip


CMD ["/bin/bash"]
