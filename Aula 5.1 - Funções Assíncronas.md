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
const minhaPromise = new Promise<string>((resolve, reject) => {
  const sucesso = true; // Simula sucesso ou falha

  if (sucesso) {
    resolve('Operação concluída com sucesso!');
  } else {
    reject('Houve um erro na operação.');
  }
});

minhaPromise
  .then(resultado => console.log(resultado))
  .catch(erro => console.error(erro));
````

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

* Evita que o servidor fique bloqueado esperando operações demoradas, como consultas ao banco.
* O código fica mais limpo e sequencial com `async`/`await`.
* Facilita o tratamento de erros com `try/catch`.

---

## 6. Tratamento de Erros com Try/Catch

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


Se quiser, posso ajudar a preparar exercícios que misturam Promises, async/await e consultas ao banco para fixação!
```
