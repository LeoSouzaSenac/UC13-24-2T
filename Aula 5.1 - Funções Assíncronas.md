# Funções Assíncronas

## Introdução

No desenvolvimento de APIs com Node.js e Express, operações que acessam bancos de dados geralmente são assíncronas, pois podem demorar para serem concluídas. Para lidar com esse tipo de operação sem bloquear o servidor, usamos funções assíncronas em JavaScript/TypeScript.

Neste documento, vamos entender como funcionam as funções assíncronas e as Promises, e como usá-las em um contexto real de uma API que acessa um banco de dados.

## 1. O Que São Promises?

Uma **Promise** é um objeto que representa a eventual conclusão ou falha de uma operação assíncrona. Pense nela como uma "promessa" de que, em algum momento, o resultado estará disponível (ou um erro ocorrerá).

### Estados de uma Promise

- **Pending (Pendente)**: A operação ainda não terminou.
- **Fulfilled (Cumprida)**: A operação terminou com sucesso e retornou um valor.
- **Rejected (Rejeitada)**: A operação falhou e retornou um erro.

### Exemplo de Promise simples

```typescript
/*
Dentro do construtor de uma Promise, você passa uma função que recebe dois parâmetros: resolve e reject.

Eles são funções, e cada uma tem um papel:

resolve: chama quando a operação assíncrona termina com sucesso. Ela "resolve" a Promise, entregando um valor para quem está esperando.

reject: chama quando a operação falha. Ela "rejeita" a Promise, entregando um erro para quem está esperando.
*/

const minhaPromise = new Promise<string>((resolve, reject) => {
  const sucesso = true; // Simula sucesso ou falha

  if (sucesso) {
    resolve('Operação concluída com sucesso!');
  } else {
    reject('Houve um erro na operação.');
  }
});
/*
As funções .then() e .catch() são métodos que você usa para lidar com o resultado dessa Promise.
.then() é chamado quando a Promise é resolvida com sucesso. O que estiver dentro do .then() vai receber o resultado passado para o resolve.
.catch() é chamado quando a Promise é rejeitada (quando alguém chama reject(erro)). O que estiver dentro do .catch() vai receber o erro passado para o reject.
*/
minhaPromise
  .then(resultado => console.log(resultado))
  .catch(erro => console.error(erro));
````
---

No entanto, normalmente, você não precisa criar Promises manualmente com new Promise(...) — isso é mais para casos específicos.
Na maior parte do tempo, funções que fazem chamadas assíncronas (como consultar banco de dados, ler arquivos, fazer requisições HTTP) já retornam Promises prontas para usar. Então, agora que entendemos o que são promises, vamos ao que interessa.
---

## 2. O Que São Funções Assíncronas?

Funções assíncronas são declaradas com a palavra-chave `async` e permitem usar a palavra-chave `await` para esperar a conclusão de Promises, simplificando o código assíncrono.

### Exemplo de Declaração

```typescript
async function exemplo(): Promise<void> {
  // código assíncrono aqui
}
```

Quando chamamos uma função `async`, ela retorna uma Promise.

---

## 3. Exemplo Prático: Listar Usuários

Veja este método de um controller que lista usuários no banco de dados. A consulta ao banco retorna uma Promise, e usamos `await` para esperar ela ser concluída.

```typescript
import { Request, Response } from 'express';
import { connection } from '../config/database';

export class UserController {
  async listUsers(req: Request, res: Response): Promise<Response> {
    const [rows] = await connection.query('SELECT * FROM usuarios');
    return res.status(200).json(rows);
  }
}
```

---

## 4. Exemplo Prático: Criar Usuário

Outro método para criar um usuário no banco, usando `await` para esperar a inserção no banco ser concluída:

```typescript
async createUser(req: Request, res: Response): Promise<Response> {
  const { nome, email } = req.body;

  if (!nome || !email) {
    return res.status(400).json({ mensagem: 'Nome e e-mail obrigatórios.' });
  }

  await connection.query('INSERT INTO usuarios (nome, email) VALUES (?, ?)', [nome, email]);

  return res.status(201).json({ mensagem: 'Usuário criado com sucesso!' });
}
```

---

## 5. Por Que Usar Funções Assíncronas?

* Evita que o servidor fique bloqueado esperando operações demoradas, como consultas ao banco. Ou seja, se o Node estivesse executando uma operação síncrona e demorada (por exemplo, uma consulta ao banco que demora 5 segundos), ele ficaria parado, esperando essa operação terminar, sem poder atender a outras requisições enquanto isso. Imagina assim:
 - Operação síncrona (bloqueante) é como se você fizesse uma ligação telefônica e, até a outra pessoa atender e a conversa acabar, você não pode fazer mais nada — está “preso” na ligação.
 - Operação assíncrona (não bloqueante) é como se você fizesse a ligação e, enquanto espera a pessoa atender, você pudesse fazer outras coisas (mandar mensagens, atender outras ligações, etc.). Quando a pessoa atender, você volta para a conversa.

* O código fica mais limpo e sequencial com `async`/`await`.
* Facilita o tratamento de erros com `try/catch`.

---

## 6. Tratamento de Erros com Try/Catch

### Por que usar `try/catch` sempre dentro de funções `async`?

Quando usamos `await` para esperar o resultado de uma Promise, essa Promise pode ser **resolvida com sucesso** ou **rejeitada com um erro**.

Se a Promise for rejeitada (por exemplo, erro ao consultar o banco, problema de conexão, dados inválidos), o erro **é lançado como uma exceção dentro da função async**.

---

### O que acontece se não usar `try/catch`?

* O erro **não será capturado dentro da função**, e a execução da função será interrompida abruptamente.
* O servidor pode não conseguir responder adequadamente à requisição, o que pode levar a **respostas incompletas**, **falhas silenciosas** ou até mesmo o **travamento da aplicação**.
* É difícil controlar e informar o que deu errado para o cliente.

---

### Por que `try/catch` resolve isso?

* O bloco `try` envolve o código que pode gerar erros.
* Se ocorrer algum erro, ele é capturado pelo `catch`.
* Dentro do `catch`, você pode:

  * Registrar o erro para ajudar no diagnóstico (ex: `console.error`).
  * Enviar uma resposta de erro adequada para o cliente (ex: status 500 e mensagem amigável).

---

### Em resumo:

* **`await` pode lançar erros.**
* **`try/catch` é a forma de capturar esses erros.**
* Usar `try/catch` **garante que sua API responda sempre**, mesmo em casos de falhas inesperadas.



```typescript
async listUsers(req: Request, res: Response): Promise<Response> {
  try {
    const [rows] = await connection.query('SELECT * FROM usuarios');
    return res.status(200).json(rows);
  } catch (error) {
    console.error('Erro ao listar usuários:', error);
    return res.status(500).json({ mensagem: 'Erro interno no servidor' });
  }
}
```

---

## 7. Conclusão

Promises e funções assíncronas são ferramentas essenciais para lidar com operações que podem levar tempo no desenvolvimento de APIs. Elas permitem que o servidor responda a múltiplas requisições de forma eficiente e com código mais fácil de entender.
