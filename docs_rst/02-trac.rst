Trac 0.12
========
Para a instalação do funcionar corretamente com outras aplicações as portas necessárias devem estar abertas para a máquina virtual que acessá-la

Install the dependencies

.. code-block::

    sudo yum install gcc python-devel python-setuptools -y
    
Install this package to use Trac with postgresql

.. code-block::

    sudo easy_install psycopg2
    
Install Trac

.. code-block::

    sudo yum install -y trac

Intiate Trac

.. code-block::

    sudo mkdir -p /opt/trac
    sudo trac-admin /opt/trac initenv
    
In ``Project Name [My Project]>`` we used ``colab``. And if you are going to use the postgresql, put this line in ``Database connection string [sqlite:db/trac.db]>``, and we are using the user ``colab``.

.. code-block::

  postgres://colab:colab@/trac_colab?host=localhost

SVN Plugin

Install subversion

.. code-block::

    sudo yum install subversion -y
    
Create a repository and initiate it

.. code-block::

    mkdir -p /home/colab/myrepo
    mkdir -p /tmp/project/{branches,tags,trunk}
    svnadmin create /home/colab/myrepo/
    svn import /tmp/project file:///home/colab/myrepo/ -m "initial import"
    sudo rm -rf /tmp/project
    find /home/colab/myrepo -type f -exec chmod 660 {} \;
    find /home/colab/myrepo -type d -exec chmod 2770 {} \;

Edit the Trac's configuration file

.. code-block::

    sudo vim /opt/trac/conf/trac.ini
    
Inside the trac.ini file.
Replace the line

.. code-block::

    repository_dir = 
    
With this one

.. code-block::

    repository_dir = /home/colab/myrepo/
    
Insert those lines in the end of file to activate the view of subversion on Trac.

.. code-block::

    [components]
    tracopt.versioncontrol.svn.* = enabled

.. code-block::

    [ESC]:wq!

Remote User

Create the plugin to set the remote user variable

.. code-block::

    sudo vim /opt/trac/plugins/remote-user-auth.py
    
And put this in the file

.. code-block::

    from trac.core import *
    from trac.config import BoolOption
    from trac.web.api import IAuthenticator
    
    class MyRemoteUserAuthenticator(Component):
    
        implements(IAuthenticator)
    
        obey_remote_user_header = BoolOption('trac', 'obey_remote_user_header', 'false',
                   """Whether the 'Remote-User:' HTTP header is to be trusted for user logins 
                    (''since ??.??').""")
    
        def authenticate(self, req):
            if self.obey_remote_user_header and req.get_header('Remote-User'):
                return req.get_header('Remote-User')
            return None

Save the file

.. code-block::

    [ESC]:wq!

Edit Trac's configuration file

.. code-block::

    sudo vim /opt/trac/conf/trac.ini
    
Insert this line in the [trac] session.

.. code-block::
    
    obey_remote_user_header = true

Save and quit
    
.. code-block::

    [ESC]:wq!

*NOTE:*
    To run Trac: ``sudo tracd --port 5000 /opt/trac`` . And to access it `http://localhost:5000 <http://localhost:5000>`_ 
