<a name="inicio"></a>
# [Maus Cheiros de Código em PowerScript][Catalogo PowerScript]

Este catálogo apresenta e descreve os principais maus cheiros de código identificados em projetos desenvolvidos com a linguagem PowerScript. Seu objetivo é auxiliar desenvolvedores na detecção de padrões de código que podem indicar problemas de design, manutenção ou legibilidade, promovendo assim práticas mais eficientes e sustentáveis de desenvolvimento.

## Sumário

1. [Long Parameter List](https://github.com/joaomello03/catalogo/blob/main/README.md#long-parameter-list)
2. [Long Function](https://github.com/joaomello03/catalogo/blob/main/README.md#long-function)
3. [Duplicated Code](https://github.com/joaomello03/catalogo/blob/main/README.md#duplicated-code)
4. [Large Class (VALIDAR)](https://github.com/joaomello03/catalogo/blob/main/README.md#large-class)
5. [Feature Envy](https://github.com/joaomello03/catalogo/blob/main/README.md#feature-envy)
6. [Message Chains (VALIDAR)](https://github.com/joaomello03/catalogo/blob/main/README.md#message-chains)
7. [Shotgun Surgery](https://github.com/joaomello03/catalogo/blob/main/README.md#shotgun-surgery)
8. [Primitive Obsession](https://github.com/joaomello03/catalogo/blob/main/README.md#primitive-obsession)
9. [Data Class](https://github.com/joaomello03/catalogo/blob/main/README.md#data-class)
10. [Repeated Switches (VALIDAR)](https://github.com/joaomello03/catalogo/blob/main/README.md#repeated-switches)
11. [Overloaded Window Script](https://github.com/joaomello03/catalogo/blob/main/README.md#overloaded-window-script)
12. [DataWindow Logic Smell](https://github.com/joaomello03/catalogo/blob/main/README.md#datawindow-logic-smell)
13. [NVO Bloat (NVO Gigante)](https://github.com/joaomello03/catalogo/blob/main/README.md#nvo-bloat)
14. [Hardcoded Paths or Connection Strings](https://github.com/joaomello03/catalogo/blob/main/README.md#hardcoded-paths)
15. [Unmanaged Object Lifetime](https://github.com/joaomello03/catalogo/blob/main/README.md#unmanaged-object-lifetime)
16. [SQL Embedded in Script](https://github.com/joaomello03/catalogo/blob/main/README.md#sql-embedded-script)
17. [Event Cascade Smell](https://github.com/joaomello03/catalogo/blob/main/README.md#event-cascade-smell)
18. [Duplicate DataWindow Objects](https://github.com/joaomello03/catalogo/blob/main/README.md#duplicate-datawindow-objects)
19. [Unused Event Scripts](https://github.com/joaomello03/catalogo/blob/main/README.md#unused-event-scripts)
20. [Modelo Exemplo](https://github.com/joaomello03/catalogo/blob/main/README.md#modelo-exemplo)

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

O **Duplicated Code** ocorre quando blocos de código idênticos ou muito semelhantes são replicados em diferentes partes da aplicação. No PowerScript, esse problema é comum em rotinas como manipulação de dados, formatações, cálculos ou validações que são implementadas repetidamente em múltiplos objetos, janelas ou componentes.

Por exemplo, uma mesma regra de validação de CPF pode ser escrita tanto no cadastro de clientes quanto no cadastro de colaboradores, cada um em objetos distintos. Essa duplicação não apenas gera retrabalho, mas também compromete a manutenibilidade do sistema, dificulta a padronização e aumenta significativamente o risco de erros e inconsistências.

### 🧠 Problemas causados

- Manutenção trabalhosa e arriscada: alterações precisam ser feitas em vários pontos.
- Risco de erros e inconsistências: regras evoluem de forma diferente em locais distintos.
- Código inchado e redundante: maior volume de código para manter e entender.
- Dificuldade de entendimento: regras espalhadas em eventos, objetos e janelas.
- Retrabalho constante: ajustes e correções precisam ser replicados manualmente.

### 🛠️ Solução/Refatoração Recomendada

A refatoração mais indicada para resolver esse problema é o **Extract Method**. É uma técnica onde você move trechos de código repetidos para um método separado e reutilizável. Esse método pode ser colocado em uma classe utilitária, permitindo reuso, centralização da lógica e fácil manutenção.

### 🔎 Exemplo de Código com Duplicated Code

Código presente na tela de cadastro de clientes
```pascal
public function boolean of_validar_dados ();String ls_CPF

ls_CPF = dw_Cliente.GetItemString(1, 'cliente_cpf')

If Not IsNumber(ls_CPF) Or Len(Trim(ls_CPF)) <> 11 Then
    MessageBox('Cadastro de Clientes', 'O CPF do cliente não é válido. Verifique!')
    Return False
End If

Return True
end function
```

Código presente na tela de cadastro de colaboradores
```pascal
public function boolean of_validar_informacoes ();String ls_CPF

ls_CPF = dw_Colaborador.GetItemString(1, 'colaborador_cpf')
ls_CPF = Replace(ls_CPF, '.', '')
ls_CPF = Replace(ls_CPF, '-', '')

If Len(ls_CPF) <> 11 Then
    MessageBox('Cadastro de Colaboradores', 'O CPF do colaborador não é válido. Verifique!')
    Return False
End If

Return True
end function
```

Problemas causados:
- Lógica repetida de validação de CPF, com pequenas variações.
- Inconsistência funcional, pois uma validação remove os caracteres especiais e outra não. 

### ✨ Exemplo de Refatoração Aplicando Extract Method

Criada uma função de validação de CPF em um objeto utilitário, por exemplo, _nv_dados_pessoais_
```pascal
public function boolean of_validar_cpf (string as_cpf);String ls_CPF

ls_CPF = as_CPF
ls_CPF = Replace(ls_CPF, '.', '')
ls_CPF = Replace(ls_CPF, '-', '')
ls_CPF = Trim(ls_CPF)

If Not IsNumber(ls_CPF) Or Len(ls_CPF) <> 11 Then
    Return False
End If

Return True
end function
```

Chamada da função de validação na tela de cadastro de clientes
```pascal
public function boolean of_validar_dados ();String ls_CPF
nv_Dados_Pessoais lnv_Dados_Pessoais

ls_CPF = dw_Cliente.GetItemString(1, 'cliente_cpf')

If Not lnv_Dados_Pessoais.of_Validar_CPF(ls_CPF) Then
    MessageBox('Cadastro de Clientes', 'O CPF do cliente não é válido. Verifique!')
    Return False
End If

Return True
end function
```

Chamada da função de validação na tela de cadastro de colaboradores
```pascal
public function boolean of_validar_informacoes ();String ls_CPF
nv_Dados_Pessoais lnv_Dados_Pessoais

ls_CPF = dw_Colaborador.GetItemString(1, 'colaborador_cpf')

If Not lnv_Dados_Pessoais.of_Validar_CPF(ls_CPF) Then
    MessageBox('Cadastro de Colaboradores', 'O CPF do colaborador não é válido. Verifique!')
    Return False
End If

Return True
end function
```

### 📈 Benefícios da Refatoração

- Evita inconsistências: uma única fonte de verdade para a lógica.
- Facilita a manutenção: alterações são feitas em um só lugar.
- Aumenta a reutilização: o código é mais modular e reaproveitável.
- Melhora a legibilidade: trechos repetitivos saem de cena.
- Reduz erros: menos chances de divergência ou esquecimento.

[Voltar ao início](#sumário)

---

<a name="large-class"></a>
## Large Class (VALIDAR)

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

Esse mau cheiro ocorre quando um método demonstra mais interesse nos dados de outro objeto do que nos dados da própria classe onde está implementado. Em vez de utilizar seus próprios atributos e comportamentos, ele acessa frequentemente métodos ou atributos de outro objeto, indicando que essa lógica provavelmente deveria estar na outra classe.

No PowerScript, esse problema é comum em _Non-Visual Objects (NVOs)_ ou scripts de janelas, onde é comum ver métodos que manipulam diretamente os atributos de outros objetos, quebrando o encapsulamento e gerando dependências desnecessárias.

### 🧠 Problemas causados

- Quebra do encapsulamento, com acesso excessivo aos dados de outros objetos.
- Alto acoplamento entre classes, tornando o código mais frágil e sensível a mudanças.
- Baixa coesão, já que o método está mais relacionado à outra classe do que à própria.
- Dificuldade na manutenção e evolução do código, pois a lógica fica espalhada em locais inadequados.
- Aumento do risco de erros quando há alterações na estrutura interna dos objetos dependentes.

### 🛠️ Solução/Refatoração Recomendada

A refatoração recomendada para tratar o **Feature Envy** é o **Move Method**, conforme descrito por Martin Fowler. Essa técnica consiste em mover o método que está mais interessado nos dados de outra classe para a classe onde esses dados residem, promovendo melhor encapsulamento e coesão.

### 🔎 Exemplo de Código com Feature Envy

Nesse exemplo, temos duas classes:
- _nv_cliente_: responsável pelos dados do cliente.
- _nv_relatorio_: responsável pela geração de relatórios.

Na classe _nv_cliente_ temos os atributos do cliente
```pascal
global type nv_cliente from nonvisualobject
	string is_nome
	string is_sobrenome
end type
```

Na classe _nv_relatorio_ temos uma função responsável por gerar o relatório com o nome completo do cliente
```pascal
public function string of_gerar_relatorio_cliente (nv_cliente lnv_cliente)
	string ls_nomecompleto

	ls_nomecompleto = lnv_cliente.is_nome + " " + lnv_cliente.is_sobrenome

	return "Relatório do Cliente: " + ls_nomecompleto
end function
```

Problema do exemplo acima:
- O método _of_gerar_relatorio_cliente()_ está muito interessado nos atributos da classe _nv_cliente_. Isso é um sinal clássico de **Feature Envy** — ele acessa diretamente dados de outra classe, quando essa responsabilidade poderia estar dentro da própria classe _nv_cliente_.

### ✨ Exemplo de Refatoração Aplicando Move Method

A solução é aplicar o **Move Method**, movendo a responsabilidade de gerar o nome completo para a classe _nv_cliente_. Assim, a classe _nv_relatorio_ não precisa conhecer a estrutura interna da classe cliente.

Criação do nome método na classe _nv_cliente_
```pascal
public function string of_obter_nome_completo()
	return is_nome + " " + is_sobrenome
end function
```

Classe _nv_relatorio_ refatorada, chamando o novo método da classe _nv_cliente_
```pascal
public function string of_gerar_relatorio_cliente (nv_cliente lnv_cliente)
	string ls_nomecompleto

	ls_nomecompleto = lnv_cliente.of_obter_nome_completo()

	return "Relatório do Cliente: " + ls_nomecompleto
end function
```

### 📈 Benefícios da Refatoração

- Melhor encapsulamento: os dados e comportamentos relacionados permanecem juntos.
- Redução do acoplamento: as classes tornam-se menos dependentes umas das outras.
- Facilidade de manutenção: alterações em uma classe têm menos impacto em outras.

[Voltar ao início](#sumário)

---

<a name="message-chains"></a>
## Message Chains (VALIDAR)

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

Shotgun Surgery é um mau cheiro que ocorre quando uma única modificação no sistema exige alterações em vários locais diferentes do código. Isso geralmente acontece quando uma lógica ou regra de negócio está espalhada por múltiplas unidades, como janelas, objetos ou funções, dificultando a manutenção e aumentando o risco de erros.

### 🧠 Problemas causados

- Alto custo de manutenção: pequenas mudanças exigem modificações em diversos pontos do sistema.
- Maior propensão a erros: é fácil esquecer de atualizar algum local, resultando em inconsistências.
- Baixa coesão: funcionalidades relacionadas estão dispersas, violando o princípio da responsabilidade única.
- Dificuldade de testes: testar uma funcionalidade requer verificar múltiplos componentes.

### 🛠️ Solução/Refatoração Recomendada

A forma mais eficaz de resolver esse mau cheiro é aplicar a refatoração **Move Method**, que consiste em centralizar a lógica repetida em um único método reutilizável.

No contexto do PowerScript, isso pode ser feito por meio da criação de um objeto auxiliar (_Non-Visual Object (NVO)_ ou _Function_) que encapsula a lógica que antes estava espalhada por janelas ou scripts diferentes. Assim, outras partes do sistema passam a delegar essa responsabilidade a um único ponto de controle, facilitando a manutenção, reduzindo a duplicação e melhorando a clareza do código.

### 🔎 Exemplo de Código com Shotgun Surgery

```pascal
// Código presente na window 1
if valor_total > 1000 then
    valor_total = valor_total - (valor_total * 0.1)
end if

// Código presente na window 2
if valor_total > 1000 then
    valor_total = valor_total - (valor_total * 0.1)
end if

// Código presente na window 3
if valor_total > 1000 then
    valor_total = valor_total - (valor_total * 0.1)
end if
```

Problema: A lógica de desconto está duplicada em várias janelas. Se a regra mudar (por exemplo, alterar o valor mínimo para desconto ou a porcentagem), será necessário modificar cada local individualmente.

### ✨ Exemplo de Refatoração Aplicando Move Method

1. Criar um objeto do tipo Function
```pascal
global function decimal uf_aplicar_desconto (ade_valortotal)
    if ade_valortotal > 1000 then
        return ade_valortotal - (ade_valortotal * 0.1)
    end if
    
    return ade_valortotal
end function
```

2. Usar a função nas janelas
```pascal
// Código presente na window 1
lde_valortotal = uf_aplicar_desconto(lde_valortotal)

// Código presente na window 2
lde_valortotal = uf_aplicar_desconto(lde_valortotal)

// Código presente na window 3
lde_valortotal = uf_aplicar_desconto(lde_valortotal)
```

### 📈 Benefícios da Refatoração

- Centralização da lógica de desconto em uma única função.
- Facilidade para atualizar regras de negócio: qualquer mudança futura será feita em apenas um lugar.
- Redução de código duplicado em todo o sistema.
- Melhor legibilidade e manutenção, alinhado com boas práticas de reuso.

[Voltar ao início](#sumário)

---

<a name="primitive-obsession"></a>
## Primitive Obsession

Primitive Obsession é um mau cheiro de código que ocorre quando usamos tipos primitivos (_string, integer, decimal_, etc.) de forma excessiva ou inadequada, em vez de abstrações mais expressivas. Isso inclui:
- Passar muitos primitivos como parâmetros que poderiam ser agrupados.
- Representar conceitos complexos com apenas uma string ou integer.
- Repetir validações ou formatações em vários lugares para o mesmo tipo de dado.

### 🧠 Problemas causados

- Repetição de código: validações ou cálculos relacionados ao mesmo tipo de dado aparecem em vários pontos.
- Dificuldade de manutenção: qualquer mudança no comportamento exige alterações em múltiplos lugares.
- Acoplamento excessivo a tipos básicos: o código se torna mais rígido e menos reutilizável.

### 🛠️ Solução/Refatoração Recomendada

Aplicar a refatoração **Replace Primitive with Value Object**, que consiste em substituir um ou mais campos primitivos por um objeto específico que represente o conceito, encapsulando os dados e as operações relacionadas.
Em PowerScript, use um _Non-Visual Object (NVO)_ ou para representar um conceito de domínio (ex: ClienteInfo, Endereco, CPF, Produto), e mova validações, formatações e outras operações para dentro dele.

### 🔎 Exemplo de Código com Primitive Obsession

```pascal
// Função de verificação com múltiplos parâmetros primitivos
public function boolean of_cadastrar_cliente(string nome, string email, string cpf, string telefone)
    if nome = "" or pos(email, "@") = 0 or len(cpf) <> 11 then
        return false
    end if

    ...

    return true
end function
```

Problemas no exemplo acima:
- Muitos parâmetros primitivos soltos.
- Validações específicas espalhadas.

### ✨ Exemplo de Refatoração Aplicando Replace Primitive with Value Object

1. Criar um objeto _nv_cliente_
```pascal
global type nv_cliente from nonvisualobject
    string nome
    string email
    string cpf
    string telefone

    public function boolean of_validar()
        if this.nome = "" then return false
        if pos(this.email, "@") = 0 then return false
        if len(this.cpf) <> 11 then return false

        return true
    end function

    public function string of_formatar_cpf()
        if len(cpf) = 11 then
            return mid(cpf,1,3)+"."+mid(cpf,4,3)+"."+mid(cpf,7,3)+"-"+mid(cpf,10,2)
        end if

        return cpf
    end function
end object
```

2. Refatorar a função de cadastro para usar o objeto
```pascal
public function boolean of_cadastrar_cliente(nv_cliente cliente)
    if not cliente.of_validar() then
        return false
    end if

    ...
    
    return true
end function
```

3. Uso no sistema
```pascal
nv_cliente cliente
cliente.nome = "João"
cliente.email = "joao@email.com"
cliente.cpf = "12345678901"
cliente.telefone = "46999999999"

boolean sucesso = of_cadastrar_cliente(cliente)

if not sucesso then
    messagebox("Erro", "Dados inválidos.")
end if
```

### 📈 Benefícios da Refatoração

- Os dados agora representam um conceito de negócio, e não apenas tipos primitivos.
- As validações e operações estão encapsuladas, evitando duplicações e facilitando testes.
- Reduz o número de parâmetros e aumenta a clareza e legibilidade das funções.
- Deixa o sistema mais coeso, reutilizável e preparado para mudanças (ex: se a validação do CPF mudar, altera-se em um só lugar).

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
global type nv_cliente from nonvisualobject
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
## Repeated Switches (VALIDAR)

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

---

# [Específicos PowerScript]

---

<a name="overloaded-window-script"></a>
## Overloaded Window Script

Scripts de eventos (como _open, clicked, itemchanged_) acumulam muita lógica de negócio diretamente no objeto visual (_Window, UserObject, DataWindow Control_). Isso geralmente ocorre quando a lógica de persistência, validação ou processamento de dados é implementada dentro da própria janela, em vez de ser delegada a um objeto especializado.

### 🧠 Problemas causados

- Mistura de camadas (UI + lógica de negócio), tornando o código difícil de testar, reutilizar e manter.
- Aumenta o acoplamento entre interface e regras de negócio.
- Qualquer mudança de regra exige alterar o código da interface.
- Viola o princípio de Separação de Responsabilidades (_Single Responsibility Principle_).

### 🛠️ Solução/Refatoração Recomendada

Mover a lógica de negócio para _Non-Visual Objects (NVOs)_, que podem ser chamados a partir dos eventos visuais.
Aplicar o padrão de _event routing_ — isto é, o evento da interface apenas delega a ação para um método especializado.

### 🔎 Exemplo com Overloaded Window Script

```pascal
// --- Evento Clicked() da janela w_Vendas ---

// Validação e persistência diretamente na janela (ruim)
String ls_Cliente, ls_Produto
Decimal lde_Total

ls_Cliente = dw_Vendas.GetItemString(1, 'cliente')
ls_Produto = dw_Vendas.GetItemString(1, 'produto')
lde_Total = dw_Vendas.GetItemDecimal(1, 'total')

If ls_Cliente = '' Or ls_Produto = '' Or ls_Total = '' Then
	MessageBox("Erro", "Informe o cliente e o produto antes de salvar.")
    Return
End If

If dw_Vendas.Update() = 1 Then
    COMMIT;
    MessageBox("Sucesso", "Venda registrada com sucesso!")
Else
    ROLLBACK;
    MessageBox("Erro", "Falha ao registrar a venda.")
End If
```

### ✨ Exemplo Refatorado

```pascal
// --- Evento Clicked() da janela w_Vendas ---

// Delegando a lógica para um serviço
n_Servico_Vendas lnv_Vendas

Create lnv_Vendas

Try
	lnv_Vendas.of_Registrar_Venda(dw_Vendas)
Catch (EXCEPTION ex)
    MessageBox("Erro", ex.GetMessage())
Finally
    Destroy(lnv_Vendas)
End Try
```

```pascal
// --- Non-Visual Object: n_Servico_Vendas ---

public function integer of_Registrar_Venda (DataWindow adw_Vendas);
	String ls_Cliente, ls_Produto
	Decimal lde_Total

	ls_Cliente = adw_Vendas.GetItemString(1, 'cliente')
	ls_Produto = adw_Vendas.GetItemString(1, 'produto')
	lde_Total = adw_Vendas.GetItemDecimal(1, 'total')

	If ls_Cliente = '' Or ls_Produto = '' Or ls_Total = '' Then
		RAISE EXCEPTION CreateException("Campos obrigatórios não informados.")
	End If
	
	If adw_Vendas.Update() = 1 Then
		COMMIT;
		Return 1
	Else
		ROLLBACK;
        RAISE EXCEPTION CreateException("Erro ao gravar a venda.")
    End If
end function
```

### 📈 Benefícios da Refatoração

- A janela (UI) fica limpa, contendo apenas chamadas de evento e delegação de ações.
- A lógica de negócio fica centralizada, reutilizável e testável em um _Non-Visual Object (NVO)_.
- Reduz o acoplamento entre interface e persistência.
- Facilita manutenção, testes unitários e reaproveitamento em outras janelas.

[Voltar ao início](#sumário)

---






















<a name="datawindow-logic-smell"></a>
## DataWindow Logic Smell

Regras de negócio, cálculos e validações implementadas diretamente em expressões, eventos ou triggers do **DataWindow** (como _itemchanged, itemfocuschanged_ ou expressões computadas). Esse tipo de implementação mistura lógica de domínio com a camada de apresentação dos dados.

### 🧠 Problemas causados

- A lógica fica “escondida” dentro do DataWindow, dificultando testes e manutenção.
- Cada alteração exige abrir o designer do DataWindow, tornando o processo lento e arriscado.
- Regras de negócio acabam duplicadas em vários DataWindows diferentes.
- Viola o princípio de Separação de Responsabilidades, pois a camada de apresentação passa a conter lógica de negócio.

### 🛠️ Solução/Refatoração Recomendada

Extrair a lógica de negócio das expressões e eventos do DataWindow e movê-la para _Non-Visual Objects (NVOs)_.
Utilizar métodos de validação e cálculo fora do objeto visual, deixando o DataWindow apenas responsável pela exibição e manipulação de dados.

### 🔎 Exemplo com DataWindow Logic Smell

```pascal
// --- Evento ItemChanged() dentro do DataWindow dw_Funcionario ---

If dwo.Name = 'salario' Then
	If Decimal(Data) > 10000 Then
		dw_Funcionario.SetItem(Row, 'classificacao', 'Sênior')
    Else
		dw_Funcionario.SetItem(Row, 'classificacao', 'Júnior')
	End If
End If
```

Neste exemplo, a lógica de classificação de funcionário está diretamente embutida no DataWindow, tornando difícil testar ou reaproveitar em outro contexto.

### ✨ Exemplo Refatorado

```pascal
// --- Evento ItemChanged() da janela ou UserObject que contém o DataWindow ---
If dwo.Name = 'salario' Then
	n_Servico_Funcionario lnv_Funcionario
	String ls_Classificacao

	Create lnv_Funcionario
	Try
		ls_Classificacao = lnv_Funcionario.of_Definir_Classificacao(Decimal(Data))
		dw_Funcionario.SetItem(Row, 'classificacao', ls_Classificacao)
	Finally
		Destroy(lnv_Funcionario)
	End Try
End If
```

```pascal
// --- Non-Visual Object: n_Servico_Funcionario ---

public function string of_Definir_Cargo (Decimal ade_Salario);
	If ade_Salario > 10000 Then
		Return 'Sênior'
	Else
		Return 'Júnior'
	End If
end function
```

### 📈 Benefícios da Refatoração

- A regra de negócio fica isolada em um _Non-Visual Object (NVO)_ reutilizável e testável.
- O DataWindow passa a cuidar apenas da interface e da manipulação de dados.
- Manutenções futuras podem ser feitas sem abrir o editor visual.
- Reduz duplicação de lógica entre diferentes DataWindows.
- Facilita a implementação de testes automatizados e a evolução do sistema.

[Voltar ao início](#sumário)






















---

<a name="nvo-bloat"></a>
## NVO Bloat (NVO Gigante)

Objetos não visuais que centralizam toda a lógica da aplicação — muitas vezes virando “God Objects”.

### 🧠 Problemas causados

Responsabilidades múltiplas, difícil de manter e testar.
Pequenas alterações podem quebrar várias funcionalidades.

### 🛠️ Solução/Refatoração Recomendada

Dividir o NVO em serviços menores e especializados (Extract NVO, Facade Pattern).

### 🔎 Exemplo com NVO Bloat (NVO Gigante)

### ✨ Exemplo Refatorado

### 📈 Benefícios da Refatoração

[Voltar ao início](#sumário)

---

<a name="hardcoded-paths"></a>
## Hardcoded Paths or Connection Strings

Strings de conexão, caminhos de arquivos e credenciais codificadas diretamente no script.

### 🧠 Problemas causados

Dificulta manutenção, viola segurança e impede portabilidade.

### 🛠️ Solução/Refatoração Recomendada

Ler valores de arquivo INI, banco de dados de configuração, ou variáveis de ambiente.

### 🔎 Exemplo com Hardcoded Paths or Connection Strings

### ✨ Exemplo Refatorado

### 📈 Benefícios da Refatoração

[Voltar ao início](#sumário)

---

<a name="unmanaged-object-lifetime"></a>
## Unmanaged Object Lifetime

Criação de objetos (CREATE), DataStores ou DataWindows sem DESTROY correspondente.

### 🧠 Problemas causados

Vazamento de memória e conexões não encerradas.

### 🛠️ Solução/Refatoração Recomendada

Garantir que cada CREATE tenha DESTROY dentro de finally ou no Close do container.

### 🔎 Exemplo com Unmanaged Object Lifetime

### ✨ Exemplo Refatorado

### 📈 Benefícios da Refatoração

[Voltar ao início](#sumário)

---

<a name="sql-embedded-script"></a>
## SQL Embedded in Script

Consultas SQL complexas escritas diretamente em scripts PowerScript em vez de DataWindows ou Stored Procedures.

### 🧠 Problemas causados

Dificulta manutenção, reaproveitamento e otimização no banco.

### 🛠️ Solução/Refatoração Recomendada

Migrar para DataWindow ou procedure; encapsular SQL em métodos especializados.

### 🔎 Exemplo com SQL Embedded in Script

### ✨ Exemplo Refatorado

### 📈 Benefícios da Refatoração

[Voltar ao início](#sumário)

---

<a name="event-cascade-smell"></a>
## Event Cascade Smell

Eventos que chamam outros eventos de forma encadeada (por exemplo, Clicked() chamando ItemChanged() manualmente).

### 🧠 Problemas causados

Difícil de entender e depurar; ordem de execução pode gerar efeitos colaterais inesperados.

### 🛠️ Solução/Refatoração Recomendada

Extrair a lógica para métodos isolados e chamar diretamente, sem dependência entre eventos.

### 🔎 Exemplo com Event Cascade Smell

### ✨ Exemplo Refatorado

### 📈 Benefícios da Refatoração

[Voltar ao início](#sumário)

---

<a name="duplicate-datawindow-objects"></a>
## Duplicate DataWindow Objects

Várias DataWindows diferentes com o mesmo SQL e estrutura, criadas para telas distintas.

### 🧠 Problemas causados

Duplicação de lógica e maior esforço de manutenção.

### 🛠️ Solução/Refatoração Recomendada

Centralizar DataWindows reutilizáveis; parametrizar visualização ou usar herança de DataWindow (inheritance).

### 🔎 Exemplo com Duplicate DataWindow Objects

### ✨ Exemplo Refatorado

### 📈 Benefícios da Refatoração

[Voltar ao início](#sumário)

---

<a name="unused-event-scripts"></a>
## Unused Event Scripts

Eventos criados automaticamente (por herança ou IDE) mas nunca utilizados.

### 🧠 Problemas causados

Ruído no código e confusão durante manutenção.

### 🛠️ Solução/Refatoração Recomendada

Remover scripts vazios ou consolidar no evento pai.

### 🔎 Exemplo com Unused Event Scripts

### ✨ Exemplo Refatorado

### 📈 Benefícios da Refatoração

[Voltar ao início](#sumário)

---

<a name="modelo-exemplo"></a>
## Modelo Exemplo

### 🧠 Problemas causados

### 🛠️ Solução/Refatoração Recomendada

### 🔎 Exemplo com Modelo Exemplo

### ✨ Exemplo Refatorado

### 📈 Benefícios da Refatoração

[Voltar ao início](#sumário)

---

<!-- Links -->
[Catalogo PowerScript]: https://github.com/joaomello03/catalogo
