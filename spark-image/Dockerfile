FROM ubuntu:18.04

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV PATH /opt/conda/bin:$PATH

RUN apt-get update --fix-missing && apt-get install -y wget bzip2 ca-certificates \
    libglib2.0-0 libxext6 libsm6 libxrender1 \
    git mercurial subversion

RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda2-4.5.11-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate base" >> ~/.bashrc

RUN apt-get install -y curl grep sed dpkg && \
    TINI_VERSION=`curl https://github.com/krallin/tini/releases/latest | grep -o "/v.*\"" | sed 's:^..\(.*\).$:\1:'` && \
    curl -L "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini_${TINI_VERSION}.deb" > tini.deb && \
    dpkg -i tini.deb && \
    rm tini.deb 

# SPARK
RUN apt-get install -y openjdk-8-jdk zip unzip

RUN wget --quiet http://apache.mirror.cdnetworks.com/spark/spark-2.4.0/spark-2.4.0-bin-hadoop2.7.tgz && \
    tar xzvf spark-2.4.0-bin-hadoop2.7.tgz && \
    mv spark-2.4.0-bin-hadoop2.7 /opt/spark

RUN apt-get install -y vim

RUN rm spark-2.4.0-bin-hadoop2.7.tgz

RUN conda create -n python372 python=3.7.2 -y && \
    echo "conda activate python372" >> ~/.bashrc

RUN echo "spark.master spark://spark-master:7077" > /opt/spark/conf/spark-defaults.conf
RUN echo "spark.driver.port 10001" >> /opt/spark/conf/spark-defaults.conf
RUN echo "spark.blockManager.port 10002" >> /opt/spark/conf/spark-defaults.conf

RUN echo "export SPARK_WORKER_PORT=10000" > /opt/spark/conf/spark-env.sh

ENV SPARK_HOME /opt/spark
ENV JAVA_HOME /usr/lib/jvm/java-1.8.0-openjdk-amd64
ENV PATH $PATH:/opt/spark/bin

COPY start-master.sh /
COPY start-worker.sh /

RUN chmod 777 start-master.sh && \
    chmod 777 start-worker.sh

ENTRYPOINT [ "/usr/bin/tini", "--" ]
CMD ["/bin/bash"]
