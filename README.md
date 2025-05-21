<a name="inicio"></a>
# [Maus Cheiros de Código em PowerScript][Catalogo PowerScript]

Este catálogo apresenta e descreve os principais maus cheiros de código identificados em projetos desenvolvidos com a linguagem PowerScript. Seu objetivo é auxiliar desenvolvedores na detecção de padrões de código que podem indicar problemas de design, manutenção ou legibilidade, promovendo assim práticas mais eficientes e sustentáveis de desenvolvimento.

## Sumário

1. [Long Parameter List](https://github.com/joaomello03/catalogo/blob/main/README.md#long-parameter-list)
2. [Long Function](https://github.com/joaomello03/catalogo/blob/main/README.md#long-function)
3. [Duplicated Code]()
4. [Large Class]()

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

### ✨ Exemplo de Refatoração Aplicando **Introduce Parameter Object**

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

### ✨ Exemplo de Refatoração Aplicando **Extract Method**

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

<!-- Links -->
[Catalogo PowerScript]: https://github.com/joaomello03/catalogo
