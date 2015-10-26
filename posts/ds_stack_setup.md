---
title: Fitting IPython Notebooks, Spark and Cassandra all together
author: carlos
date: "2015-10-23"
description: An article on how to configure all these tools together
tags: ["Cassandra", "Spark", "DataStax Enterprise", "IPython Notebooks", "IPython"]
categories: ["devops"]
---

Yesterday after more than a whole week working on this I've finally managed to set all this stack up.

This stack is one of the most hot and trending topics nowadays as it is very useful for BigData, specifically for data exploration purposes.

My starting point was a 3 nodes Cassandra cluster, intended for analytics, data exploration and adhoc reporting. This cluster was running
DSE (DataStax Enterprise) 4.6, which ships with Cassandra 2.0. Worths mentioning that the cluster was running in Cassandra only mode.

After having attended to the latest [Cassandra Summit '15](http://mrcalonso.com/notes-on-cassandra-summit-2015/), it was made clear not to use
Spark with any Cassandra earlier than 2.1.8, because the integration was buggy. Therefore:

1. Upgrade DSE to latest (4.8) version, that includes Cassandra 2.1.10.
  * Step by step instructions here: [http://docs.datastax.com/en/upgrade/doc/upgrade/datastax_enterprise/upgradeDSE47.html](http://docs.datastax.com/en/upgrade/doc/upgrade/datastax_enterprise/upgradeDSE47.html)
2. Next step is to enable Spark on the cluster. This is one of the points where relying on something like DSE comes really helpful, as the DSE distribution comes
with a Spark installation and the Cassandra-Spark connector already configured and optimised for maximum compatbility and throughput. It is also really easy to
enable Spark on the nodes.
  * For more information on this, head here: [http://docs.datastax.com/en/datastax_enterprise/4.8/datastax_enterprise/anaHome/anaHomeTOC.html](http://docs.datastax.com/en/datastax_enterprise/4.8/datastax_enterprise/anaHome/anaHomeTOC.html)
3. Once I had Cassandra and Spark running on all nodes and one of them was designated as the Spark Master (first of the Cassandra seeds by default, but you can check it with `dse client-tool spark master-address`), it's time to install IPython and all its dependencies:
  1. First step is to install all system required packages (use apt-get, yum, etc depending on your OS): `sudo apt-get install build-essential libcurl4-openssl-dev libssl-dev zlib1g-dev libpcre3-dev gfortran libblas-dev libblas3gf liblapack3gf liblapack-dev libncurses5-dev libatlas-dev libatlas-base-dev libscalapack-mpi1 libscalapack-pvm1 liblcms-utils python-imaging-doc python-imaging-dbg libamd2.3.1 libjpeg-turbo8 libjpeg8 liblcms1 libumfpack5.6.2 python-imaging libpng12-0 libpng12-dev libfreetype6 libfreetype6-dev libcurl4-gnutls-dev python-pycurl-dbg git-core cython libhdf5-7 libhdf5-serial-dev python-egenix-mxdatetime vim python-numpy python-scipy pandoc openjdk-7-jdk`
  2. Next step is to install [Python Virtualenv](http://docs.python-guide.org/en/latest/dev/virtualenvs/), as IPython depends on Python and changing the system's Python installation can be dangerous: `sudo pip install virtualenv`
  3. Then, create a folder for your installation. This folder will contain the virtual environment for the notebooks installation (ipynb in this example). `mkdir ipynb` and `cd ipynb`
  4. Create a virtual environment: `virtualenv ipython`. Where ipython is the name of the virtual environment we're creating.
  5. To begin using the virtual environment we need to activate it: `source ipython/bin/activate`. At this point our prompt will indicate us that we're inside the `ipython` virtual env.
  6. Install all IPython's dependencies (using pip): `pip install uwsgi numpy freetype-py pillow scipy python-dateutil pytz six scikit-learn pandas matplotlib pygments readline nose pexpect cython networkx numexpr tables patsy statsmodels sympy scikit-image theano xlrd xlwt ipython[notebook]`
4. Let's create a IPython default profile (we'll not use it, but it's safe to create it to avoid bugs and strange issues)
  * `./ipython/bin/ipython profile create --profile=default --ipython-dir .ipython`
5. Then we create the pyspark ipython profile we'll use.
  * `./ipython/bin/ipython profile create --profile=pyspark --ipython-dir .ipython`
6. Now install MathJax extension:
  * `python -c "from IPython.external.mathjax import install_mathjax; install_mathjax(replace=True, dest='~/ipynb/ipython/lib/python2.7/site-packages/IPython/html/static/mathjax')"`
7. Create now the following file under `~/ipynb/.ipython/profile_pyspark/ipython_notebook_config.py`

```
c = get_config()

# The IP address the notebook server will listen on.
# If set to '*', will listen on all interfaces.
c.NotebookApp.ip= '*'

# Port to host on (e.g. 8888, the default)
c.NotebookApp.port = 8888

# Open browser (probably want False)
c.NotebookApp.open_browser = False
```

8. Create now the following file under `~/ipynb/.ipython/profile_pyspark/startup/00-pyspark-setup.py`

```
import os
import sys

spark_home = os.environ.get('SPARK_HOME', None)
if not spark_home:
  raise ValueError('SPARK_HOME environment variable is not set')
sys.path.insert(0, os.path.join(spark_home, 'python'))
sys.path.insert(0, os.path.join(spark_home, 'python/lib/py4j-0.8.2.1-src.zip'))
execfile(os.path.join(spark_home, 'python/pyspark/shell.py'))
```

9. Prepare the environment:
  1. `export SPARK_HOME=/usr/share/dse/spark`
  2. `export PYSPARK_SUBMIT_ARGS='--master spark://<spark_master_host>:<spark_master_port> pyspark-shell'`

10. Start your Notebooks server!!:
  * `ipython/bin/ipython notebook --profile=pyspark`

Now you should be able to browse to `<host_running_notebooks_server>:8888` and and see the WebUI!

Finally, check that everything is working by creating a new notebook and typing `sc`. You should see `<pyspark.context.SparkContext at 0x7fc70ac8af10>`

## Troubleshooting:

1. I've followed the steps but when I type and run `sc`, an empty string is returned.
  * Make sure environment variables defined at step 9 are properly set.
  * Make sure the `py4j-0.8.2.1-src.zip` actually exists under `SPARK_HOME/python/lib` (Update your version of it on the startup script accordingly).
  * That's because the startup script (saved at step 8 under ...startup/00-pyspark-setup.py) hasn't run properly. Try to run it's contents as a notebook to debug what's happening.

2. When running the startup script as a notebook I get: `Java gateway process exited before sending the driver its port number`
  * You are not running using Java JDK but JRE instead.

