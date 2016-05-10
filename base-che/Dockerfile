# Dockerizing image for eXo Platform and Che development.
#
FROM       exoplatform/base-jdk:jdk8
MAINTAINER GREAU Maxime <mgreau+docker@exoplatform.com>

ENV LANG C.UTF-8

# Environment variables
ENV EXO_VERSION 4.3.0
ENV EXO_EDITION community
ENV EXO_DOWNLOAD http://sourceforge.net/projects/exo/files/Platform4.3/eXo-Platform-${EXO_EDITION}-${EXO_VERSION}.zip/download

ENV EXO_APP_DIR   /opt/exo
ENV EXO_CONF_DIR  /etc/exo
ENV EXO_DATA_DIR  /srv/exo
ENV EXO_LOG_DIR   /var/log/exo
ENV EXO_TMP_DIR   /tmp/exo-tmp

# URL used to download configuration files
ENV EXO_CHE_CONF_URL https://raw.githubusercontent.com/exo-docker/exo-che/master/base-che

# user for eXo Platform
ENV EXO_USER exo
ENV EXO_GROUP ${EXO_USER}

# user for Che Server
ENV CHE_USER user
ENV CHE_GROUP ${EXO_USER}

ENV MAVEN_VERSION=3.2.5
ENV M2_HOME=/home/${CHE_USER}/apache-maven-$MAVEN_VERSION
ENV PATH=$JAVA_HOME/bin:$M2_HOME/bin:$PATH

# add eXo user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN useradd --create-home --user-group --shell /bin/bash ${EXO_USER}
# giving all rights to eXo user
RUN echo "exo   ALL = NOPASSWD: ALL" > /etc/sudoers.d/exo && chmod 440 /etc/sudoers.d/exo

# add Che user
RUN useradd --create-home --user-group --shell /bin/bash ${CHE_USER}
# giving all rights to user user
RUN echo "user   ALL = NOPASSWD: ALL" > /etc/sudoers.d/user && chmod 440 /etc/sudoers.d/user

# Install some useful or needed tools
## Che Customization: Add git
RUN apt-get -qq update && \
  apt-get -qq -y upgrade ${_APT_OPTIONS} && \
  apt-get -qq -y install ${_APT_OPTIONS} libreoffice-calc libreoffice-draw libreoffice-impress libreoffice-math libreoffice-writer && \
  add-apt-repository ppa:git-core/ppa && \
  apt-get update && \
  sudo apt-get install git -y && \
  apt-get -qq -y autoremove && \
  apt-get -qq -y clean && \
  rm -rf /var/lib/apt/lists/*

# Create needed directories
RUN mkdir -p ${EXO_APP_DIR}
RUN mkdir -p ${EXO_DATA_DIR}  && chown ${EXO_USER}:${EXO_GROUP} ${EXO_DATA_DIR}
RUN mkdir -p ${EXO_TMP_DIR}   && chown ${EXO_USER}:${EXO_GROUP} ${EXO_TMP_DIR}
RUN mkdir -p ${EXO_LOG_DIR}   && chown ${EXO_USER}:${EXO_GROUP} ${EXO_LOG_DIR}
RUN ln -s ${EXO_APP_DIR}/current/gatein/conf /etc/exo

# Install eXo Platform
RUN curl -L -o /srv/downloads/eXo-Platform-${EXO_EDITION}-${EXO_VERSION}.zip ${EXO_DOWNLOAD} && \
    unzip -q /srv/downloads/eXo-Platform-${EXO_EDITION}-${EXO_VERSION}.zip -d ${EXO_APP_DIR} && \
    rm -f /srv/downloads/eXo-Platform-${EXO_EDITION}-${EXO_VERSION}.zip && \
    ln -s ${EXO_APP_DIR}/platform-${EXO_EDITION}-${EXO_VERSION} ${EXO_APP_DIR}/current && \
    chown -R ${EXO_USER}:${EXO_GROUP} ${EXO_APP_DIR}/current/

# Install Docker customization file
## Che Customization: Add file from URL
RUN curl -L -o ${EXO_APP_DIR}/current/bin/setenv-docker-customize.sh https://raw.githubusercontent.com/exo-docker/exo-community/master/setenv-docker-customize.sh

RUN chmod 755 ${EXO_APP_DIR}/current/bin/setenv-docker-customize.sh & chown ${EXO_USER}:${EXO_GROUP} ${EXO_APP_DIR}/current/bin/setenv-docker-customize.sh
RUN sed -i '/# Load custom settings/i \
\# Load custom settings for docker environment\n\
[ -r "$CATALINA_BASE/bin/setenv-docker-customize.sh" ] \
&& . "$CATALINA_BASE/bin/setenv-docker-customize.sh" \
|| echo "No Docker eXo Platform customization file : $CATALINA_BASE/bin/setenv-docker-customize.sh"\n\
' ${EXO_APP_DIR}/current/bin/setenv.sh && \
  grep 'setenv-docker-customize.sh' ${EXO_APP_DIR}/current/bin/setenv.sh

RUN rm -rf ${EXO_APP_DIR}/current/logs && ln -s ${EXO_LOG_DIR} ${EXO_APP_DIR}/current/logs

# Add eXo Custom Maven settings.xml and exo.properties
RUN mkdir -p /home/${EXO_USER}/.m2/repository && chown ${EXO_USER}:${EXO_USER} /home/${EXO_USER}/.m2/repository && \
    curl -L -o /home/${EXO_USER}/.m2/settings.xml ${EXO_CHE_CONF_URL}/m2-settings.xml && \
    chown ${EXO_USER}:${EXO_USER} /home/${EXO_USER}/.m2/settings.xml && \
    curl -L -o ${EXO_CONF_DIR}/exo.properties ${EXO_CHE_CONF_URL}/exo-sample.properties && \
    chown ${EXO_USER}:${EXO_USER} ${EXO_CONF_DIR}/exo.properties

# Install Maven on Che user home (required?)
USER ${CHE_USER}
RUN mkdir /home/${CHE_USER}/apache-maven-$MAVEN_VERSION && \
  wget -qO- "http://apache.ip-connect.vn.ua/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz" | tar -zx --strip-components=1 -C /home/user/apache-maven-$MAVEN_VERSION/
RUN echo "export JAVA_HOME=$JAVA_HOME\nexport M2_HOME=/home/user/apache-maven-$MAVEN_VERSION\nexport PATH=$JAVA_HOME/bin:$M2_HOME/bin:$PATH" >> /home/user/.bashrc

USER ${EXO_USER}

VOLUME ["/var/log/exo", "/srv/exo"]

EXPOSE 4403 8000 8080 9876 22

# TODO: Move to Task specific Dockerfile
RUN curl -L -o ${EXO_CONF_DIR}/catalog.json https://raw.githubusercontent.com/exo-docker/exo-che/master/task/catalog.json

# CMD for Che Server
CMD sudo /usr/sbin/sshd -D && \
    tail -f /dev/null
