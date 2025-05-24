<a name="inicio"></a>
# [Maus Cheiros de Código em PowerScript][Catalogo PowerScript]

Este catálogo apresenta e descreve os principais maus cheiros de código identificados em projetos desenvolvidos com a linguagem PowerScript. Seu objetivo é auxiliar desenvolvedores na detecção de padrões de código que podem indicar problemas de design, manutenção ou legibilidade, promovendo assim práticas mais eficientes e sustentáveis de desenvolvimento.

## Sumário

1. [Long Parameter List](https://github.com/joaomello03/catalogo/blob/main/README.md#long-parameter-list)
2. [Long Function](https://github.com/joaomello03/catalogo/blob/main/README.md#long-function)
3. [Duplicated Code](https://github.com/joaomello03/catalogo/blob/main/README.md#duplicated-code)
4. [Large Class](https://github.com/joaomello03/catalogo/blob/main/README.md#large-class)
5. [Feature Envy](https://github.com/joaomello03/catalogo/blob/main/README.md#feature-envy)
6. [Message Chains](https://github.com/joaomello03/catalogo/blob/main/README.md#message-chains)
7. [Shotgun Surgery](https://github.com/joaomello03/catalogo/blob/main/README.md#shotgun-surgery)
8. [Primitive Obsession](https://github.com/joaomello03/catalogo/blob/main/README.md#primitive-obsession)
9. [Data Class](https://github.com/joaomello03/catalogo/blob/main/README.md#data-class)
10. [Repeated Switches](https://github.com/joaomello03/catalogo/blob/main/README.md#repeated-switches)

---

<a name="LongParameterList"></a>
## Long Parameter List

Esse mau cheiro ocorre quando um método ou função recebe muitos parâmetros. Métodos com longas listas de parâmetros são difíceis de entender, propensos a erros (como a troca de ordem dos argumentos) e tornam a manutenção do código mais trabalhosa.
Em PowerScript, que tem forte foco em manipulação de dados e eventos, esse problema pode surgir facilmente.

### 🧠 Problemas causados

- Dificuldade de leitura e compreensão do método.
- Risco de erro ao passar argumentos incorretos ou na ordem errada.
- Dificuldade para fazer alterações futuras (adicionar, remover ou reorganizar parâmetros).
- Redução da coesão e aumento da dependência entre partes do sistema.

### 🛠️ Solução/Refatoração Recomendada

Aplicar a refatoração **Introduce Parameter Object**, onde cria-se uma estrutura (em PowerScript, uma _structure_ ou _class_) que agrupa os parâmetros relacionados.
Dessa forma, o método recebe apenas um objeto, melhorando a clareza e a robustez do código.

### 🔎 Exemplo de Código com Long Parameter List

```pascal
function decimal of_calcular_valor(decimal preco, decimal desconto, integer quantidade, string tipoProduto, string metodoPagamento, decimal taxaAdicional)
    decimal valor_final

    valor_final = (preco - desconto) * quantidade

    if lower(tipoProduto) = "promocional" then
        valor_final = valor_final * 0.9
    end if

    if lower(metodoPagamento) = "pix" then
        valor_final = valor_final * 0.97
    end if

    valor_final += taxaAdicional

    return valor_final
end function
```

Problemas no exemplo acima:
- Seis parâmetros distintos, o que aumenta a chance de erro.
- Difícil de entender rapidamente qual conjunto de dados o método manipula.

### ✨ Exemplo de Refatoração Aplicando Introduce Parameter Object

1. Criar uma estrutura para agrupar os dados
```pascal
global type info_produto from structure
    decimal preco
    decimal desconto
    integer quantidade
    string tipoProduto
    string metodoPagamento
    decimal taxaAdicional
end type
```

2. Refatorar o método para receber a estrutura
```pascal
// Método refatorado
function decimal of_calcular_valor(info_produto produto)
    decimal valor_final

    valor_final = (produto.preco - produto.desconto) * produto.quantidade

    if lower(produto.tipoProduto) = "promocional" then
        valor_final = valor_final * 0.9
    end if

    if lower(produto.metodoPagamento) = "pix" then
        valor_final = valor_final * 0.97
    end if

    valor_final += produto.taxaAdicional

    return valor_final
end function
```

3. Exemplo de chamada do método
```pascal
Info_Produto produto

produto.preco = 100
produto.desconto = 10
produto.quantidade = 2
produto.tipoProduto = "Promocional"
produto.metodoPagamento = "PIX"
produto.taxaAdicional = 1

decimal valor = of_calcular_valor(produto)
```

### 📈 Benefícios da Refatoração

- Redução no número de parâmetros, facilitando a chamada do método.
- Melhor organização dos dados relacionados.
- Facilita a manutenção e a extensão do sistema (adicionar novos atributos ao _Info_Produto_, por exemplo).
- Torna o código mais legível e menos propenso a erros.

[Voltar ao início](#sumário)

---

<a name="LongFunction"></a>
## Long Function

Esse mau cheiro ocorre quando uma função ou método realiza tarefas demais e possui muitas linhas de código. Em PowerScript, esse problema é muito comum em eventos de janelas (como _clicked, constructor, open_) ou funções que realizam múltiplas etapas de lógica de negócio e interface em um único bloco.
Funções longas tendem a misturar níveis de abstração, como acesso a banco de dados, validações, lógica de interface e cálculos em um só lugar, dificultando a leitura, testes e manutenção.

### 🧠 Problemas causados

- Redução da legibilidade: difícil de entender o que a função realmente faz.
- Dificuldade de reutilização: partes úteis do código ficam "presas" dentro da função.
- Aumento do risco de erro: pequenas alterações em um ponto podem afetar outras partes.
- Testabilidade comprometida: difícil testar partes específicas do comportamento.
- Viola o princípio da responsabilidade única (SRP – _Single Responsibility Principle_).

### 🛠️ Solução/Refatoração Recomendada

A técnica mais apropriada para esse mau cheiro é a **Extract Method**:
- Consiste em extrair blocos de código logicamente coesos para métodos auxiliares com nomes claros e descritivos.

Isso reduz o tamanho da função original, melhora a modularidade e facilita a leitura e o reuso.
Em PowerScript, pode-se criar _functions, object functions_, ou _user events_ para separar responsabilidades.

### 🔎 Exemplo de Código com Long Function

```pascal
// Evento clicked de um botão com lógica excessiva
subroutine clicked()
    string nome
    integer idade
    decimal salario
    decimal imposto

    // Captura de dados
    nome = sle_nome.text
    idade = integer(sle_idade.text)
    salario = decimal(sle_salario.text)

    // Validação
    if nome = "" or idade <= 0 or salario <= 0 then
        messagebox("Erro", "Dados inválidos.")
        return
    end if

    // Cálculo de imposto
    if salario > 5000 then
        imposto = salario * 0.2
    else
        imposto = salario * 0.1
    end if

    // Atualização do display
    sle_resultado.text = "Imposto: " + string(imposto)

    // Grava no banco
    insert into funcionario
        (nome, idade, salario, imposto)
    values
        (:nome, :idade, :salario, :imposto)
end subroutine
```

Função com múltiplas responsabilidades: captura de dados, validação, cálculo, atualização de UI e acesso ao banco de dados — tudo junto.

### ✨ Exemplo de Refatoração Aplicando Extract Method

```pascal
subroutine clicked()
    string nome
    integer idade
    decimal salario

    if not of_captura_dados(nome, idade, salario) then return
    if not of_valida_dados(nome, idade, salario) then return

    decimal imposto = of_calcula_imposto(salario)

    of_exibe_resultado(imposto)
    of_salva_dados(nome, idade, salario, imposto)
end subroutine

// Métodos auxiliares
boolean function of_captura_dados(ref string nome, ref integer idade, ref decimal salario)
    nome = sle_nome.text
    idade = integer(sle_idade.text)
    salario = decimal(sle_salario.text)

    return true
end function

boolean function of_valida_dados(string nome, integer idade, decimal salario)
    if nome = "" or idade <= 0 or salario <= 0 then
        messagebox("Erro", "Dados inválidos.")
        return false
    end if

    return true
end function

decimal function of_calcula_imposto(decimal salario)
    if salario > 5000 then
        return salario * 0.2
    else
        return salario * 0.1
    end if
end function

subroutine of_exibe_resultado(decimal imposto)
    sle_resultado.text = "Imposto: " + string(imposto)
end subroutine

subroutine of_salva_dados(string nome, integer idade, decimal salario, decimal imposto)
    insert into funcionario
        (nome, idade, salario, imposto)
    values
        (:nome, :idade, :salario, :imposto)
end subroutine
```

Cada responsabilidade agora está separada em funções coesas e reutilizáveis. O evento _clicked()_ virou apenas um orquestrador claro e legível.

### 📈 Benefícios da Refatoração

- Leitura facilitada: agora a função principal descreve claramente o fluxo do processo.
- Responsabilidades isoladas: cada parte da lógica está encapsulada.
- Melhor manutenção: é mais fácil corrigir ou estender partes específicas.
- Reuso de lógica: funções como _of_calcula_imposto_ podem ser reaproveitadas em outros lugares.
- Testabilidade aumentada: agora é possível escrever testes para funções menores isoladamente.

[Voltar ao início](#sumário)

---

<a name="duplicated-code"></a>
## Duplicated Code

Esse mau cheiro ocorre quando blocos de código idênticos ou muito semelhantes aparecem em múltiplos lugares. Isso é comum em sistemas legados PowerScript, onde lógicas semelhantes são copiadas entre janelas, scripts ou funções.

### 🧠 Problemas causados

- Dificulta manutenção: alterações precisam ser replicadas manualmente.
- Aumenta o risco de inconsistências.
- Redundância desnecessária.

### 🛠️ Solução/Refatoração Recomendada

**Extract Method** – crie uma função reutilizável.

### 🔎 Exemplo com Duplicated Code

```pascal
decimal total1 = (preco - desconto) * quantidade
total1 += taxa

decimal total2 = (preco - desconto) * quantidade
total2 += taxa
```

### ✨ Exemplo Refatorado

```pascal
function decimal of_calcula_total(decimal preco, decimal desconto, integer quantidade, decimal taxa)
    return ((preco - desconto) * quantidade) + taxa
end function

decimal total1 = of_calcula_total(preco, desconto, quantidade, taxa)
decimal total2 = of_calcula_total(preco, desconto, quantidade, taxa)
```

### 📈 Benefícios da Refatoração

- Centraliza a lógica.
- Facilita manutenção.
- Reduz erros.

[Voltar ao início](#sumário)

---

<a name="large-class"></a>
## Large Class

Classes com responsabilidades demais, comum em janelas PowerScript com lógica de UI, banco de dados e regras de negócio misturadas.

### 🧠 Problemas causados

- Viola o princípio da responsabilidade única (SRP – _Single Responsibility Principle_).
- Código difícil de entender, testar e manter.

### 🛠️ Solução/Refatoração Recomendada

**Extract Class** – mover responsabilidades para objetos auxiliares.

### 🔎 Exemplo com Large Class

```pascal
window w_funcionario
    string nome
    integer idade
    decimal salario
    subroutine calcular_salario()
    subroutine validar_campos()
    subroutine salvar_dados()
end window
```

### ✨ Exemplo Refatorado

```pascal
nonvisualobject nvo_funcionario
    string nome
    integer idade
    decimal salario
    function boolean validar()
    function decimal calcular_salario()
end object

window w_funcionario
    nvo_funcionario funcionario
end window
```

### 📈 Benefícios da Refatoração

- Redução da complexidade.
- Código mais organizado e testável.

[Voltar ao início](#sumário)

---

<a name="feature-envy"></a>
## Feature Envy

Métodos que utilizam mais dados de outro objeto do que da própria classe.

### 🧠 Problemas causados

- Aumenta o acoplamento.
- Reduz a coesão.

### 🛠️ Solução/Refatoração Recomendada

**Move Method** – mover método para a classe que contém os dados.

### 🔎 Exemplo com Feature Envy

```pascal
function decimal of_calcular_bonus(empregado e)
    if e.salario > 5000 then
        return e.salario * 0.1
    else
        return e.salario * 0.05
    end if
end function
```

### ✨ Exemplo Refatorado

```pascal
function decimal of_calcular_bonus()
    if this.salario > 5000 then
        return this.salario * 0.1
    else
        return this.salario * 0.05
    end if
end function
```

### 📈 Benefícios da Refatoração

- Reduz dependência externa.
- Aumenta coesão e encapsulamento.

[Voltar ao início](#sumário)

---

<a name="message-chains"></a>
## Message Chains

Cadeia longa de chamadas entre objetos (ex: `a().b().c()`).

### 🧠 Problemas causados

- Alta fragilidade a mudanças internas.
- Código difícil de entender.

### 🛠️ Solução/Refatoração Recomendada

**Hide Delegate** – encapsular a cadeia em um método intermediário.

### 🔎 Exemplo com Message Chains

```pascal
string cidade = this.of_get_empregado().of_get_endereco().of_get_cidade()
```

### ✨ Exemplo Refatorado

```pascal
function string of_get_cidade_empregado()
    return this.empregado.of_get_endereco().of_get_cidade()
end function
```

### 📈 Benefícios da Refatoração

- Reduz acoplamento.
- Facilita refatorações futuras.

[Voltar ao início](#sumário)

---

<a name="shotgun-surgery"></a>
## Shotgun Surgery

Uma única mudança requer alterações em vários locais.

### 🧠 Problemas causados

- Aumenta o risco de erros.
- Baixa coesão.

### 🛠️ Solução/Refatoração Recomendada

**Move Method** ou **Extract Method** – centralizar lógica repetida.

### 🔎 Exemplo com Shotgun Surgery

```pascal
if metodo_pagamento = "PIX" then
    desconto = valor * 0.05
end if
```

### ✨ Exemplo Refatorado

```pascal
function decimal of_calcular_desconto(string metodo, decimal valor)
    if lower(metodo) = "pix" then
        return valor * 0.05
    end if
    return 0
end function
```

### 📈 Benefícios da Refatoração

- Centralização da lógica.
- Redução da duplicação.

[Voltar ao início](#sumário)

---

<a name="primitive-obsession"></a>
## Primitive Obsession

Uso excessivo de tipos primitivos ao invés de abstrações.

### 🧠 Problemas causados

- Repetição de validações.
- Redução da expressividade.

### 🛠️ Solução/Refatoração Recomendada

**Introduce Value Object** – agrupar dados relacionados em estruturas.

### 🔎 Exemplo com Primitive Obsession

```pascal
function boolean of_verificar_cliente(string nome, string cpf, string telefone)
```

### ✨ Exemplo Refatorado

```pascal
global type cliente_info from structure
    string nome
    string cpf
    string telefone
end type

function boolean of_verificar_cliente(cliente_info cli)
```

### 📈 Benefícios da Refatoração

- Código mais legível e reutilizável.
- Parâmetros agrupados com significado.

[Voltar ao início](#sumário)

---

<a name="data-class"></a>
## Data Class

Esse mau cheiro ocorre quando um objeto existe apenas para armazenar dados, sem conter nenhum comportamento associado. Em PowerScript, é comum vermos objetos que apenas agrupam campos, enquanto toda a lógica associada fica espalhada por outras partes do código.

### 🧠 Problemas causados

- Falta de encapsulamento: os dados ficam expostos e são manipulados livremente fora do objeto.
- Lógica dispersa: operações sobre os dados ficam espalhadas pelo sistema.
- Baixa coesão: o objeto não representa uma unidade funcional.
- Dificuldade de manutenção e evolução: alterações nas regras exigem buscas por todo o sistema.

### 🛠️ Solução/Refatoração Recomendada

Aplicar a refatoração **Replace Data Value with Object**, que consistie em substitur estruturas puramente informacionais por objetos que encapsulam dados e comportamentos relacionados. No contexto do PowerScript, isso significa mover funções de validação, formatação ou decisão para dentro do próprio objeto que representa a entidade.

### 🔎 Exemplo de Código com Data Class

```pascal
// Structure simples de cliente
global type st_cliente from structure
    string nome
    string email
    string cpf
end type

// Uso em outro código
if not of_validar_email(cliente.email) then
    messagebox("Erro", "Email inválido")
end if
```

Problema: a estrutura apenas armazena os dados. Toda a lógica fica fora dela.

### ✨ Exemplo de Refatoração Aplicando Replace Data Value with Object

1. Criar um objeto com os dados e comportamentos
```pascal
nonvisualobject nv_cliente
    string nome
    string email
    string cpf

    public function boolean of_validar_email()
        return pos(this.email, "@") > 0
    end function

    public function string of_formatar_cpf()
        if len(cpf) = 11 then
            return mid(cpf,1,3)+"."+mid(cpf,4,3)+"."+mid(cpf,7,3)+"-"+mid(cpf,10,2)
        end if
        
        return cpf
    end function
end object
```

2. Exemplo de uso
```pascal
nv_cliente cliente

cliente.nome = "João"
cliente.email = "joao@email.com"
cliente.cpf = "12345678901"

if not cliente.of_validar_email() then
    messagebox("Erro", "Email inválido")
end if

sle_cpf.text = cliente.of_formatar_cpf()
```

### 📈 Benefícios da Refatoração

- O objeto deixa de ser apenas um “porta-dados” e passa a encapsular comportamento relevante.
- Toda a lógica relacionada ao cliente está em um só lugar, facilitando testes e manutenção.
- Aproxima o código de um design orientado a objetos verdadeiro, mesmo no PowerScript.

[Voltar ao início](#sumário)

---

<a name="repeated-switches"></a>
## Repeated Switches

Repetição da mesma estrutura `switch` ou `case` em vários lugares.

### 🧠 Problemas causados

- Duplicação de código.
- Dificuldade para manutenção.

### 🛠️ Solução/Refatoração Recomendada

**Replace Conditional with Polymorphism** ou **Strategy Pattern**.

### 🔎 Exemplo com Repeated Switches

```pascal
choose case tipo_usuario
case "admin"
    of_permissao_admin()
case "cliente"
    of_permissao_cliente()
end choose

choose case tipo_usuario
case "admin"
    of_log_admin()
case "cliente"
    of_log_cliente()
end choose
```

### ✨ Exemplo Refatorado

```pascal
nvo_usuario usuario
usuario.tipo = "admin"
usuario.executar_permissao()
usuario.registrar_log()
```

### 📈 Benefícios da Refatoração

- Reduz duplicação.
- Permite extensão fácil.
- Código mais organizado.

[Voltar ao início](#sumário)

<!-- Links -->
[Catalogo PowerScript]: https://github.com/joaomello03/catalogo
