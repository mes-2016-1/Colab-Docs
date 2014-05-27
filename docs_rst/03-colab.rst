Python 2.7 + Django 1.6
=======================
Para a instalação do funcionar corretamente com outras aplicações as portas necessárias devem estar abertas para a máquina virtual que acessá-la

Nginx 1.6
=========

Download the nginx

.. code-block::
    cd /tmp
    wget http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
    sudo rpm -ivh nginx-release-centos-6-0.el6.ngx.noarch.rpm

Install nginx

.. code-block::

    sudo yum install nginx -y

Start nginx with the system

.. code-block::

    sudo chkconfig nginx on

Install the devel tools to build specific python 2.7 modules

.. code-block::

    sudo yum groupinstall "Development tools" -y
    sudo yum install zlib-devel bzip2-devel openssl-devel ncurses-devel libxslt-devel -y

Download and compile Python 2.7

.. code-block::

    cd /tmp
    sudo wget --no-check-certificate https://www.python.org/ftp/python/2.7.6/Python-2.7.6.tar.xz
    sudo tar xf Python-2.7.6.tar.xz
    cd Python-2.7.6
    sudo ./configure --prefix=/usr/local
    sudo make
    
Install python 2.7 as an alternative python, because cent os uses python 2.6 in the system.
    
.. code-block::

    sudo make altinstall

Update the PATH variable to execute python as root.

.. code-block::

    sudo su
    echo "export PATH=$PATH:/usr/local/bin/" >> ~/.bashrc
    source ~/.bashrc
    exit

Install the easy_install for python 2.7

.. code-block::

    cd /tmp
    sudo wget https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py
    sudo /usr/local/bin/python2.7 ez_setup.py
    
Instal pip 2.7

.. code-block::

    sudo /usr/local/bin/easy_install-2.7 pip

Install additional packages to python.

.. code-block::

    sudo yum remove libevent -y
    sudo yum install mercurial libevent-devel python-devel -y

Edit sudores file to let ``python2.7`` execute in sudo mode. 

*NOTE:*

    The path ``/usr/bin:/usr/pgsql-9.3/bin/`` will be only in this file if you installed postgresql before, if you didn't just remove it from those lines.

.. code-block::

    sudo vim /etc/sudoers

Change the line

.. code-block::

    Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/pgsql-9.3/bin/
    
To

.. code-block::

    Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/pgsql-9.3/bin/:/usr/local/bin/
    
.. code-block::

    [ESC]:wq!
    
Django 1.6

Install django and uwsgi

.. code-block::

    sudo pip2.7 install django
    sudo pip2.7 install uwsgi

Colab
=====

Install git and clone colab

.. code-block::

    sudo yum install git -y
    cd /opt
    sudo git clone https://github.com/colab-community/colab.git
    
Install colab requirements

.. code-block::

    sudo pip2.7 install mimeparse
    sudo pip2.7 install -r /opt/colab/requirements.txt
    
Create the local_settings file in colab folder

.. code-block::

    sudo cp /opt/colab/src/colab/local_settings-dev.py /opt/colab/src/colab/local_settings.py

And edit it inserting browser id in the end of file

.. code-block::

    sudo vim /opt/colab/src/colab/local_settings.py
    
.. code-block::

    BROWSERID_AUDIENCES = [SITE_URL, SITE_URL.replace('https', 'http')]

Edit also the the correct host for gitlab, redmine, trac, etc

.. code-block::

    COLAB_TRAC_URL = 'http://localhost:5000/trac/'
    COLAB_CI_URL = 'http://localhost:8080/ci/'
    COLAB_GITLAB_URL = 'http://localhost:8090/gitlab/'
    COLAB_REDMINE_URL = 'http://localhost:9080/redmine/'
    
.. code-block::

    [ESC]:wq!

Build the solr schema.xml and give it to solr

.. code-block::

    cd /opt/colab/src
    sudo su
    python2.7 manage.py build_solr_schema > /usr/share/solr/example/solr/collection1/conf/schema.xml
    exit

Edit the schema to change the ``stopwords_en.txt`` to ``lang/stopwords_en.txt``

.. code-block::

    sudo vim /usr/share/solr/example/solr/collection1/conf/schema.xml

.. code-block::

    [ESC]:%s/stopwords_en.txt/lang\/stopwords_en.txt
    [ESC]:wq!


Syncronize and migrate the colab's database

.. code-block::

    cd /opt/colab/src
    python2.7 manage.py syncdb
    python2.7 manage.py migrate

Start Solr in a terminal, and then, in other terminal, update colab index

.. code-block::

        cd /opt/colab/src
        python2.7 manage.py update_index

Now you can close this terminal, and stop solr with ``Ctrl+C``

Import mailman e-mails

.. code-block::

    sudo python2.7 /opt/colab/src/manage.py import_emails

*NOTE:*

    To run Colab: python2.7 /opt/colab/src/manage.py runserver . To access colab go in: `http://localhost:8000 <http://localhost:8000>`_
