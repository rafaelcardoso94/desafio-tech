# Desafio-tech
Desafio Tech - CI/CD

# Descrição Pipeline

Workflow criado para deploy de um tema no cms wordpress com versionamento das atualizações realizadas nele.

Trigger do workflow, qualquer commit realizado para branch "main".
```yaml
on:
  push:
    branches:
      - main
```
Step responsavel pela criação da pasta backup caso nao houvesse sido criada, pensando em um cenario de template, onde eu posso replicar a mesma esteira para varias aplicações no mesmo cenario.
```yaml
- name: Validation
  uses: appleboy/ssh-action@master
  with:
    host: server-hml.com
    username: username
    key: ${{ secrets.SSH_PRIVATE_KEY }} 
    script: |
      cd /var/www/
      if [ -d backup ]; then
        echo "A pasta já "backup" existe."
      else
        echo "A pasta "backup" não existe. Criando..."
        mkdir backup
      fi 
```   

Step responsavel pelo versionamento do tema, onde seto o template atual com a data que sera sofrido a alteracao e jogo em um diretorio backup para armazenamento do mesmo, dessa forma o novo sempre sobrescreve o atual, onde caso necessario um roolback apenas removemos a versao atualizada e sobrescrevemos com o renomeado. 
```yaml
- name: Versioning Themes
  uses: appleboy/ssh-action@master
    with:
      host: server-hml.com
      username: username
      key: ${{ secrets.SSH_PRIVATE_KEY }} #Secret Environment Hml
      script: |
        echo [Criando variavel para exibição de hora]
        export time=$(date +%Hh)
        echo [Criando variavel para exibição de dia,mes e ano] 
        export current_date=$(date +"%F")
        echo [Versionando Tema antigo]
        mv /var/www/html/wp-content/themes/twentyfifteen /var/www/html/wp-content/themes/twentyfifteen_${current_date}/${time}
        echo [Movendo tema versionando para pasta backup]
        mv /var/www/html/wp-content/themes/twentyfifteen_${current_date}/${time} /var/www/backup 
```            

Step responsavel por enviar tema atualizado do repositorio para o servidor on-premise.
```yaml
- name: Send New Themes
  uses: appleboy/scp-action@master
    with:
      host: server-hml.com
      username: username
      key: ${{ secrets.SSH_PRIVATE_KEY }}
      source: ./wordpress/wp-content/themes/twentyfifteen #Origem
      target: /var/www/html/wp-content/themes/            #Destino
```

Mesmo processo se repete para o ambiente de PRD, onde via github consiguimos separar as secrets e variaveis com mesmo nome alterando apenas o valor.
```yaml
environment: hml
environment: prd
```

