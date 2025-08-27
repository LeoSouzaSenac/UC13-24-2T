# 📌 Como Consumir APIs com o Frontend em HTML + JavaScript

Este guia explica, de forma simples, como fazer o **frontend em HTML** se comunicar com uma **API** usando **JavaScript**.  
Vamos utilizar o método **`fetch`**, que é nativo dos navegadores modernos.

---

## 🔹 O que é uma API?

API significa **Application Programming Interface**.  
No contexto da web, uma API expõe **endpoints** (endereços/rotas) que permitem que o frontend busque ou envie informações para o backend (servidor).

Exemplo de endpoint de API:
```

[https://meuservidor.com/api/users](https://meuservidor.com/api/users)

```

---

## 🔹 Estrutura Básica do Projeto

📁 Estrutura de pastas:
```

meu-projeto/
│── index.html
│── js/
│    └── index.js
│── css/
└── index.css

````

---

## 🔹 Criando o Formulário (HTML)

O arquivo `index.html` terá um formulário simples para cadastrar usuários:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Cadastro de Usuário</title>
  <link rel="stylesheet" href="css/style.css">
</head>
<body>
  <h2>Cadastro de Usuário</h2>
  <form id="formCadastro">
    <label for="nome">Nome:</label><br>
    <input type="text" id="nome" name="nome" required><br><br>

    <label for="email">E-mail:</label><br>
    <input type="email" id="email" name="email" required><br><br>

    <label for="senha">Senha:</label><br>
    <input type="password" id="senha" name="senha" required><br><br>

    <button type="submit">Cadastrar</button>
  </form>

  <p id="mensagem"></p>

  <script src="js/index.js"></script>
</body>
</html>
````

---

## 🔹 Consumindo a API com JavaScript (Fetch)

No arquivo `js/index.js`, vamos usar **`fetch`** para enviar os dados do formulário para a API:

```javascript
// Seleciona o formulário e adiciona um "ouvinte" para o evento de submit
document.getElementById("formCadastro").addEventListener("submit", async function(event) {
  event.preventDefault(); // Impede o recarregamento da página

  // Captura os valores do formulário
  const name = document.getElementById("nome").value;
  const email = document.getElementById("email").value;
  const password = document.getElementById("senha").value;

  try {
    // Faz a requisição para a API
    const resposta = await fetch("http://localhost:3000/auth/register", {
      method: "POST", // Tipo da requisição
      headers: {
        "Content-Type": "application/json" // Informa que está enviando JSON
      },
      body: JSON.stringify({ name, email, password }) // Converte os dados em JSON
    });

    // Se a resposta não for bem-sucedida, lança um erro
    if (!resposta.ok) {
      const erro = await resposta.text();
      throw new Error(erro);
    }

    // Converte a resposta da API em objeto JavaScript
    const dados = await resposta.json();

    // Mostra mensagem de sucesso
    document.getElementById("mensagem").textContent = "✅ Usuário cadastrado com sucesso!";
    document.getElementById("mensagem").style.color = "green";

  } catch (erro) {
    // Mostra mensagem de erro
    document.getElementById("mensagem").textContent = "❌ Erro: " + erro.message;
    document.getElementById("mensagem").style.color = "red";
  }
});
```

---

## 🔹 Fluxo de Funcionamento

1. Usuário preenche o formulário no **HTML**.
2. O **JavaScript** captura os dados e envia para a API com **`fetch`**.
3. A API processa e retorna uma resposta (sucesso ou erro).
4. O frontend mostra a resposta na tela.

---



## ✅ Conclusão

* O **HTML** cria a interface.
* O **JavaScript** usa `fetch` para se comunicar com a API.
* O **backend** processa a requisição e retorna uma resposta.

Com isso, já conseguimos **consumir APIs no frontend** usando apenas HTML + JavaScript. 
