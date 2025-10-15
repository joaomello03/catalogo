<a name="inicio"></a>
# [Maus Cheiros de Código em PowerScript][Catalogo PowerScript]

Este catálogo apresenta e descreve os principais maus cheiros de código identificados em projetos desenvolvidos com a linguagem PowerScript. Seu objetivo é auxiliar desenvolvedores na detecção de padrões de código que podem indicar problemas de design, manutenção ou legibilidade, promovendo assim práticas mais eficientes e sustentáveis de desenvolvimento.

## Sumário

### Traditional Code Smells
1. [Long Parameter List](#long-parameter-list)
2. [Long Function](#long-function)
3. [Duplicated Code](#duplicated-code)
4. [Large Class](#large-class)
5. [Feature Envy](#feature-envy)
6. [Message Chains](#message-chains)
7. [Shotgun Surgery](#shotgun-surgery)
8. [Primitive Obsession](#primitive-obsession)
9. [Data Class](#data-class)
10. [Repeated Switches](#repeated-switches)

### PowerScript Specific Code Smells
11. [Overloaded Window Script](#overloaded-window-script)
12. [DataWindow Logic Smell](#datawindow-logic-smell)
13. [Hardcoded Paths or Connection Strings](#hardcoded-paths)
14. [Unmanaged Object Lifetime](#unmanaged-object-lifetime)
15. [SQL Embedded in Script](#sql-embedded-script)
16. [Event Cascade Smell](#event-cascade-smell)
17. [Duplicate DataWindow Objects](#duplicate-datawindow-objects)
18. [Unused Event Scripts](#unused-event-scripts)
19. [Communication Object](#communication-object)
20. [Public Field](#public-field)
21. [GOTO Backward Jump](#goto-backward-jump)
22. [Improper Use of Destroy Function](#improper-use-destroy)
23. [DataWindow Object Reference](#datawindow-object-reference)

---

<a id="long-parameter-list"></a>
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

<a id="long-function"></a>
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

<a id="duplicated-code"></a>
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

<a id="large-class"></a>
## Large Class

Ocorre quando uma classe (por exemplo, um _Non-Visual Object — NVO_) acumula **responsabilidades demais** ou contém **muitos métodos e atributos**.
Isso torna o código difícil de entender, manter e testar, já que a classe concentra lógicas que deveriam estar distribuídas entre múltiplos objetos.

Em PowerScript, é comum encontrar esse problema em _Non-Visual Objects — NVOs_ genéricos, que acabam se tornando “_classes deus_” (_God Objects_).

### 🧠 Problemas causados

- Viola o **Princípio da Responsabilidade Única (SRP)**.
- Dificulta a manutenção, pois qualquer mudança pode impactar múltiplas áreas.
- Gera dependências desnecessárias entre funcionalidades sem relação direta.
- Reduz a capacidade de reutilização e testabilidade.
- Aumenta o tempo de entendimento para novos desenvolvedores.

### 🛠️ Solução/Refatoração Recomendada

Aplicar a refatoração **Extract Class** (Extrair Classe) — separar responsabilidades em classes menores e coesas.
Cada classe deve focar em uma funcionalidade específica (por exemplo, Gestão de Cliente, Gestão de Vendas, Relatórios).

Além disso, é possível aplicar **Move Method** para transferir lógicas diretamente para as novas classes criadas.

### 🔎 Exemplo de Código com Large Class

```pascal
// --- Non-Visual Object: n_Gerenciador ---
// Exemplo de classe "inchada", com múltiplas responsabilidades

public function integer of_Salvar_Cliente (DataWindow adw_Cliente)
	adw_Cliente.Update()
	COMMIT USING SQLCA;
	Return 1
end function

public function integer of_Gerar_Relatorio_Vendas ()
	String ls_SQL

	ls_SQL = "SELECT * FROM VENDAS WHERE DATA >= TODAY() - 30"
	EXECUTE IMMEDIATE :ls_SQL USING SQLCA;

	Return 1
end function

public function integer of_Enviar_Email (String as_Destinatario, String as_Mensagem)
	// Simulação de envio de e-mail
	MessageBox("E-mail enviado", "Para: " + as_Destinatario + "~r~nMensagem: " + as_Mensagem)
	Return 1
end function

public function decimal of_Calcular_Desconto (Decimal ade_Valor, Integer ai_Percentual)
	Return ade_Valor - (ade_Valor * ai_Percentual / 100)
end function

public function integer of_Gerar_Backup ()
	// Simulação de backup
	MessageBox("Backup", "Backup concluído com sucesso.")
	Return 1
end function
```

Neste exemplo, _n_Gerenciador_ centraliza várias funções sem relação direta — cliente, vendas, e-mail, cálculo e backup. A classe está sobrecarregada e viola o princípio de responsabilidade única.

### ✨ Exemplo Refatorado

```pascal
// --- Non-Visual Object: n_Servico_Cliente ---
public function integer of_Salvar_Cliente (DataWindow adw_Cliente)
	adw_Cliente.Update()
	COMMIT USING SQLCA;
	Return 1
end function
```

```pascal
// --- Non-Visual Object: n_Servico_Vendas ---
public function integer of_Gerar_Relatorio_Vendas ()
	String ls_SQL

	ls_SQL = "SELECT * FROM VENDAS WHERE DATA >= TODAY() - 30"
	EXECUTE IMMEDIATE :ls_SQL USING SQLCA;

	Return 1
end function

public function decimal of_Calcular_Desconto (Decimal ade_Valor, Integer ai_Percentual)
	Return ade_Valor - (ade_Valor * ai_Percentual / 100)
end function
```

```pascal
// --- Non-Visual Object: n_Servico_Email ---
public function integer of_Enviar_Email (String as_Destinatario, String as_Mensagem)
	// Simulação de envio de e-mail
	MessageBox("E-mail enviado", "Para: " + as_Destinatario + "~r~nMensagem: " + as_Mensagem)
	Return 1
end function
```

```pascal
// --- Non-Visual Object: n_Servico_Backup ---
public function integer of_Gerar_Backup ()
	// Simulação de backup
	MessageBox("Backup", "Backup concluído com sucesso.")
	Return 1
end function
```

```pascal
// --- Non-Visual Object: n_Gerenciador (refatorado) ---
// Centraliza a orquestração, delegando responsabilidades específicas

n_Servico_Cliente    inv_Cliente
n_Servico_Vendas     inv_Vendas
n_Servico_Email      inv_Email
n_Servico_Backup     inv_Backup

public function integer of_Inicializar ()
	Create inv_Cliente
	Create inv_Vendas
	Create inv_Email
	Create inv_Backup

	Return 1
end function

public function integer of_Finalizar ()
	Destroy inv_Cliente
	Destroy inv_Vendas
	Destroy inv_Email
	Destroy inv_Backup

	Return 1
end function
```

Agora, o _n_Gerenciador_ atua apenas como coordenador, enquanto as classes menores são focadas e reutilizáveis.

### 📈 Benefícios da Refatoração

- Cada classe possui uma responsabilidade única e bem definida.
- Reduz a complexidade e facilita a leitura e manutenção.
- Melhora a testabilidade e a modularidade do sistema.
- Facilita a reutilização de componentes em diferentes janelas e contextos.
- Diminui o risco de efeitos colaterais ao alterar uma funcionalidade.

[Voltar ao início](#sumário)

---

<a id="feature-envy"></a>
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

<a id="message-chains"></a>
## Message Chains

Ocorre quando um objeto acessa uma longa cadeia de chamadas para alcançar outro objeto ou método, como _obj_a.of_get_b().of_get_c().of_get_d()_.
Esse padrão cria um forte **acoplamento entre classes** e viola a **Lei de Deméter** (“não fale com estranhos”).
Em PowerScript, isso aparece com frequência quando uma janela ou DataWindow acessa diretamente métodos de objetos internos de serviços, controladores ou repositórios.

### 🧠 Problemas causados

- Excesso de acoplamento entre camadas (UI, serviço, repositório).
- Viola o encapsulamento, expondo detalhes internos da hierarquia de objetos.
- Pequenas mudanças em classes internas quebram várias partes do código cliente.
- Torna o código mais difícil de entender e testar isoladamente.

### 🛠️ Solução/Refatoração Recomendada

Aplicar a refatoração **Hide Delegate**, a ideia é ocultar a delegação criando métodos intermediários que encapsulam o acesso aos objetos internos.

Assim, o cliente não precisa conhecer a cadeia de dependências — ele interage apenas com a fachada principal, que encaminha as chamadas internamente.

Essa abordagem reduz o acoplamento e simplifica o relacionamento entre objetos.

### 🔎 Exemplo de Código com Message Chains

```pascal
// --- Script no evento ue_Processar() de uma janela ---

n_Servico_Processar lnv_Processar
String ls_NomeCliente

Create lnv_Processar

// Cadeia longa de chamadas — viola a Lei de Deméter
ls_NomeCliente = lnv_Processar.of_Get_Servico_Vendas().of_Get_Repositorio_Cliente().of_Get_Cliente_Nome(sle_Id_Cliente.Text)

MessageBox("Cliente", "Nome: " + ls_NomeCliente)

Destroy lnv_Processar
```

Neste caso, a janela conhece detalhes demais da estrutura interna do sistema (_serviço → repositório → cliente_). Qualquer mudança em um desses níveis obrigaria alterações na interface da janela.


### ✨ Exemplo Refatorado

```pascal
// --- Evento ue_Processar() da janela ---

n_Servico_Processar lnv_Processar
String ls_NomeCliente

Create lnv_Processar

Try
	ls_NomeCliente = lnv_Processar.of_Obter_Nome_Cliente(sle_ID_Cliente.Text)
	MessageBox("Cliente", "Nome: " + ls_NomeCliente)
Finally
	Destroy lnv_Processar
End Try
```

```pascal
// --- Non-Visual Object: n_Servico_Processar ---

// Método criado para ocultar a delegação — aplica Hide Delegate
public function string of_Obter_Nome_Cliente (Long al_IdCliente)
	Return of_Get_Servico_Vendas().of_Obter_Nome_Cliente(al_IdCliente)
end function

public function n_servico_vendas of_Get_Servico_Vendas ()
	Return inv_Servico_Vendas
end function
```

```pascal
// --- Non-Visual Object: n_Servico_Vendas ---

public function string of_Obter_Nome_Cliente (Long al_IdCliente)
	Return of_Get_Repositorio_Cliente().of_Get_Cliente_Nome(al_IdCliente)
end function

public function n_repositorio_cliente of_Get_Repositorio_Cliente ()
	Return inv_Repositorio_Cliente
end function
```

```pascal
// --- Non-Visual Object: n_Repositorio_Cliente ---

public function string of_Get_Cliente_Nome (Long al_IdCliente)
	String ls_Nome

	SELECT NOME INTO :ls_Nome FROM CLIENTE WHERE ID = :al_IdCliente USING SQLCA;

	Return ls_Nome
end function
```

Agora, a janela conversa apenas com _n_Servico_Processar_, que oculta toda a delegação interna. Mesmo que o serviço ou o repositório mudem, a interface da janela continua estável.

### 📈 Benefícios da Refatoração

- Aplica corretamente o **Hide Delegate**, reduzindo o acoplamento.
- Encapsula detalhes internos e protege o código cliente contra mudanças estruturais.
- Facilita testes unitários e evolução modular do sistema.
- Segue a **Lei de Deméter**, mantendo comunicações mais diretas e seguras entre camadas.

[Voltar ao início](#sumário)

---

<a id="shotgun-surgery"></a>
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

<a id="primitive-obsession"></a>
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

<a id="data-class"></a>
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

<a id="repeated-switches"></a>
## Repeated Switches

Ocorre quando múltiplos blocos _CHOOSE CASE (ou IF...ELSE IF...)_ são usados repetidamente em diferentes partes do código para tomar decisões baseadas no mesmo tipo de valor. Essa repetição indica falta de abstração e torna o sistema mais difícil de manter e evoluir.

### 🧠 Problemas causados

- Viola o princípio **DRY (Don’t Repeat Yourself)**.
- Qualquer alteração na lógica exige modificar vários pontos do sistema.
- Aumenta o risco de inconsistências entre blocos semelhantes.
- Dificulta a extensão do código para novos tipos ou casos.

### 🛠️ Solução/Refatoração Recomendada

Substituir estruturas repetitivas por **polimorfismo** (via _Non-Visual Objects (NVOs)_ especializados), **tabelas de decisão** ou **métodos centralizados**.
Criar uma hierarquia de objetos que encapsule a lógica de cada tipo de caso, evitando duplicação e facilitando manutenção.

### 🔎 Exemplo de Código com Repeated Switches

```pascal
// --- Evento Clicked do botão "Processar" ---
String ls_Tipo = sle_Tipo.Text

Choose Case ls_Tipo
	Case "V"
		MessageBox("Venda", "Processando venda...")
	Case "C"
		MessageBox("Compra", "Processando compra...")
	Case "E"
		MessageBox("Estoque", "Atualizando estoque...")
	Case Else
		MessageBox("Erro", "Tipo desconhecido.")
End Choose

// Outro ponto do sistema (relatório)
Choose Case ls_Tipo
	Case "V"
		ls_Titulo = "Relatório de Vendas"
	Case "C"
		ls_Titulo = "Relatório de Compras"
	Case "E"
		ls_Titulo = "Relatório de Estoque"
	Case Else
		ls_Titulo = "Relatório Desconhecido"
End Choose
```

Neste exemplo, a decisão baseada em _ls_Tipo_ é repetida em diferentes partes do sistema. Se for adicionado um novo tipo (“Devolução”, por exemplo), será necessário atualizar todos os blocos _CHOOSE CASE_ manualmente.

### ✨ Exemplo Refatorado

```pascal
// --- Evento Clicked do botão "Processar" ---
n_Operacao lnv_Operacao
Create lnv_Operacao

Try
	lnv_Operacao.of_Executar_Operacao(sle_Tipo.Text)
Finally
	Destroy lnv_Operacao
End Try
```

```pascal
// --- Non-Visual Object: n_Operacao ---

public function integer of_Executar_Operacao (String as_Tipo)
	n_Operacao_Base lnv_Handler
	lnv_Handler = of_Get_Handler(as_Tipo)
	
	If Not IsValid(lnv_Handler) Then
		MessageBox("Erro", "Tipo de operação inválida.")
		Return -1
	End If
	
	lnv_Handler.of_Executar()
	Destroy lnv_Handler
	Return 1
end function

private function n_operacao_base of_Get_Handler (String as_Tipo)
	Choose Case as_Tipo
		Case "V"
			Return Create n_Operacao_Venda
		Case "C"
			Return Create n_Operacao_Compra
		Case "E"
			Return Create n_Operacao_Estoque
		Case Else
			Return Null
	End Choose
end function
```

```pascal
// --- Classe base: n_Operacao_Base ---
public subroutine of_Executar()
	// Método sobrescrito nas subclasses
end subroutine


// --- Subclasse: n_Operacao_Venda ---
public subroutine of_Executar()
	MessageBox("Venda", "Processando venda...")
end subroutine


// --- Subclasse: n_Operacao_Compra ---
public subroutine of_Executar()
	MessageBox("Compra", "Processando compra...")
end subroutine


// --- Subclasse: n_Operacao_Estoque ---
public subroutine of_Executar()
	MessageBox("Estoque", "Atualizando estoque...")
end subroutine
```

Neste exemplo, o comportamento específico é **delegado a classes especializadas**, eliminando a repetição de blocos _CHOOSE CASE_ espalhados.
Adicionar uma nova operação agora requer apenas criar uma nova subclasse — sem modificar código existente.

### 📈 Benefícios da Refatoração

- Elimina duplicação de código e blocos de decisão redundantes.
- Facilita a adição de novos comportamentos sem alterar código existente.
- Centraliza a lógica de decisão, reduzindo erros e inconsistências.
- Melhora a legibilidade e a extensibilidade da aplicação.
- Adere aos princípios de **Polimorfismo**.

[Voltar ao início](#sumário)

---

<a id="overloaded-window-script"></a>
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

### 🔎 Exemplo de Código com Overloaded Window Script

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

public function integer of_Registrar_Venda (DataWindow adw_Vendas)
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

<a id="datawindow-logic-smell"></a>
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

### 🔎 Exemplo de Código com DataWindow Logic Smell

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

public function string of_Definir_Cargo (Decimal ade_Salario)
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

<a id="hardcoded-paths"></a>
## Hardcoded Paths or Connection Strings

Strings de conexão, caminhos de arquivos, diretórios e credenciais são definidos diretamente no código PowerScript. Esse tipo de implementação torna o sistema rígido, pouco configurável e vulnerável a falhas ou exposição de informações sensíveis.

### 🧠 Problemas causados

- Dificulta a manutenção, pois qualquer alteração exige recompilar e redistribuir a aplicação.
- Viola boas práticas de segurança, expondo senhas e informações de ambiente.
- Reduz a portabilidade, já que caminhos e configurações ficam fixos no código.
- Impede a reutilização do mesmo código em diferentes ambientes (desenvolvimento, teste, produção).

### 🛠️ Solução/Refatoração Recomendada

Remover valores fixos do código e carregá-los dinamicamente a partir de **arquivos de configuração (INI, JSON, XML)**, **tabelas de configuração no banco de dados**, ou **variáveis de ambiente**.

Criar um objeto responsável por gerenciar as configurações de conexão e caminhos de forma centralizada.

### 🔎 Exemplo de Código com Hardcoded Paths or Connection Strings

```pascal
// --- Script no evento Open() da aplicação ---
SQLCA.DBMS = "ODBC"
SQLCA.Database = "db_vendas"
SQLCA.UserID = "admin"
SQLCA.DBPass = "12345"
SQLCA.LogId = "app_user"
SQLCA.LogPass = "abcde"

CONNECT USING SQLCA;

If SQLCA.SQLCode = 0 Then
	MessageBox("Conexão", "Conexão estabelecida com sucesso.")
Else
	MessageBox("Erro", "Falha ao conectar ao banco de dados.")
End If
```

Neste exemplo, as credenciais e parâmetros de conexão estão codificados diretamente no código, o que torna o sistema inseguro e inflexível.

### ✨ Exemplo Refatorado

```pascal
// --- Evento Open() da aplicação ---
n_Servico_Config lnv_Config

Create lnv_Config

Try
	SQLCA = lnv_Config.of_Carregar_Conexao("producao")

	CONNECT USING SQLCA;

	If SQLCA.SQLCode = 0 Then
		MessageBox("Conexão", "Conexão estabelecida com sucesso.")
	Else
		MessageBox("Erro", "Falha ao conectar ao banco de dados.")
	End If
Finally
	Destroy(lnv_Config)
End Try
```

```pascal
// --- Non-Visual Object: n_Servico_Config ---

public function transaction of_Carregar_Conexao (String as_Ambiente)
	Transaction lt_SQLCA
	String ls_Arquivo_INI = "app_config.ini"

	Create lt_SQLCA

	lt_SQLCA.DBMS      = ProfileString(ls_Arquivo_INI, as_Ambiente, "DBMS", "")
	lt_SQLCA.Database  = ProfileString(ls_Arquivo_INI, as_Ambiente, "Database", "")
	lt_SQLCA.UserID    = ProfileString(ls_Arquivo_INI, as_Ambiente, "UserID", "")
	lt_SQLCA.DBPass    = ProfileString(ls_Arquivo_INI, as_Ambiente, "DBPass", "")
	lt_SQLCA.LogId     = ProfileString(ls_Arquivo_INI, as_Ambiente, "LogId", "")
	lt_SQLCA.LogPass   = ProfileString(ls_Arquivo_INI, as_Ambiente, "LogPass", "")

	Return lt_SQLCA
end function
```

```pascal
// --- Arquivo app_config.ini ---
[producao]
DBMS=ODBC
Database=db_vendas
UserID=admin
DBPass=12345
LogId=app_user
LogPass=abcde
```

### 📈 Benefícios da Refatoração

- Facilita a troca de ambiente sem alterar o código-fonte.
- Melhora a segurança ao evitar que credenciais fiquem expostas no código.
- Centraliza o gerenciamento de configurações, reduzindo duplicação.
- Permite alterar conexões e caminhos dinamicamente em tempo de execução.
- Torna o sistema mais flexível, configurável e aderente a boas práticas corporativas.

[Voltar ao início](#sumário)

---

<a id="unmanaged-object-lifetime"></a>
## Unmanaged Object Lifetime

Criação de objetos, DataStores, DataWindows e outros recursos no PowerScript sem o devido controle de ciclo de vida (ou seja, sem um **DESTROY** correspondente). Isso gera vazamentos de memória, conexões abertas indevidamente e instabilidade da aplicação ao longo do tempo.

### 🧠 Problemas causados

- Consumo de memória desnecessário, especialmente em aplicações longas ou com múltiplas janelas abertas.
- Objetos permanecem em memória mesmo após o uso, degradando o desempenho.
- Conexões de banco de dados e recursos do sistema podem permanecer abertos.
- Aumenta o risco de falhas e crashes difíceis de rastrear.

### 🛠️ Solução/Refatoração Recomendada

Garantir que todo objeto criado com **CREATE** seja explicitamente destruído após o uso com **DESTROY**.

Utilizar blocos **TRY...FINALLY** para assegurar a liberação mesmo em caso de exceção.

Centralizar a criação e destruição de objetos em serviços de controle ou métodos auxiliares, especialmente para DataStores e _Non-Visual Objects (NVOs)_.

### 🔎 Exemplo de Código com Unmanaged Object Lifetime

```pascal
// --- Script de processamento em um botão de janela ---
n_Relatorio lnv_Relatorio

Create lnv_Relatorio

lnv_Relatorio.of_Gerar_Relatorio("mensal")

// Nenhum DESTROY é chamado — o objeto permanece em memória
```

Neste exemplo, o objeto _lnv_Relatorio_ é criado e utilizado, mas **nunca destruído**. Se esse script for executado várias vezes, cada instância permanece em memória, consumindo recursos até o fechamento do aplicativo.

### ✨ Exemplo Refatorado

```pascal
// --- Script de processamento em um botão de janela ---
n_Relatorio lnv_Relatorio

Try
	Create lnv_Relatorio
	lnv_Relatorio.of_Gerar_Relatorio("mensal")
Finally
	If IsValid(lnv_Relatorio) Then
		Destroy lnv_Relatorio
	End If
End Try
```

```pascal
// --- Non-Visual Object: n_Relatorio ---

public function integer of_Gerar_Relatorio (String as_Tipo)
	DataStore lds_Relatorio
	
	lds_Relatorio = Create DataStore
	
	lds_Relatorio.DataObject = "d_relatorio_" + as_Tipo
	lds_Relatorio.SetTransObject(SQLCA)
	lds_Relatorio.Retrieve()
	
	// Simula geração de relatório
	MessageBox("Relatório", "Relatório " + as_Tipo + " gerado com sucesso!")
	
	Destroy(lds_Relatorio)
	
	Return 1
end function
```

### 📈 Benefícios da Refatoração

- Garante liberação de memória e recursos após o uso.
- Evita vazamentos e degradação de desempenho em longas sessões.
- Melhora a estabilidade e previsibilidade da aplicação.
- Facilita o rastreamento de erros e a depuração.
- Segue boas práticas de gerenciamento de ciclo de vida de objetos em PowerBuilder.

[Voltar ao início](#sumário)

---

<a id="sql-embedded-script"></a>
## SQL Embedded in Script

Instruções SQL podem ser escritas diretamente dentro dos scripts PowerScript (como eventos, botões ou funções em janelas). Essa prática mistura lógica de negócio com acesso a dados, reduz a reutilização e torna a manutenção do código mais complexa e propensa a erros.

### 🧠 Problemas causados

- Dificulta a manutenção e evolução do sistema, pois instruções SQL estão espalhadas em vários scripts.
- Viola o princípio de separação de responsabilidades (UI + lógica de banco).
- Aumenta o risco de SQL Injection quando valores são concatenados diretamente.
- Torna a depuração e a portabilidade para outros bancos de dados mais difíceis.

### 🛠️ Solução/Refatoração Recomendada

Mover instruções SQL para DataWindows, DataStores ou _Non-Visual Objects (NVOs)_ especializados em acesso a dados.
Usar binding de variáveis (via argumentos ou parâmetros) em vez de concatenar strings SQL diretamente.

### 🔎 Exemplo de Código com SQL Embedded in Script

```pascal
// --- Script de um botão para atualizar o status de um pedido ---
String ls_IdPedido, ls_SQL

ls_IdPedido = sle_Pedido.Text

ls_SQL = "UPDATE PEDIDOS SET STATUS = 'Enviado' WHERE ID = " + ls_IdPedido

EXECUTE IMMEDIATE :ls_SQL USING SQLCA;

If SQLCA.SQLCode = 0 Then
	MessageBox("Sucesso", "Pedido atualizado com sucesso.")
Else
	MessageBox("Erro", "Falha ao atualizar pedido.")
End If
```

Neste exemplo, o SQL está **diretamente embutido** no script da interface. Se o sistema tiver várias telas que manipulam o mesmo tipo de informação, haverá duplicação de código SQL em diversos lugares.

### ✨ Exemplo Refatorado

```pascal
// --- Script de um botão para atualizar o status de um pedido ---
n_Servico_Pedido lnv_Pedido

Create lnv_Pedido

Try
	// Apenas delega para um serviço especializado
	If lnv_Pedido.of_Atualizar_Status(sle_Pedido.Text, "Enviado") = 1 Then
		MessageBox("Sucesso", "Pedido atualizado com sucesso.")
	Else
		MessageBox("Erro", "Falha ao atualizar pedido.")
	End If
Finally
	Destroy lnv_Pedido
End Try
```

```pascal
// --- Non-Visual Object: n_Servico_Pedido ---

public function integer of_Atualizar_Status (String as_IdPedido, String as_Status)
	// Centraliza a lógica SQL aqui
	PREPARE SQLSA FROM "UPDATE PEDIDOS SET STATUS = ? WHERE ID = ?";
	EXECUTE SQLSA USING :as_Status, :as_IdPedido;
	
	If SQLCA.SQLCode = 0 Then
		COMMIT USING SQLCA;
		Return 1
	Else
		ROLLBACK USING SQLCA;
		Return -1
	End If
end function
```

### 📈 Benefícios da Refatoração

- Centraliza o acesso a dados, facilitando manutenção e auditoria.
- Reduz duplicação de código SQL em diferentes partes do sistema.
- Melhora a segurança, evitando SQL Injection.
- Permite reutilizar o mesmo serviço em múltiplos pontos da aplicação.
- Facilita a evolução para camadas de persistência mais sofisticadas (DataStores, ORMs, procedures).

[Voltar ao início](#sumário)

---

<a id="event-cascade-smell"></a>
## Event Cascade Smell

Ocorre quando um evento dispara outro evento de forma implícita ou encadeada, criando uma cadeia de execuções não controlada entre eventos (por exemplo, o evento _Clicked_ de um botão chama o evento _ItemChanged_ de um DataWindow, que por sua vez aciona outro evento).
Esse comportamento torna o fluxo de execução **difícil de entender, prever e depurar**.

### 🧠 Problemas causados

- O fluxo de execução fica imprevisível, dificultando a identificação da origem de erros.
- Um pequeno ajuste em um evento pode causar efeitos colaterais em várias partes da interface.
- Viola o princípio de baixo acoplamento, pois eventos passam a depender uns dos outros.
- Torna o código mais frágil e difícil de testar isoladamente.

### 🛠️ Solução/Refatoração Recomendada

Evitar chamar eventos diretamente (_TriggerEvent()_ ou _PostEvent()_) entre objetos visuais.

Extrair a lógica de negócio dos eventos para métodos dedicados em _Non-Visual Objects (NVOs)_.

Usar uma abordagem clara de delegação de responsabilidades, em que cada evento apenas aciona métodos específicos — sem depender de outros eventos para completar o fluxo.

### 🔎 Exemplo de Código com Event Cascade Smell

```pascal
// --- Evento Clicked do botão "Salvar" ---
dw_Dados.AcceptText()
dw_Dados.Event ItemChanged() // Chama explicitamente outro evento

// --- Evento ItemChanged da DataWindow ---
If dw_Dados.GetItemStatus(Row, "status", Primary!) = DataModified! Then
	dw_Dados.Event ue_Validar_Dados() // Encadeia outro evento
End If

// --- Evento ue_validar_dados() ---
If dw_Dados.GetItemString(Row, "nome") = "" Then
	MessageBox("Erro", "Campo nome é obrigatório.")
End If
```

Neste exemplo, o clique no botão dispara o evento _ItemChanged_, que por sua vez chama outro evento (_ue_Validar_Dados_).

O fluxo de execução passa a depender de múltiplas chamadas indiretas, tornando o comportamento da tela **difícil de prever e depurar**.

### ✨ Exemplo Refatorado

```pascal
// --- Evento Clicked do botão "Salvar" ---
n_Servico_Dados lnv_Dados
Create lnv_Dados

Try
	lnv_Dados.of_Salvar_Dados(dw_Dados)
Finally
	Destroy lnv_Dados
End Try
```

```pascal
// --- Non-Visual Object: n_Servico_Dados ---

public function integer of_Salvar_Dados (DataWindow adw_Dados)
	adw_Dados.AcceptText()
	
	If of_Validar_Dados(adw_Dados) = 1 Then
		adw_Dados.Update()
		COMMIT USING SQLCA;
		MessageBox("Sucesso", "Dados salvos com sucesso.")
		Return 1
	Else
		ROLLBACK USING SQLCA;
		Return -1
	End If
end function

private function integer of_Validar_Dados (DataWindow adw_Dados)
	Long ll_Row

	ll_Row = adw_Dados.GetRow()

	If adw_Dados.GetItemString(ll_Row, "nome") = "" Then
		MessageBox("Erro", "Campo nome é obrigatório.")
		Return -1
	End If

	Return 1
end function
```

Neste exemplo refatorado, o **fluxo é linear e previsível**: o evento _Clicked_ chama apenas um método (_of_Salvar_Dados_) que centraliza toda a lógica — sem dependência entre eventos.

### 📈 Benefícios da Refatoração

- Elimina dependências ocultas entre eventos.
- Facilita a leitura, depuração e manutenção do código.
- Melhora a testabilidade, pois a lógica é movida para métodos isolados.
- Aumenta a previsibilidade e reduz efeitos colaterais.
- Garante um fluxo de execução controlado e de fácil rastreio.

[Voltar ao início](#sumário)

---

<a id="duplicate-datawindow-objects"></a>
## Duplicate DataWindow Objects

Ocorre quando múltiplos objetos DataWindow diferentes implementam a mesma estrutura de dados, consulta SQL ou layout visual — geralmente com pequenas variações cosméticas.
Essa duplicação aumenta o esforço de manutenção e causa inconsistências em relatórios, formulários e telas de consulta.

### 🧠 Problemas causados

- Alterações em uma consulta SQL exigem atualizar várias DataWindows semelhantes.
- Aumenta o risco de inconsistência entre telas que exibem os mesmos dados.
- Dificulta a manutenção e evolução do sistema, especialmente em projetos grandes.
- Gera retrabalho e versões divergentes de um mesmo artefato.

### 🛠️ Solução/Refatoração Recomendada

Centralizar o uso de DataWindows reutilizáveis, criando objetos genéricos parametrizáveis ou camadas de serviço (_Non-Visual Objects (NVOs)_) que encapsulem as consultas.

Evitar criar novos objetos para pequenas variações — prefira configuração dinâmica via SetSQLSelect() e Modify().

### 🔎 Exemplo de Código com Duplicate DataWindow Objects

```pascal
// --- d_Cliente_Listagem - Exibe clientes ativos ---
SELECT IDCLIENTE, NOME, EMAIL, STATUS
FROM CLIENTE
WHERE STATUS = 'Ativo'

// --- d_Cliente_Consulta - Exibe clientes ativos também (duplicado) ---
SELECT IDCLIENTE, NOME, EMAIL, STATUS
FROM CLIENTE
WHERE STATUS = 'Ativo'

// --- d_Cliente_Relatorio - Idêntico, apenas muda a cor de fundo ---
SELECT IDCLIENTE, NOME, EMAIL, STATUS
FROM CLIENTE
WHERE STATUS = 'Ativo'
```

Esses três objetos (_d_Cliente_Listagem, d_Cliente_Consulta, d_Cliente_Relatorio_) possuem a mesma consulta SQL e estrutura, mas foram duplicados para finalidades ligeiramente diferentes. Se o nome de uma coluna mudar no banco, será necessário atualizar todas as versões manualmente.

### ✨ Exemplo Refatorado

```pascal
// --- DataWindow genérica: d_cliente_base ---
SELECT IDCLIENTE, NOME, EMAIL, STATUS
FROM CLIENTE
WHERE STATUS = :RA_STATUS
```

```pascal
// --- Non-Visual Object: n_Servico_Cliente ---

public function integer of_Carregar_Clientes (DataWindow adw_Cliente, String as_Status)
	adw_Cliente.DataObject = "d_cliente_base"
	adw_Cliente.SetTransObject(SQLCA)
	adw_Cliente.SetSQLSelect("SELECT IDCLIENTE, NOME, EMAIL, STATUS FROM CLIENTE WHERE STATUS = '" + as_Status + "'")
	adw_Cliente.Retrieve()

	Return 1
end function
```

```pascal
// --- Evento Open de uma janela (w_Cliente_Listagem) ---
n_Servico_Cliente lnv_Cliente
Create lnv_Cliente

Try
	lnv_Cliente.of_Carregar_Clientes(dw_Clientes, "Ativo")
Finally
	Destroy lnv_Cliente
End Try
```

Neste exemplo, todas as telas compartilham um único DataWindow base, parametrizando o comportamento via código. As diferenças visuais podem ser aplicadas dinamicamente por meio do método _Modify()_, enquanto alterações na consulta SQL podem ser realizadas em tempo de execução utilizando o método _SetSQLSelect()_.

### 📈 Benefícios da Refatoração

- Elimina redundância de código SQL e layout.
- Facilita a manutenção — uma alteração reflete em todos os usos.
- Reduz inconsistências e retrabalho.
- Melhora a reutilização de componentes e a padronização visual.
- Segue boas práticas de design e modularidade no desenvolvimento PowerBuilder.

[Voltar ao início](#sumário)

---

<a id="unused-event-scripts"></a>
## Unused Event Scripts

Ocorre quando eventos padrão ou personalizados (como _ue_validate, ue_refresh, Clicked, ItemChanged_, etc.) são declarados, mas nunca utilizados ou invocados no ciclo de execução da aplicação.
Esses scripts permanecem no código sem propósito, acumulando complexidade e confundindo o entendimento da lógica do sistema.

### 🧠 Problemas causados

- Polui o código com eventos inativos ou desatualizados.
- Dificulta a leitura e manutenção, pois o desenvolvedor perde tempo analisando scripts que não são mais utilizados.
- Pode gerar comportamentos inesperados caso o evento volte a ser invocado acidentalmente.
- Aumenta o tamanho do código-fonte e o risco de inconsistências em versões futuras.

### 🛠️ Solução/Refatoração Recomendada

- Remover eventos que não são mais usados.
- Consolidar lógica redundante em métodos ativos ou _Non-Visual Objects (NVOs)_.
- Utilizar revisões de código e ferramentas de análise estática para identificar scripts não referenciados.

### 🔎 Exemplo de Código com Unused Event Scripts

```pascal
// --- Evento ue_Validate() no UserObject uo_Cliente ---
MessageBox("Validação", "Validando dados do cliente...")
// (Este evento nunca é chamado em nenhum lugar do sistema)


// --- Evento ue_Refresh() no mesmo UserObject ---
dw_Dados.Retrieve()
// (Este evento também não é referenciado em nenhum outro script)


// --- Evento Clicked() do botão "Salvar" ---
n_Servico_Cliente lnv_Cliente
Create lnv_Cliente

lnv_Cliente.of_Salvar_Cliente(dw_Dados)

Destroy lnv_Cliente
```

Neste exemplo, os eventos _ue_validate()_ e _ue_refresh()_ existem, mas nunca são chamados — nem por outros eventos, nem por código externo.

### ✨ Exemplo Refatorado

```pascal
// --- UserObject uo_Cliente — apenas mantém o que é realmente utilizado ---

// --- Evento Clicked() do botão "Salvar" ---
n_Servico_Cliente lnv_Cliente
Create lnv_Cliente

Try
	lnv_Cliente.of_Validar_Cliente(dw_Dados)
	lnv_Cliente.of_Salvar_Cliente(dw_Dados)
	MessageBox("Sucesso", "Cliente salvo com sucesso.")
Finally
	Destroy lnv_Cliente
End Try
```

```pascal
// --- Non-Visual Object: n_Servico_Cliente ---

public function integer of_Validar_Cliente (DataWindow adw_Dados)
	Long ll_Row

	ll_Row = adw_Dados.GetRow()

	If adw_Dados.GetItemString(ll_Row, "nome")) = "" Then
		MessageBox("Erro", "O nome do cliente é obrigatório.")
		Return -1
	End If

	Return 1
end function

public function integer of_Salvar_Cliente (DataWindow adw_Dados)
	adw_Dados.Update()
	COMMIT USING SQLCA;
	Return 1
end function
```

Neste exemplo refatorado, os eventos não utilizados foram removidos e sua lógica essencial foi concentrada em um _Non-Visual Object (NVO)_ reutilizável, tornando o código mais limpo, rastreável e fácil de manter.

### 📈 Benefícios da Refatoração

- Reduz o volume de código e melhora a legibilidade.
- Evita confusão e comportamentos inesperados com eventos não intencionais.
- Facilita o rastreamento do fluxo de execução da interface.
- Garante que cada evento existente tem um propósito claro e ativo.
- Contribui para uma base de código mais enxuta, consistente e de fácil manutenção.

[Voltar ao início](#sumário)

---

<a id="communication-object"></a>
## Communication Object

Ocorre quando o código utiliza objetos de comunicação legados do PowerBuilder, como _SOAP_ ou _INET_, para consumo de serviços externos.
Esses objetos estão obsoletos e não são mais suportados em versões recentes do PowerBuilder, podendo causar instabilidade, falhas silenciosas e comportamento inesperado.

### 🧠 Problemas causados

- Instabilidade e erros imprevisíveis em tempo de execução.
- Dificuldade de depuração e manutenção.
- Falhas de compatibilidade ao migrar para versões novas do PowerBuilder.
- Maior risco de vulnerabilidades de segurança em conexões externas.

### 🛠️ Solução/Refatoração Recomendada

Substituir o uso de _SOAP_ e _INET_ por objetos mais modernos e suportados, como:
- _HTTPClient_ — para chamadas _REST/HTTP_ seguras e estáveis.
- _Web Service Proxy_ — para serviços _SOAP_ com suporte nativo e forte tipagem.

Essas abordagens garantem compatibilidade, melhor desempenho e maior segurança.

### 🔎 Exemplo de Código com Communication Object

```pascal
// --- Uso do objeto INET (obsoleto) ---
Inet li_Inet
String ls_HTML

li_Inet = Create Inet
li_Inet.RequestMethod = "GET"
li_Inet.ConnectToServer("https://api.meuservico.com")
ls_HTML = li_Inet.GetURL("https://api.meuservico.com/clientes")

Destroy li_Inet

MessageBox("Resultado", ls_HTML)
```

### ✨ Exemplo Refatorado

```pascal
// --- Uso do objeto HTTPClient ---
String ls_Response
Integer li_RC
HttpClient lnv_Http

lnv_Http = Create HttpClient

li_RC = lnv_Http.SendRequest("GET", "https://api.meuservico.com/clientes")
If li_RC = 1 Then
	lnv_Http.GetResponseBody(ls_Response)
	MessageBox("Resultado", ls_Response)
Else
	MessageBox("Erro", "Falha ao acessar o serviço.")
End If

Destroy lnv_Http
```

O código refatorado é **mais seguro, legível e compatível**, utilizando o objeto _HttpClient_ que é nativo e plenamente suportado.

### 📈 Benefícios da Refatoração

- Elimina dependência de APIs legadas (_INET, SOAP_).
- Melhora o tratamento de erros e a segurança nas chamadas HTTP.
- Facilita o consumo de APIs REST modernas.
- Reduz riscos de falhas e comportamento inesperado.

[Voltar ao início](#sumário)

---

<a id="public-field"></a>
## Public Field

Esse mau cheiro ocorre quando variáveis de instância são declarados como **públicos**, permitindo que qualquer código externo leia e modifique diretamente o estado interno de um objeto (_Windows, UserObjects, Non-Visual Objects — NVOs_ etc.). Em PowerScript isso costuma aparecer quando usa-se _public:_ para variáveis que deveriam ser _private:_ ou _protected:_, quebrando o encapsulamento e tornando o comportamento do sistema imprevisível.

### 🧠 Problemas causados

- Quebra do encapsulamento e violação do **Princípio da Responsabilidade Única (SRP)**.
- Aumento do acoplamento entre componentes (outros objetos passam a depender do estado interno).
- Possibilidade de alterações indevidas no estado do objeto sem validação (efeitos colaterais).
- Dificulta testes unitários e manutenção (validação espalhada pelo código).
- Potencial risco de segurança e inconsistência de dados.

### 🛠️ Solução/Refatoração Recomendada

Aplicar a refatoração **Encapsulate Field**: tornar os campos _private_ (ou _protected_) e expor acesso controlado por métodos públicos ou por operações específicas que validem e normalizem os valores.

### 🔎 Exemplo de Código com Public Field

```pascal
// --- Non-Visual Object — NVO com campos públicos ---
public type n_cliente from nonvisualobject
public string is_nome
public integer ii_idade

public subroutine of_definir_dados (string as_nome, integer ai_idade)
    is_nome = as_nome
    ii_idade = ai_idade
end subroutine
end type

// Uso em outro lugar
n_cliente lnv_cliente
lnv_cliente = create n_cliente
lnv_cliente.is_nome = "Ana"        // Acesso direto — sem validação
lnv_cliente.ii_idade = -5          // Valor inválido permitido
```

### ✨ Exemplo Refatorado

```pascal
// --- Non-Visual Object — NVO com encapsulamento ---
public type n_cliente from nonvisualobject
private string is_nome
private integer ii_idade

public subroutine of_definir_dados (string as_nome, integer ai_idade)
    // validação centralizada
    IF Trim(as_nome) = "" THEN
        RAISE EXCEPTION CreateException("Nome é obrigatório.")
    END IF
    IF ai_idade < 0 THEN
        RAISE EXCEPTION CreateException("Idade inválida.")
    END IF

    is_nome = as_nome
    ii_idade = ai_idade
end subroutine

public function string of_obter_nome()
    return is_nome
end function

public function integer of_obter_idade()
    return ii_idade
end function
end type

// Uso seguro
n_cliente lnv_cliente
lnv_cliente = create n_cliente
lnv_cliente.of_definir_dados("Ana", 30)
MessageBox("Cliente", "Nome: " + lnv_cliente.of_obter_nome() + "~r~nIdade: " + string(lnv_cliente.of_obter_idade()))
```

### 📈 Benefícios da Refatoração

- Protege o estado interno do objeto e mantém invariantes.
- Centraliza validações, evitando duplicação e comportamentos inconsistentes.
- Reduz acoplamento e efeitos colaterais; facilita refatorações futuras.
- Melhora a testabilidade e a previsibilidade do comportamento do sistema.
- Aumenta a segurança e a robustez do código.

[Voltar ao início](#sumário)

---

<a id="goto-backward-jump"></a>
## GOTO Backward Jump

O uso da instrução _GOTO_ para pular para uma linha anterior no código é um mau cheiro que deve ser evitado em PowerScript.
Esse tipo de salto cria fluxos de controle não estruturados, difíceis de entender e de manter, podendo causar confusão, loops infinitos e erros de lógica.
Em vez disso, deve-se utilizar estruturas de controle como _FOR_, _WHILE_ ou sub-rotinas que tornem o código mais previsível e legível.

### 🧠 Problemas causados

- Gera fluxo de controle não estruturado (**spaghetti code**).
- Dificulta depuração e leitura do código.
- Pode causar loops infinitos ou condições inesperadas.
- Quebra o fluxo lógico da aplicação e compromete a manutenibilidade.
- Indica ausência de abstração ou de uso adequado de laços e funções.

### 🛠️ Solução/Refatoração Recomendada

Substituir o uso de _GOTO_ por estruturas de repetição ou condicionais apropriadas (_FOR, WHILE, DO...LOOP_).
Em casos mais complexos, extrair a lógica em **funções ou métodos especializados**, evitando saltos manuais no fluxo do código.

### 🔎 Exemplo de Código com GOTO Backward Jump

```pascal
Integer li_Total, li_Iterador

li_Total = 0
li_Iterador = 1

check_items:
If li_Iterador > 10 Then GOTO end_loop

li_Total += li_Iterador
li_Iterador++

If li_Iterador <= 10 Then GOTO check_items

end_loop:

MessageBox("Resultado", "Total: " + String(li_Total))
```

### ✨ Exemplo Refatorado

```pascal
Integer li_Total, li_Iterador

li_Total = 0

For li_Iterador = 1 To 10
	li_Total += li_Iterador
Next

MessageBox("Resultado", "Total: " + String(li_Total))
```

Neste exemplo, o loop _FOR_ substitui o salto manual com _GOTO_, tornando o código estruturado, seguro e claro.

### 📈 Benefícios da Refatoração

- Elimina saltos não estruturados no fluxo do código.
- Facilita a leitura e depuração.
- Evita riscos de loops infinitos e condições não controladas.
- Melhora a manutenibilidade e previsibilidade.
- Segue princípios da programação estruturada e limpa.

[Voltar ao início](#sumário)

---

<a id="improper-use-destroy"></a>
## Improper Use of Destroy Function

Em PowerScript, o uso incorreto do _Destroy_ pode causar comportamento inconsistente na liberação de objetos.
Enquanto _Destroy(lo_objeto)_ destrói o objeto **imediatamente**, a forma _Destroy lo_objeto_ pode **demorar** para executar, deixando o objeto temporariamente ativo na memória.

### 🧠 Problemas causados

- Possibilidade de vazamento de memória ou uso indevido de objetos já destruídos.
- Inconsistência no momento em que os recursos são realmente liberados.
- Dificuldade de depuração, já que o comportamento pode variar dependendo do contexto de execução.

### 🛠️ Solução/Refatoração Recomendada

Utilizar sempre a forma _Destroy(lo_objeto)_ para garantir a destruição imediata e previsível do objeto.
Evitar o uso de _Destroy lo_objeto_, que depende da execução posterior do _garbage collector_ interno do PowerBuilder.

### 🔎 Exemplo de Código com Improper Use of Destroy Function

```pascal
n_Cliente lnv_Cliente
lnv_Cliente = Create n_Cliente

lnv_Cliente.of_Carregar_Dados()

// Uso incorreto - destruição não imediata
Destroy lnv_Cliente
```

### ✨ Exemplo Refatorado

```pascal
n_Cliente lnv_Cliente
lnv_Cliente = Create n_Cliente

lnv_Cliente.of_Carregar_Dados()

// Uso correto - destruição imediata e segura
Destroy(lnv_Cliente)
```

### 📈 Benefícios da Refatoração

- Liberação imediata e previsível da memória.
- Reduz risco de _memory leaks_ e inconsistências.
- Melhora a performance e estabilidade do aplicativo.

[Voltar ao início](#sumário)

---

<a id="datawindow-object-reference"></a>
## DataWindow Object Reference

Em PowerScript, o uso frequente do _.Object_ em DataWindows ou DataStores para acessar e manipular valores pode causar degradação de desempenho. Isso ocorre porque o PowerBuilder precisa resolver dinamicamente o caminho completo do objeto a cada acesso, especialmente dentro de loops, o que consome mais memória e tempo de execução.

### 🧠 Problemas causados

- Redução de performance devido à resolução dinâmica de propriedades a cada chamada.
- Maior consumo de memória e lentidão perceptível em loops ou grandes volumes de dados.
- Código mais propenso a erros em tempo de execução, pois as referências via _.Object_ não são verificadas em tempo de compilação.
- Dificulta manutenção e depuração, já que erros podem surgir apenas durante a execução.

### 🛠️ Solução/Refatoração Recomendada

Evitar o uso direto de _.Object_ sempre que possível. Prefira utilizar os métodos _GetItem()_ e _SetItem()_ para buscar ou definir valores nas colunas. Esses métodos são mais seguros, verificáveis e oferecem melhor desempenho, especialmente em operações repetitivas ou críticas.

### 🔎 Exemplo de Código com DataWindow Object Reference

```pascal
// --- Exemplo com problema de performance ---
Integer li_Row
Decimal lde_Total

For li_Row = 1 To dw_Vendas.RowCount()
	// Uso repetido de .Object em loop
	lde_Total += dw_Vendas.Object.valor[li_Row] * dw_Vendas.Object.quantidade[li_Row]
Next
```

Neste exemplo, cada acesso a _dw_Vendas.Object_ força o PowerBuilder a resolver o caminho do objeto dinamicamente, o que gera overhead em loops grandes.

### ✨ Exemplo Refatorado

```pascal
// --- Exemplo otimizado usando GetItemNumber ---
Integer li_Row
Decimal lde_Total, lde_Preco, lde_Quantidade

For li_Row = 1 To dw_Vendas.RowCount()
	lde_Preco = dw_Vendas.GetItemNumber(li_Row, "preco")
	lde_Quantidade = dw_Vendas.GetItemNumber(li_Row, "quantidade")
	lde_Total += lde_Preco * lde_Quantidade
Next
```

Agora, o código é mais eficiente e seguro. Os métodos _GetItemNumber()_ realizam acesso direto aos dados sem precisar resolver o caminho _.Object_ a cada iteração, reduzindo drasticamente o tempo de execução.

### 📈 Benefícios da Refatoração

- Aumento significativo de performance em loops e operações de massa.
- Código mais limpo, seguro e fácil de manter.
- Redução de falhas em tempo de execução (por exemplo, erros de nome de coluna).
- Melhoria geral na estabilidade e previsibilidade do sistema.

[Voltar ao início](#sumário)

---

<!-- Links -->
[Catalogo PowerScript]: https://github.com/joaomello03/catalogo
