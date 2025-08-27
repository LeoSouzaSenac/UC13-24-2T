# O que é CORS e para que serve

## Introdução
CORS é a sigla para **Cross-Origin Resource Sharing** (Compartilhamento de Recursos entre Origens).  
É um mecanismo de segurança implementado pelos navegadores para **controlar quais domínios podem acessar recursos de um servidor**.

---

## Por que o CORS existe?
Imagine que você está em um site qualquer e ele tenta, sem você saber, buscar dados de outro sistema no qual você já está logado (como seu banco ou e-mail).  
Se não existisse CORS, isso seria uma **falha de segurança grave**, permitindo roubo de dados sensíveis.

Para evitar esse risco, o navegador **bloqueia requisições vindas de origens diferentes** (diferente domínio, porta ou protocolo), a menos que o servidor autorize explicitamente.

---

## Como o CORS funciona
Quando uma aplicação cliente (ex: uma página HTML ou uma aplicação React) tenta acessar uma API em outro domínio, o navegador envia um **pedido especial chamado *preflight*** (usando `OPTIONS`).  
Esse pedido pergunta ao servidor:  
*"Você permite que este domínio faça requisições para você?"*

O servidor pode responder com cabeçalhos HTTP, como por exemplo:

```http
Access-Control-Allow-Origin: https://meusite.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
````

Esses cabeçalhos dizem ao navegador que é seguro permitir a requisição.

---

## Situações comuns onde o CORS aparece

* Você está rodando o **frontend em `http://localhost:3000`** e o **backend em `http://localhost:5000`**.
  O navegador entende que são origens diferentes e exige CORS.
* Uma aplicação hospedada em `https://meusite.com` tenta consumir uma API em `https://api.outrasite.com`.

---

## Como resolver problemas de CORS

1. **No backend**: habilitar o CORS para os domínios que precisam acessar a API.
   Exemplo em **Express (Node.js)**:

   ```ts
   import express from "express";
   import cors from "cors";

   const app = express();
   app.use(cors({ origin: "https://meusite.com" }));
   ```

   Se quiser liberar para qualquer origem (apenas em desenvolvimento):

   ```ts
   app.use(cors());
   ```

2. **No frontend**: não há como "burlar" o CORS, pois ele é uma proteção do navegador.
   O único jeito correto é configurar o backend para aceitar as requisições.

---

## Resumindo

* O **CORS é uma proteção de segurança do navegador**.
* Ele evita que sites maliciosos consumam dados de outros sistemas sem autorização.
* Para o frontend funcionar corretamente, o **backend deve responder com cabeçalhos de CORS** liberando os domínios autorizados.
* Em desenvolvimento, normalmente usamos `cors()` liberado para todos, mas em produção é importante restringir para o domínio do seu site.

---

