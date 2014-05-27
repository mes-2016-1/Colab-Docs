Postgres Server 9.3
===================
Para a instalação do funcionar corretamente com outras aplicações as portas necessárias devem estar abertas para a máquina virtual que acessá-la

Install postgresql

.. code-block::

    sudo yum localinstall http://yum.postgresql.org/9.3/redhat/rhel-6-x86_64/pgdg-centos93-9.3-1.noarch.rpm -y
    sudo yum install postgresql93 postgresql93-devel postgresql93-libs postgresql93-server -y

Initialize database

.. code-block::

    sudo service postgresql-9.3 initdb

Start postgresql with the system

.. code-block::

    sudo chkconfig postgresql-9.3 on

Start postgresql

.. code-block::

    sudo service postgresql-9.3 start

Put the binaries of postgres in the PATH variable

.. code-block::

    echo "export PATH=$PATH:/usr/pgsql-9.3/bin/" >> ~/.bashrc
    source ~/.bashrc
    sudo su
    echo "export PATH=$PATH:/usr/pgsql-9.3/bin/" >> ~/.bashrc
    source ~/.bashrc
    exit

Edit sudoers file

.. code-block::

    sudo vim /etc/sudoers
    
Inside sudoers file change the line

.. code-block::

    Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin
    
To this line

.. code-block::

    Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/pgsql-9.3/bin/

And save the file

.. code-block::

    [ESC]:wq!

Create a password for postgresql database, in this case we have an user called ``colab``, and we  will set its password too.

.. code-block::

    sudo -u postgres psql
    
.. code-block::

    CREATE USER colab SUPERUSER INHERIT CREATEDB CREATEROLE;
    ALTER USER colab PASSWORD 'colab';

    CREATE DATABASE colab;
    CREATE DATABASE "trac_colab" WITH OWNER "colab" ENCODING 'UTF8' LC_COLLATE='en_US.UTF-8' LC_CTYPE='en_US.UTF-8' TEMPLATE=template0;
    \q

Edit again with template1

.. code-block::    

    sudo -u postgres psql -d template1

.. code-block::

    CREATE USER git;
    CREATE DATABASE gitlabhq_production OWNER git;
    \q

Edit the users permissions on pg_hba.conf

.. code-block::

    sudo vi /var/lib/pgsql/9.3/data/pg_hba.conf

Set the permission like this

.. code-block::

    # TYPE  DATABASE        USER            ADDRESS                 METHOD
    
    # "local" is for Unix domain socket connections only
    local   all             git                                     trust
    local   all             postgres                                     peer
    local   all             colab                                     md5
    # IPv4 local connections:
    host    gitlabhq_production             git             127.0.0.1/32            trust
    host    all             postgres             127.0.0.1/32            ident
    host    all             colab             127.0.0.1/32            md5
    # IPv6 local connections:
    host    gitlabhq_production             git             ::1/128                 trust
    host    all             postgres             ::1/128                 ident
    host    all             colab             ::1/128                 md5

.. code-block::

    [ESC]:wq!
  
Restart the postgresql

.. code-block::

    sudo service postgresql-9.3 restart



