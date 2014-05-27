Gitlab
======
Para a instalação do funcionar corretamente com outras aplicações as portas necessárias devem estar abertas para a máquina virtual que acessá-la

*NOTE:*

    Source Tutorial: `https://github.com/gitlabhq/gitlab-recipes/tree/master/install/centos <https://github.com/gitlabhq/gitlab-recipes/tree/master/install/centos>`_ 

Add EPEL repository

.. code-block::

    sudo wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6 https://www.fedoraproject.org/static/0608B895.txt
    sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6

Add PUIAS Computational repository (necessary to download some exclusive dependencies)

.. code-block::

    sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    sudo wget -O /etc/yum.repos.d/PUIAS_6_computational.repo https://gitlab.com/gitlab-org/gitlab-recipes/raw/master/install/centos/PUIAS_6_computational.repo
    sudo wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-puias http://springdale.math.ias.edu/data/puias/6/x86_64/os/RPM-GPG-KEY-puias
    sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-puias
    
Enable PUIAS repository

.. code-block::

    sudo yum -y install yum-utils
    sudo yum-config-manager --enable epel --enable PUIAS_6_computational
    
Update all packages, and install some addtional packages

.. code-block::

    sudo yum -y update
    sudo yum -y groupinstall 'Development Tools'
    sudo yum -y install readline readline-devel ncurses-devel gdbm-devel glibc-devel tcl-devel openssl-devel curl-devel expat-devel db4-devel byacc sqlite-devel libyaml libyaml-devel libffi libffi-devel libxml2 libxml2-devel libxslt libxslt-devel libicu libicu-devel system-config-firewall-tui redis sudo wget crontabs logwatch logrotate perl-Time-HiRes

Add redis to start with the system, and start it

.. code-block::

    sudo chkconfig redis on
    sudo service redis start

Install the mail server ``postfix``
    
.. code-block::

    sudo yum -y install postfix

Remove any git package that you may had install

.. code-block::

    sudo yum -y remove git

Install git 1.9.0 and its dependencies
    
.. code-block::

    sudo yum -y install zlib-devel perl-CPAN gettext curl-devel expat-devel gettext-devel openssl-devel
    sudo mkdir /tmp/git && cd /tmp/git
    sudo wget https://git-core.googlecode.com/files/git-1.9.0.tar.gz
    sudo tar xzf git-1.9.0.tar.gz
    cd git-1.9.0/
    sudo ./configure
    sudo make
    sudo make prefix=/usr/local install

Remove any ruby installed before, and download ``ruby-2.0.0-p451``

.. code-block::

    sudo yum remove ruby -y
    mkdir /tmp/ruby && cd /tmp/ruby
    sudo curl --progress ftp://ftp.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p451.tar.gz | tar xz
    
*NOTE:*

    If you can't reach the host from ``ruby-2.0.0-p451``, you alsa can try this command: sudo curl --progress http://cache.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p451.tar.bz2 | tar xj
    
Install ruby 2.0.0

.. code-block::

    cd ruby-2.0.0-p451
    ./configure --disable-install-rdoc
    make
    sudo make prefix=/usr/local install
    
Install the bundler gem

.. code-block::

    sudo /usr/local/bin/gem install bundler --no-ri --no-rdoc

Create the user ``git`` to give the rights permissions to Gitlab application

.. code-block::

    sudo adduser --system --shell /bin/bash --comment 'GitLab' --create-home --home-dir /home/git/ git
    
Clone the gitlab-shell repository

.. code-block::

    sudo su
    cd /home/git
    sudo -u git -H /usr/local/bin/git clone https://gitlab.com/gitlab-org/gitlab-shell.git
    cd gitlab-shell/
    /usr/local/bin/git reset --hard v1.9.3

Configure the host name and install gitlab-shell

.. code-block::

    sudo -u git -H cp config.yml.example config.yml
    sudo -u git -H vim config.yml
    sudo -u git -H /usr/local/bin/ruby ./bin/install
    restorecon -Rv /home/git/.ssh
 
The database ``gitlabhq_production``, should be already created in the postgreSQL VM
And the git user in pg_hba.conf(of that VM) to grant the permissions

Clone and configure the ``gitlab`` repository

.. code-block::

    cd /home/git
    sudo -u git -H /usr/local/bin/git clone https://github.com/colab-community/gitlabhq.git -b 6-8-stable gitlab
    cd /home/git/gitlab
    sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml
    chown -R git {log,tmp}
    chmod -R u+rwX {log,tmp}
    sudo -u git -H mkdir /home/git/gitlab-satellites
    chmod u+rwx,g+rx,o-rwx /home/git/gitlab-satellites
    chmod -R u+rwX tmp/{pids,sockets}
    chmod -R u+rwX public/uploads
    sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb
    sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb

If you are using the port 8080 change the unicorn file

.. code-block::

    sudo vim /home/git/gitlab/config/unicorn.rb
    
Change

.. code-block::

    listen "127.0.0.1:8080", :tcp_nopush => true

To

.. code-block::

    listen "127.0.0.1:6000", :tcp_nopush => true

.. code-block::

    [ESC]:wq!

Configure git and database

.. code-block::

    sudo -u git -H /usr/local/bin/git config --global user.name "GitLab"
    sudo -u git -H /usr/local/bin/git config --global user.email "gitlab@localhost"
    sudo -u git -H /usr/local/bin/git config --global core.autocrlf input
    sudo -u git cp config/database.yml.postgresql config/database.yml
    sudo -u git -H chmod o-rwx config/database.yml

Configure the bundle

.. code-block::

    cd /home/git/gitlab
    sudo -u git -H /usr/local/bin/bundle config build.pg --with-pg-config=/usr/pgsql-9.3/bin/pg_config
    sudo -u git -H /usr/local/bin/bundle config build.nokogiri --use-system-libraries

Edit sudores file to let ``bundle``, ``git`` and ``gem`` execute in sudo mode. 

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

Give the bundle install to install the required gems, if you are going to devel to gitlab change the env to ``RAILS_ENV=development``

.. code-block::

    sudo -u git -H /usr/local/bin/bundle install --deployment --without development test mysql aws
    sudo -u git -H /usr/local/bin/bundle exec rake gitlab:setup RAILS_ENV=production

Type ``yes`` to create the database tables

*NOTE:*
    
    Admin login and password -- login: admin@local.host -- password: 5iveL!fe

Add gitlab to start with system, this step is not require to development mode

.. code-block::

    wget -O /etc/init.d/gitlab https://gitlab.com/gitlab-org/gitlab-recipes/raw/master/init/sysvinit/centos/gitlab-unicorn
    chmod +x /etc/init.d/gitlab
    chkconfig --add gitlab
    chkconfig gitlab on
    cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab
    service gitlab start
    
Compile the asstes, to development change the env to ``RAILS_ENV=development``

.. code-block::

    sudo -u git -H /usr/local/bin/bundle exec rake assets:precompile RAILS_ENV=production
 
Configure the nginx to the production mode

.. code-block::

    sudo cp /home/git/gitlab/lib/support/nginx/gitlab /etc/nginx/conf.d/gitlab.conf

.. code-block::

    sudo vim /etc/nginx/conf.d/gitlab.conf

Change

.. code-block::

    listen *:80 default_server; # e.g., listen 192.168.1.1:80; In most cases *:80 is a good idea
    server_name YOUR_SERVER_FQDN; # e.g., server_name source.example.com;

To

.. code-block::

    listen 8090; # e.g., listen 192.168.1.1:80; In most cases *:80 is a good idea
    server_name localhost; # e.g., server_name source.example.com;
    
.. code-block::

    [ESC]:wq!
    
Add nginx to git user group, and give the permission to git user folder

.. code-block::

    usermod -a -G git nginx
    chmod g+rx /home/git/
    
Restart Nginx

.. code-block::

    service nginx restart
    
Restart gitlab

.. code-block::

    sudo service gitlab restart

*NOTE:*

    You can access gitlab in this url: `http://localhost:8090 <http://localhost:8090>`_
