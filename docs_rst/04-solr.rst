Solr 4.6.1
==========
Para a instalação do funcionar corretamente com outras aplicações as portas necessárias devem estar abertas para a máquina virtual que acessá-la

Download Solr and unpack it

.. code-block::

    cd /tmp
    sudo wget http://archive.apache.org/dist/lucene/solr/4.6.1/solr-4.6.1.tgz
    sudo tar xvzf solr-4.6.1.tgz
    
Install Solr in ``/usr/share``
    
.. code-block::

    sudo mv solr-4.6.1 /usr/share/solr
    sudo cp /usr/share/solr/example/webapps/solr.war /usr/share/solr/example/solr/solr.war

Remove the ``updateLog`` tag, editing the solrconfig.xml

.. code-block::

    sudo vim /usr/share/solr/example/solr/collection1/conf/solrconfig.xml
    
And remove those lines

.. code-block::

    <updateLog>
      <str name="dir">${solr.ulog.dir:}</str>
    </updateLog>
    
.. code-block::

    [ESC]wq! 

*NOTE:*

    To run Solr: cd /usr/share/solr/example/; sudo java -jar start.jar . And to access it `http://localhost:8983 <http://localhost:8983>`_

