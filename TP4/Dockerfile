FROM debian

RUN apt update -y && \
    apt install -y apache2 

ADD index.html /var/www/html

ADD apache2Conf.conf /etc/apache2/apache2.conf 

RUN mkdir /etc/apache2/logs

CMD [ "/usr/lib/apache2", "-g", "daemon off;" ]

