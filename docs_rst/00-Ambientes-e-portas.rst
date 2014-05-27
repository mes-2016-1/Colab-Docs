Ambientes
=========

Para a funcionamento correto destes tutoriais de instalação, deve-se primeiro garantir que todas as portas estão devidamente abertas e com os acessos necessários.

As seguintes portas são utilizadas:

- 8090 -> Gitlab
- 9080 -> Redmine
- 8983 -> Solr
- 5000 -> Trac
- 25   -> Postfix(envio de emails)
- 80   -> Colab
- 3690 ?-> SVN
- 9418 ?-> GIT
- 5432 -> PostgreSQL

As máquinas para o desenvolvimento estão mapeadas de acordo com as ferramentas a serem instaladas:

- 1> Colab                 (Deve acessar todas, em todas as portas listadas acima)
- 2> Squid
- 3> Noosfero              (Deve acessar #11 na porta do PostgreSQL)
- 4> Redmine, Gitlab, Trac (Deve acessar #11 na porta do PostgreSQL)
- 5> Git, SVN              (Deve acessar #11 na porta do PostgreSQL)
- 6> Mailman               (Deve acessar porta do Postfix para envio externo)
- 7> OpenFire
- 8> Jenkins
- 9> Mezuro/Analizo
- 10> Solr
- 11> PostgreSQL

