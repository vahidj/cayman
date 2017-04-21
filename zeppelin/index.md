---
layout: default
---

Zeppelin: Getting Started for Dummies

# [](#header-1)What is Zeppelin?

[Zeppelin](https://zeppelin.apache.org/) is a web-based interactive notebook for data processing (exploration, transformation, model building, etc.) and visualization.

# [](#header-1)Where to download?

You can download the latest release of Zeppelin from [https://zeppelin.apache.org/download.html](https://zeppelin.apache.org/download.html).

# [](#header-1)Which package should I download?

In this tutorial I will download the "Binary package with Spark interpreter and interpreter net-install script". If you are interested in interpreters other than Spark you can download the "Binary package with all interpreters" and if you are interested in building the tool from its source code you can download the source package.

# [](#header-1)Step by step guide

## [](#header-2)Download the package

```bash
cd /path/to/download/zeppelin
wget http://apache.claz.org/zeppelin/zeppelin-0.7.1/zeppelin-0.7.1-bin-netinst.tgz
```

## [](#header-2)Extract the archive

```bash
tar xzf zeppelin-0.7.1-bin-netinst.tgz
```

## [](#header-2)Configuring Zeppelin

```bash
cp conf/zeppelin-site.xml.template conf/zeppelin-site.xml
cp conf/zeppelin-env.sh.template conf/zeppelin-env.sh
```

find "zeppelin.server.port" and change the port number as you wish

edit zeppelin-env.sh and add the following lines:
export SPARK_HOME=/usr/local/spark
export SPARK_SUBMIT_OPTIONS="--driver-class-path /opt/hadoopgpl/lib/hadoop-lzo.jar"
