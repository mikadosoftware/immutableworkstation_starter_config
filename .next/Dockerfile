FROM ubuntu:18.04
MAINTAINER pbrian
LABEL Name="pbriandev" Version=0.0.1

### Constants
ENV WKDIR /staging
ENV USERHOME /home/pbrian
ENV USERNAME pbrian
ENV USERFULLNAME "Paul Brian"
RUN mkdir $WKDIR

#Investigate this more
ENV DEBIAN_FRONTEND=noninteractive


### {{ apt }}

# build the basic development world
RUN apt-get update && \
    apt-get install -y apt-transport-https \
                       build-essential \
                       build-essential \
                       dbus-x11 \
                       dos2unix \
                       emacs25 \
                       fonts-inconsolata \
                       git \
                       konsole \
                       libpython3.6-dev \
                       openssh-server \
                       python3.6 \
                       python3.6-dev \
                       software-properties-common \
                       wget curl \
                       x11-apps \
                       firefox \
		       xpdf zathura

#: snapd allows easy install of hub, but building one inside a docker seems
# like a nightmare https://ograblog.wordpress.com/2017/06/02/dock-a-snap/
# So I still want to use `hub`, so I need another approch
#
# https://github.com/github/hub/releases/download/v2.11.1/hub-linux-amd64-2.11.1.tgz
# Install binary and documentation
ARG hubpkg=https://github.com/github/hub/releases/download/v2.11.1/hub-linux-amd64-2.11.1.tgz
RUN wget ${hubpkg}
RUN tar zvxvf hub-linux-amd64-2.11.1.tgz
RUN ./hub-linux-amd64-2.11.1/install

# Setup autocomplete for bash:
#mkdir -p ~/.bash/completions
#mv ./hub-linux-amd64-2.2.9/etc/hub.bash_completion.sh ~/.bash/completions/_hub
#echo "if [ -f ~/.bash/completions/_hub ]; then" >> ~/.bashrc
#echo "    . ~/.bash/completions/_hub" >> ~/.bashrc
#echo "fi" >> ~/.bashrc

# add alias
#echo "eval "$(hub alias -s)"" >> ~/.bashrc

# Cleanup
#rm -rf hub-linux-amd64-2.11.1



### {{ baseconfig }}

#################### Misc config ##########################
## Locales
# We dont have any languages setup at this point
# 
RUN apt-get update && \
    apt-get install -y locales && \
    locale-gen en_GB.UTF-8 && \
    update-locale LC_ALL=en_GB.UTF-8 LANGUAGE=en_GB.UTF-8
#these need to be run in user sections
#RUN echo "export LC_ALL=en_GB.UTF-8" >> /home/pbrian/.bashrc
#RUN echo "export LC_LANGUAGE=en_GB.UTF-8" >> /home/pbrian/.bashrc

    
    

#THis should now work in our terminal (prints Greek for Thank you)
#python3 -c "print(u''.join([u'\u0395', u'\u03c5', u'\u03c7', u'\u03B1', u'\u03c1', u'\u03B9', u'\u03C3', u'\u03c4', u'\u03c9']))"

## fonts
RUN apt-get -y install fonts-inconsolata

ENV TZ=Europe/London
#not ideal
ENV DEBIAN_FRONTEND=noninteractive
RUN DEBIAN_FRONTEND=noninteractive && \
    apt-get update && \
    apt-get install -y tzdata
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone



### {{ otherdevtools }}


### Other Dev Tools
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash -
RUN apt-get install -y \
                        nodejs \
			whois
RUN npm install -g create-react-app
# this has to be done when instance is running ...
# RUN cd /var/projects/learnreact && create-react-app test1



### {{ sshX }}

### Create SSH access to box
### NB Fix root access 
RUN mkdir -p /var/run/sshd		       
RUN echo 'root:root' | chpasswd	
RUN sed -ri 's/^#PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config
RUN sed -ri 's/^#AllowTcpForwarding\s+.*/AllowTcpForwarding yes/g' /etc/ssh/sshd_config
RUN sed -ri 's/^#X11Forwarding\s+.*/X11Forwarding yes/g' /etc/ssh/sshd_config
RUN sed -ri 's/^#X11UseLocalhost\s+.*/X11UseLocalhost no/g' /etc/ssh/sshd_config





### {{ user }}


### Setup me as only user .... because I will be !
RUN apt-get install sudo
RUN useradd -m -c "$USERFULLNAME" $USERNAME --shell /bin/bash 
RUN usermod -aG sudo $USERNAME
RUN echo "$USERNAME:$USERNAME" | chpasswd

# change user locale settings (see baseconfig)
RUN echo "export LC_ALL=en_GB.UTF-8" >> $USERHOME/.bashrc
RUN echo "export LC_LANGUAGE=en_GB.UTF-8" >> $USERHOME/.bashrc

# This is really just the config settings 
# any secrfets are added via '/var/secrets' volume per run
# as such this is a "safe" image to keep on say a hub
COPY rcassets/.emacs $USERHOME
COPY rcassets/.pylintrc $USERHOME
COPY rcassets/.gitconfig $USERHOME
COPY rcassets/.ssh $USERHOME/.ssh
COPY rcassets/ENTRYPOINT.sh $USERHOME
COPY rcassets/ENTRYPOINT.sh $USERHOME
RUN chmod 0777 $USERHOME/ENTRYPOINT.sh




### {{ py3 }}

###### Install with apt
RUN apt-get install -y python3.7 \
                       python3.7-dev \
                       python3-distutils \
                       python3-distlib \
                       python3-pip \
		       python-matplotlib

###### symlinking to have `pip` and `python`
RUN cd /usr/bin \
       && ln -sf python3.7 python \
       && ln -sf python3.7 python3 \
       && ln -sf pip3 pip

#####
RUN python3 -m pip install --upgrade pip
RUN python3 -m pip install pygithub

### update python pkgs
# Install any needed packages (ie above those for the runtime)
RUN pip install --trusted-host pypi.python.org sphinx \
                                               pytest \
					       pylint


# install requirements file. (why not specify in thisfile???)
COPY rcassets/requirements.txt $WKDIR/
RUN pip install -r $WKDIR/requirements.txt
# Additonal setup for spacy.  I think this is sensible to do specify in this file them
RUN python -m spacy download en

# seems ubunut installs pip_internal again mucking things up

# lastly redo symlinks
RUN rm /usr/bin/python
RUN ln -s /usr/bin/python3 /usr/bin/python




### {{ emacs }}

#USER command in DOCKER to run correctly as me
USER pbrian
RUN emacs -nw --script $USERHOME/.emacs 
USER root



### {{ latex }}

RUN apt-get install -y texlive-base \
                       texlive-latex-recommended \
                       texlive-latex-extra \
                       texlive-fonts-recommended \
                       texlive-fonts-extra \
                       texlive-latex-base \
                       texlive-font-utils 


EXPOSE 22
ENTRYPOINT ["/home/pbrian/ENTRYPOINT.sh"]


