FROM mcr.microsoft.com/java/jre-headless:11-zulu-alpine-with-tools

LABEL maintainer="janedoe@outlook.com"


# NOTE: Using the latest version of Tomcat (as of March 2020)
ENV TOMCAT_VERSION 9.0.31
ENV TOMCAT_MAJOR 9


ENV PORT 80
ENV SSH_PORT 2222

ENV PATH /usr/local/tomcat/bin:$PATH

# Remove the sample webapps provided by Tomcat
#RUN rm -rf /usr/local/tomcat/webapps/

COPY tmp/shared/misc/init_container.sh /bin/init_container.sh
COPY tmp/shared/misc/sshd_config /etc/ssh/

RUN apk add --update openssh-server bash openrc \
        && rm -rf /var/cache/apk/* \
        # Remove unnecessary services
        && rm -f /etc/init.d/hwdrivers \
                 /etc/init.d/hwclock \
                 /etc/init.d/mtab \
                 /etc/init.d/bootmisc \
                 /etc/init.d/modules \
                 /etc/init.d/modules-load \
                 /etc/init.d/modloop \
        # Can't do cgroups
        && sed -i 's/\tcgroup_add_service/\t#cgroup_add_service/g' /lib/rc/sh/openrc-run.sh \
        && echo "root:Docker!" | chpasswd \
        && wget -O /tmp/apache-tomcat-$TOMCAT_VERSION.tar.gz https://archive.apache.org/dist/tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz --no-verbose \
        && tar xvzf /tmp/apache-tomcat-$TOMCAT_VERSION.tar.gz -C /tmp \
        && mv /tmp/apache-tomcat-$TOMCAT_VERSION /usr/local/tomcat \
        && rm -rf /usr/local/tomcat/webapps \
        && rm -f /tmp/apache-tomcat-$TOMCAT_VERSION.tar.gz \
        && chmod 755 /bin/init_container.sh 

COPY tmp/shared/tomcat/9.0/server.xml /usr/local/tomcat/conf/server.xml
COPY tmp/shared/tomcat/9.0/catalina.properties /usr/local/tomcat/conf/catalina.properties
COPY tmp/shared/tomcat/9.0/context.xml /usr/local/tomcat/conf/context.xml
COPY tmp/shared/tomcat/9.0/logging.properties /usr/local/tomcat/conf/logging.properties
COPY tmp/shared/tomcat/9.0/web-appservice.xml /usr/local/tomcat/conf/web.xml
COPY tmp/shared/easyauth/azure.appservice.easyauth.jar /usr/local/tomcat/lib/
COPY tmp/shared/appservice/azure.appservice.jar /usr/local/tomcat/lib/
COPY tmp/shared/tomcat/common/setenv.sh /usr/local/tomcat/bin/

#NOTE: This is where you copy your Maven package to the Docker Image
COPY target/MyMavenWebApp.war /home/site/wwwroot/webapps/ROOT.war


EXPOSE 80 2222

ENTRYPOINT ["/bin/init_container.sh"]