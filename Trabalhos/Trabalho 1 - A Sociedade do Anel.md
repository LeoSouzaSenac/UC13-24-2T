## 🧙‍♂️ **Trabalho Avaliativo — O Chamado da Sociedade do Anel**

"Três Anéis para os Reis-Elfos sob este céu,
Sete para os Senhores-Anões em seus rochosos corredores,
Nove para os Homens Mortais fadados ao eterno sono,
Um para o Senhor do Escuro em seu escuro trono
Na Terra de Mordor onde as Sombras se deitam.
Um Anel para a todos governar, Um Anel para encontrá-los,
Um Anel para a todos trazer e na escuridão aprisioná-los
Na Terra de Mordor onde as Sombras se deitam."


> \*\*"Vocês são todos convocados, jovens aprendizes de código.
> A ameaça de Sauron cresce a cada dia, e precisamos de um sistema para registrar todos que caminham pela Terra Média.
> Com esse registro, saberemos quem pode cruzar a Ponte de Khazad-dûm… e quem deve ser detido a todo custo.
>
> Eu, Gandalf, o Cinzento,s ervo do Fogo Secreto, portador da chama de Anor confio a vocês esta missão."\*\*


> "Mas Gandalf, eu não queria ter que fazer esse trabalho." - Frodo Bolseiro
> "Não podemos escolher os os trabalhos que fazemos. Só podemos decidir que gambiarra fazer com o tempo que nos é dado. Que no caso, é até sexta-feira."
---

### **Sua tarefa**

Criar um **sistema de CRUD** para personagens da Terra Média, usando:

* **TypeScript**
* **Express**
* **MySQL2/promise**
* Estrutura **MVC** (sem view)
* Testes de rotas no **Thunderclient**

---

### **O Banco de Dados**

Crie a tabela `personagens` com os seguintes campos:

* `id` → número, auto incremento
* `nome` → texto
* `tipo` → texto (`Sociedade`, `Nazgûl` ou `Balrog`)
* `raca` → texto (`Hobbit`, `Humano`, `Elfo`, `Anão`, `Mago`, `Criatura`, etc.)
* `arma` → texto
* `status` → texto (`vivo`, `ferido`, `morto`)

---

### **Os Middlewares da Jornada**

#### 1️⃣ **Chamado do Anel**

> *"Sempre que um Nazgûl surge… Frodo sente."*

* Antes de qualquer **POST**, **PUT** ou **DELETE** envolvendo um personagem do tipo `"Nazgûl"`, mostre no console:

```
Frodo sente o Um Anel querendo retornar ao seu Mestre...
```

---

#### 2️⃣ **A Ponte de Khazad-dûm**

> *"Nem todos têm passagem livre pela ponte de Moria."*

* Em qualquer **GET** (listar ou buscar), mostre no console:

  * Se for `"Sociedade"` →
    `Pode passar pela Ponte de Khazad-dûm.`
  * Se for `"Nazgûl"` →
    `Os Nazgûl não estão em Moria.`
  * Se for `"Balrog"` →
    `Você não vai passar!`

---

#### 3️⃣ **A Rota Perdida**

> *"Nem todos os caminhos levam a Mordor."*

* Se o usuário acessar uma rota inexistente, retorne:

```json
{ "erro": "A passagem de Caradhras está fechada por Saruman. Esta rota não existe para nós. Só nos sobrou...Moria." }
```

---

### **Regras do Conselho de Elrond**

* O middleware do Chamado do Anel **só** roda se o personagem for `"Nazgûl"`.
* O middleware da Ponte **só** roda em requisições GET (Dica: use `if (req.method === "GET")` para isso)
* O middleware de rota inexistente deve ser o **último** no `server.ts`.


---

### **Exemplo de uso**

#### Criando um Nazgûl

```
POST /personagens
{
  "nome": "Rei Bruxo de Angmar",
  "tipo": "Nazgûl",
  "raca": "Humano corrompido",
  "arma": "Espada Morgul",
  "status": "vivo"
}
```

**Console:**

```
Frodo sente o Um Anel chamando por seu Mestre...
```

---

#### Buscando o Balrog

```
GET /personagens/12
```

**Console:**

```
Você não vai passar!
```

---

#### Buscando um membro da Sociedade

```
GET /personagens/1
```

**Console:**

```
Pode passar pela Ponte de Khazad-dûm.
```

---

#### Buscando um Nazgûl

```
GET /personagens/4
```

**Console:**

```
Os Nazgûl não estão em Moria.
```

---

> **"Façam o sistema com zelo, jovens codificadores… pois um erro poderá custar o destino de toda a Terra Média."**
> — *Gandalf, O Cinzento*
