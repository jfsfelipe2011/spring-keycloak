## Descrição:

Esse é um repositório onde foi desenvolvido um api de teste simples e que
usa como autenticação/autorização o protocolo [SSO/Oauth2](https://www.mutuallyhuman.com/blog/choosing-an-sso-strategy-saml-vs-oauth2/).
Para isso foi usado um servidor com [Keycloak](https://www.keycloak.org/) para 
a geração de Token e o [Spring Boot](https://spring.io/projects/spring-boot).

## Como usar ?

### Subindo servidor keycloak:

Para subir o servidor do keycloak, uso o docker-compose que está na raiz
do projeto. Com isso execute o seguinte comando em um terminal de sua escolha:

```shell
docker-compose up
```

Ele vai levar um tempo para realizar a configuração inicial, sendo assim, 
aguarde até aparacer uma mensagem como a seguinte:

```shell
[org.jboss.as] (Controller Boot Thread) WFLYSRV0051: Admin console listening on http://127.0.0.1:9990
```

### Acessando e Configurando o keycloak:

Para acessar o keycloak que subiu em sua máquina, acesse o seguinte endereço:

[http://localhost:8080/auth/admin/](http://localhost:8080/auth/admin/)

Será aberto uma tela de login, use as seguintes credenciais para o acesso:

| Username or email | Password |
| ----------------- | -------- |
| admin             | admin    |

Nesse ponto já teremos configurado os seguintes pontos:

* **Demo-Realm** - O realm é o espaço onde temos o grupo de todas as configurações de segurança usadas na aplicação.
* **Client Springboot-microservice** - Clients são as determinações de um aplicação. Nesse client já temos configurado 
a url de acesso a aplicação, o tipo de acesso como confidencial e a associação com as regras admin e user.
* **Roles app-admin e app-user** - São como permissões de aplicação, esses dois já configurados tem associação com as
regras do client springboot-microservice.

Nesse ponto é necessário realizar o cadastro dos usuários, que podem ter o acesso ao client, para assim podermos
gerar tokens. Nesse exemplo precisamos de 3 usuários diferentes que iram ter vínculos com regras diferentes.
Para isso acesse o seguinte endereço: 
[http://localhost:8080/auth/admin/master/console/#/realms/Demo-Realm/users](http://localhost:8080/auth/admin/master/console/#/realms/Demo-Realm/users)
ou navegue pelo menu Users no console do keycloack.

Nessa tela clique na opção de Add user e será necessário colocar um Username e deixar
"ON" a opção **Email Verified**. Como criaremos 3, coloque os seguintes nomes: employee1, employee2 e employee3.
Para todos os usuários iremos definir a senha **mypassword**, para isso acesse o usuário criado e vá na opção
de **Credentials**, adicione a senha e confirme, deixe "OFF" a opção **Temporary** e clique em **Set Password**.
Também é necessário fazer o vínculo com as regras de aplicação. Para isso acesse a opção de **Role Mappings** no 
usuário, e faremos a seguinte vinculação:

| Usuário | Roles |
| ------- | ----- |
| employee1 | app-user |
| employee2 | app-admin |
| employee3 | app-user e app-admin |

### Geração de Token:

Temos duas formas de gerar um token a primeira seria com o grant_type **password**, que pode ser usado assim:

**Endpoint:** http://localhost:8080/auth/realms/{realm}/protocol/openid-connect/token
(no nosso caso http://localhost:8080/auth/realms/Demo-Realm/protocol/openid-connect/token)

**Body:**

| Campo | Valor |
| ----- | ----- |
| grant_type | password |
| client_id | springboot-microservice |
| client_secret | Nesse caso é necessário gerar um novo secret, para isso acesse o client **springboot-microservice** e vá até a opção **Credentials** e clique no botão **Regenerate Secret**. |
| username | pode ser employee1 ou employee2 ou employee3 |
| password | mypassword |

**Response**

| Campo | Tipo | Descrição |
| ----- | ---- | --------- |
| access_token | String | Token que será usado para a autorização. |
| expires_in | int | Tempo de expiração do token. |
| refresh_expires_in | int | Tempo de expiração do token de refresh. |
| refresh_token | String | Token usado para atualizar o token de acesso. |
| token_type | String | Tipo de token, em geral é Bearer. |
| not-before-policy | int | Politica de geração de token. |
| session_state | String | Estado da sessão do token. |
| scope | String | Escopo do client, caso não definido será "email profile". |

A segunda opção é gerar com **refresh_token**. Tanto o endpoint quanto o response são os mesmos que a opção anterior, 
apenas o Body que é diferente, assim só será descrito o mesmo.

**Body:**

| Campo | Valor |
| ----- | ----- |
| grant_type | refresh_token |
| client_id | springboot-microservice |
| client_secret | Nesse caso use o mesmo processo do anterior caso não tenha o secret. |
| refresh_token | Token refresh que é gerado na primeira chamada. |

### Endpoints da aplicação

Nessa aplicação temos 4 tipos de endpoints, que são apenas exemplos de como é usado
os tokens gerados pelo keycloak.

| Endpoint | Header | Role | Usuários que podem acessar |
| -------- | ------ | ---- | -------------------------- |
| http://localhost:8180/test/anonymous | Não | Nenhuma | Todos |
| http://localhost:8180/test/user | Authorization do tipo Bearer com o Token | user | employer1 e employee3 |
| http://localhost:8180/test/admin | Authorization do tipo Bearer com o Token | admin | employer2 e employee3 |
| http://localhost:8180/test/all-user | Authorization do tipo Bearer com o Token | admin e user | Todos |

## Bibliografia

* [https://medium.com/devops-dudes/securing-spring-boot-rest-apis-with-keycloak-1d760b2004e](https://medium.com/devops-dudes/securing-spring-boot-rest-apis-with-keycloak-1d760b2004e)
* [https://www.keycloak.org/docs/latest/getting_started/](https://www.keycloak.org/docs/latest/getting_started/)
* [https://medium.com/swlh/stateless-jwt-authentication-with-spring-boot-a-better-approach-1f5dbae6c30f](https://medium.com/swlh/stateless-jwt-authentication-with-spring-boot-a-better-approach-1f5dbae6c30f)
* [https://www.baeldung.com/spring-security-oauth-jwt](https://www.baeldung.com/spring-security-oauth-jwt)
* [https://www.toptal.com/spring/spring-boot-oauth2-jwt-rest-protection](https://www.toptal.com/spring/spring-boot-oauth2-jwt-rest-protection)