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

Esse code smell ocorre quando um método ou função recebe muitos parâmetros. Métodos com longas listas de parâmetros são difíceis de entender, propensos a erros (como a troca de ordem dos argumentos) e tornam a manutenção do código mais trabalhosa.
Em PowerScript, que tem forte foco em manipulação de dados e eventos, esse problema pode surgir facilmente.

### 🧠 Problemas causados

- Dificuldade de leitura e compreensão do método.
- Risco de erro ao passar argumentos incorretos ou na ordem errada.
- Dificuldade para fazer alterações futuras (adicionar, remover ou reorganizar parâmetros).
- Redução da coesão e aumento da dependência entre partes do sistema.

### 🛠️ Solução/Refatoração Recomendada

Aplicar a refatoração **Introduce Parameter Object**, onde cria-se uma estrutura (em PowerScript, uma structure ou class) que agrupa os parâmetros relacionados.
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
- Facilita a manutenção e a extensão do sistema (adicionar novos atributos ao Info_Produto, por exemplo).
- Torna o código mais legível e menos propenso a erros.

[Voltar ao início](#sumário)

<a name="LongFunction"></a>
## Long Function

Descrição

### 🧠 Problemas causados

Problemas causados

### 🛠️ Solução/Refatoração Recomendada

Solução/Refatoração Recomendada

### 🔎 Exemplo de Código com Long Parameter List

```pascal
// Exemplo de Código com Long Parameter List
```

### ✨ Exemplo de Refatoração Aplicando Introduce Parameter Object

Exemplo de Refatoração Aplicando Introduce Parameter Object

### 📈 Benefícios da Refatoração

Benefícios da Refatoração

<!-- Links -->
[Catalogo PowerScript]: https://github.com/joaomello03/catalogo
