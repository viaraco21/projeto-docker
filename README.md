# Projeto kube-news

### Objetivo
O projeto Kube-news é uma aplicação escrita em NodeJS e tem como objetivo ser uma aplicação de exemplo pra trabalhar com o uso de containers.

### Configuração
Pra configurar a aplicação, é preciso ter um banco de dados Postgre e pra definir o acesso ao banco, configure as variáveis de ambiente abaixo:

DB_DATABASE => Nome do banco de dados que vai ser usado.

DB_USERNAME => Usuário do banco de dados.

DB_PASSWORD => Senha do usuário do banco de dados.

DB_HOST => Endereço do banco de dados.

# comando para criar o container do postgres
docker container run -d -p 5432:5432 --name kube_news_db -e POSTGRES_PASSWORD=Pg123 -e POSTGRES_USER=Kubenews -e POSTGRES_DB=kubenews --network kube_news_net --mount type=volume,source=kube_news_vol,target=/var/lib/postgresql/data postgres:12.17

# comando para criar o container da aplicacao
docker container run -d -p 8080:8080 --name kubenews -e DB_DATABASE=kubenews -e DB_USERNAME=kubenews -e DB_PASSWORD=Pg123 -e DB_HOST=kube_news_db --network kube_news_net raco21/kube-news-docker:v1

# Pare e remova tudo do app/banco
docker stop kubenews kube_news_db
docker rm kubenews kube_news_db
docker volume rm kube_news_vol
docker network rm kube_news_net

# comando para criar a rede
docker network create kube_news_net

# comando para ver a rede
docker inspect kube_news_net

# comando para criar o volume
docker volume create kube_news_vol

# comando para criar o banco postgres

docker run -d \
  --name kube_news_db \
  --network kube_news_net \
  -e POSTGRES_PASSWORD=Pg123 \
  -e POSTGRES_USER=kubenews \
  -e POSTGRES_DB=kubenews \
  -p 5432:5432 \
  postgres:12.17

# Aguarde o banco inicializar completamente
sleep 10
docker logs kube_news_db

# Recrie o app (só depois do banco estar 100% OK!)
docker run -d \
  --name kubenews \
  --network kube_news_net \
  -e DB_DATABASE=kubenews \
  -e DB_USERNAME=kubenews \
  -e DB_PASSWORD=Pg123 \
  -e DB_HOST=kube_news_db \
  -p 8080:8080 \
  raco21/kube-news-docker:v1

# Teste de novo dentro do container da aplicação
docker exec -it kubenews sh
nc -zvw3 kube_news_db 5432