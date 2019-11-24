# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
ARG BASE_CONTAINER=jupyter/scipy-notebook
FROM $BASE_CONTAINER

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

USER root

# Spark dependencies
ENV APACHE_SPARK_VERSION 2.4.4
ENV HADOOP_VERSION 2.7

RUN apt-get -y update && \
    apt-get install --no-install-recommends -y openjdk-8-jre-headless ca-certificates-java && \
    rm -rf /var/lib/apt/lists/*

RUN cd /tmp && \
    wget -q http://mirrors.ukfast.co.uk/sites/ftp.apache.org/spark/spark-${APACHE_SPARK_VERSION}/spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz && \
    echo "2E3A5C853B9F28C7D4525C0ADCB0D971B73AD47D5CCE138C85335B9F53A6519540D3923CB0B5CEE41E386E49AE8A409A51AB7194BA11A254E037A848D0C4A9E5 *spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz" | sha512sum -c - && \
    tar xzf spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz -C /usr/local --owner root --group root --no-same-owner && \
    rm spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz
RUN cd /usr/local && ln -s spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} spark

# Mesos dependencies
# Install from the Xenial Mesosphere repository since there does not (yet)
# exist a Bionic repository and the dependencies seem to be compatible for now.
COPY mesos.key /tmp/
RUN apt-get -y update && \
    apt-get install --no-install-recommends -y gnupg && \
    apt-key add /tmp/mesos.key && \
    echo "deb http://repos.mesosphere.io/ubuntu xenial main" > /etc/apt/sources.list.d/mesosphere.list && \
    apt-get -y update && \
    apt-get --no-install-recommends -y install mesos=1.2\* && \
    apt-get purge --auto-remove -y gnupg && \
    rm -rf /var/lib/apt/lists/*

# Spark and Mesos config
ENV SPARK_HOME /usr/local/spark
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.7-src.zip
ENV MESOS_NATIVE_LIBRARY /usr/local/lib/libmesos.so
ENV SPARK_OPTS --driver-java-options=-Xms1024M --driver-java-options=-Xmx4096M --driver-java-options=-Dlog4j.logLevel=info

#Setup bashrc for class
ENV BASHRC /home/${NB_USER}/.bashrc
RUN touch ${BASHRC}
RUN echo "export PATH=${SPARK_HOME}:${PATH}" >> ${BASHRC}
#RUN echo "export PYSPARK_DRIVER_PYTHON=\"jupyter\"" >> ${BASHRC}
#RUN echo "export PYSPARK_DRIVER_PYTHON_OPTS=\"notebook\"" >> ${BASHRC}
#RUN echo "export PYSPARK_PYTHON=python3" >> ${BASHRC}
RUN fix-permissions /home/${NB_USER}
RUN pip install py4j 
RUN pip install findspark
RUN apt-get -y update && \
    apt-get install --no-install-recommends -y scala
RUN apt-get install -y libmysqlclient-dev && \
    apt-get install -y python3-dev && \
    apt-get install -y libaio1
RUN pip install mysqlclient
RUN mkdir /opt/oracle
ENV ORACLEVER oracle_client_11_2
ENV ORACLEZIP ${ORACLEVER}.zip
COPY ${ORACLEZIP} /opt/oracle
RUN cd /opt/oracle && unzip ${ORACLEZIP} && rm -f ${ORACLEZIP}
RUN echo /opt/oracle/oracle_client_11_2 >> /etc/ld.so.conf.d/oic.conf
RUN sudo ldconfig
RUN echo "export ORACLE_HOME=/opt/oracle/${ORACLEVER}" >> ${BASHRC}
RUN echo "export LD_LIBRARY_PATH=/opt/oracle/${ORACLEVER}" >> ${BASHRC}
RUN pip install cx_Oracle
RUN pip install sqlalchemy

USER $NB_UID

# Install pyarrow
RUN conda install --quiet -y 'pyarrow' && \
    conda clean --all -f -y && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER
