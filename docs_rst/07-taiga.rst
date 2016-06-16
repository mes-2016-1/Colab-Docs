Taiga
=======
Para a instalação do funcionar corretamente com outras aplicações as portas necessárias devem estar abertas para a máquina virtual que acessá-la

Taiga-Vagrant
=========

Clone the taiga-vagrant repository

.. code-block::
    git clone https://github.com/taigaio/taiga-vagrant.git


Add this line to the Vagrantfile

.. code-block::
    config.vm.network "private_network", ip: "172.28.124.3"

Boot the VM environment running this command on terminal

.. code-block::
    vagrant up

Open the taiga.conf file

.. code-block::
vi /etc/nginx/sites-available/taiga

Replace all the code with this new code

.. code-block::
    server {
        listen 80 default_server;
        listen 8000 default_server;
        server_name _;
        root /home/vagrant/taiga-front/dist/;

        large_client_header_buffers 4 32k;

        client_max_body_size 50M;
        charset utf-8;

        access_log /home/vagrant/logs/nginx.access.log;
        error_log /home/vagrant/logs/nginx.error.log;

        location / {
            try_files $uri $uri/ /index.html;
        }

        location /api {
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Scheme $scheme;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:8001/api;
            proxy_redirect off;
        }

        location /static {
            alias /home/vagrant/taiga-back/static;
        }

        location /media {
            alias /home/vagrant/taiga-back/media;
        }
    }

Add this line before the "<!-- Main meta-->" line

.. code-block::
    <base href="/taiga/" />


Colab configuration
=========

Add this line to the Vagrantfile

.. code-block::
    config.vm.network "private_network", ip: "172.28.124.4"

Clone the colab-taiga-plugin repository

.. code-block::
    git clone https://github.com/mes-2016-1/colab-taiga-plugin.git

Go inside the colab_taiga_plugin folder through the colab VM and run
.. code-block::
    pip install -e .

Create sites-available folder inside nginx

.. code-block::
sudo mkdir /etc/nginx/sites-available

Create the colab.conf file running and open it

.. code-block::
sudo vi /etc/nginx/sites-available/colab

Add the following code to the colab file and save it

.. code-block::
    server {
      listen                8001;
      server_name           _;

      access_log            /var/log/nginx/colab.access.log;
      error_log             /var/log/nginx/colab.error.log;

      location / {
        proxy_pass http://0.0.0.0:8000;
      }

      location /v-1463481183206 {
        proxy_pass http://172.28.128.3;
      }

      location = /conf.json {
        proxy_pass http://172.28.128.3;
      }
    }


Open the taiga.py file

.. code-block::
    vi /etc/colab/plugins.d/taiga.py

Change the upstream to the following

.. code-block::
    upstream = 'http://172.28.128.3'

Open the file

.. code-block::
    sudo vi /etc/nginx/nginx.conf

Add the following line below "include  /etc/nginx/mime.types;"

.. code-block::
    include             /etc/nginx/sites-available/*;


Restart de nginx service

.. code-block::
sudo service nginx restart
