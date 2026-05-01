# Aula Prática: Primeiro Agente — Construindo um Triador de E-mails Executivos

**Módulo:** 1 — Primeiro Agente  
**Aula:** 02  
**Versão testada:** agno==1.4 · groq==0.11 · gradio==4.x  
**Última revisão:** 2026-05

---

Na aula anterior você montou o ambiente, entendeu como um LLM funciona e criou seu primeiro agente em 10 linhas. Você sabe que um agente recebe uma pergunta e devolve uma resposta — e que as `instructions` são o que define o comportamento dele.

O que ainda falta é estrutura. Aquele agente da Aula 1 funcionava, mas era genérico. Qualquer pergunta entrava, qualquer resposta saía. Num ambiente corporativo isso não serve — você precisa de um agente que tem papel definido, comportamento previsível e resposta padronizada.

Hoje você vai construir um **Triador de E-mails Executivos**: um agente que lê qualquer e-mail corporativo, classifica a urgência, identifica o tipo de demanda e rascunha uma resposta profissional — tudo em segundos, com interface visual que qualquer colega pode usar.

---

## 1. A Estrutura Formal de um Agente

Na Aula 1 você usou `name`, `role`, `model` e `instructions` sem entender exatamente o que cada um faz. Agora vamos abrir cada parâmetro e entender o papel que ele desempenha.

```
Agent
 │
 ├── name         → quem ele é (identidade)
 ├── role         → o que ele faz (função)
 ├── model        → como ele pensa (capacidade)
 └── instructions → como ele deve se comportar (regras)
```

### `name` — A Identidade

O `name` é o que aparece nos logs, nos relatórios e na interface. Ele não afeta o comportamento do agente, mas afeta como você e sua equipe se referem a ele.

| Ruim | Bom |
| --- | --- |
| `"agente1"` | `"Triador de E-mails Executivos"` |
| `"chatbot_rh_v3"` | `"Assistente de Políticas de RH"` |
| `"teste"` | `"Co-piloto Jurídico"` |

Nomeie como você nomearia um cargo interno. Se você teria vergonha de apresentar o nome para o seu diretor, mude.

### `role` — A Função

O `role` é a primeira coisa que o modelo lê antes de processar qualquer pergunta. Ele funciona como uma declaração de missão: define o contexto de quem está "falando" antes mesmo das instruções.

```python
role="Especialista em análise e triagem de comunicações corporativas"
```

Um `role` bem escrito já influencia o tom da resposta. Compare:

| `role` | Tom resultante |
| --- | --- |
| `"Assistente"` | Genérico, informal |
| `"Analista de comunicação corporativa"` | Profissional, estruturado |
| `"Especialista sênior em triagem de e-mails executivos"` | Preciso, orientado a prioridade |

### `model` — A Capacidade de Raciocínio

O modelo define o "motor" do agente. Por enquanto usamos `llama-3.3-70b-versatile` via Groq — gratuito e suficiente para triagem de texto. A partir do Módulo 2 vamos aprender quando e por que trocar.

### `instructions` — As Regras de Comportamento

Este é o parâmetro mais importante. É aqui que fica o seu conhecimento de negócio. Um agente com instruções ruins vai sempre decepcionar, independente do modelo.

> 💡 **Dica de Negócios:** Pense nas `instructions` como o manual de integração de um novo funcionário. Você não escreve "seja profissional" — você escreve "quando receber uma solicitação de prazo, sempre confirme a data antes de responder". Quanto mais específico, mais previsível e útil o agente.

---

## 2. Como Escrever Boas Instructions

As `instructions` são uma lista de strings. Cada string é uma regra ou orientação. O modelo as lê em ordem antes de processar qualquer mensagem.

### Anatomia de uma Instruction eficaz

```python
instructions=[
    # 1. Papel e contexto (quem você é e para quem trabalha)
    "Você analisa e-mails corporativos para executivos com agenda cheia.",

    # 2. Comportamento esperado (o que fazer em cada situação)
    "Para cada e-mail, classifique a urgência: ALTA, MÉDIA ou BAIXA.",
    "Identifique o tipo: Solicitação, Informativo, Aprovação, Reclamação.",

    # 3. Formato de saída (como estruturar a resposta)
    "Sempre responda com: Urgência, Tipo, Resumo em 1 frase, Resposta sugerida.",

    # 4. Limites e guardrails (o que NÃO fazer)
    "Nunca tome partido em conflitos entre pessoas. Seja neutro.",
    "Se o e-mail for pessoal ou sensível, classifique como ALTA e avise o executivo.",
]
```

### Os Erros Mais Comuns

| Erro | Exemplo ruim | Exemplo correto |
| --- | --- | --- |
| Instrução vaga | `"Seja profissional"` | `"Use linguagem formal. Evite gírias e abreviações."` |
| Falta de formato | `"Analise o e-mail"` | `"Responda sempre com: Urgência / Tipo / Resumo / Resposta"` |
| Sem limite de escopo | _(sem guardrail)_ | `"Se o e-mail não for corporativo, informe que está fora do escopo."` |
| Instrução contraditória | `"Seja breve" + "Detalhe tudo"` | Escolha um comportamento padrão e seja consistente |

> ⚠️ **Importante:** O modelo tenta seguir todas as instruções ao mesmo tempo. Instruções contraditórias geram comportamento imprevisível. Se você perceber que o agente ora faz X ora faz Y, há uma contradição nas suas instruções.

---

## 3. A Arquitetura do Projeto

```
Executivo cola o e-mail na interface
              │
              ▼
        Gradio (interface)
              │
              ▼
         Agent.run(email)
              │
         ┌────┴────┐
         │  Agent  │
         │  name   │
         │  role   │← define quem analisa
         │  model  │← Groq / llama-3.3-70b
         │  instr. │← regras de triagem
         └────┬────┘
              │
              ▼
       Groq API (inferência)
              │
              ▼
     Resposta estruturada:
     - Urgência: ALTA / MÉDIA / BAIXA
     - Tipo: Solicitação / Aprovação / etc.
     - Resumo: 1 frase
     - Resposta sugerida: rascunho pronto
              │
              ▼
   Aparece na interface para o executivo
```

---

## 4. Pré-requisitos

| Requisito | Status |
| --- | --- |
| Conta Groq com chave de API configurada nos Secrets | Da Aula 1 |
| Google Colab aberto com sessão ativa | Da Aula 1 |
| Exercício extraclasse da Aula 1 feito | Recomendado |

---

## 5. Construindo o Projeto Bloco a Bloco

### [00:00 – 00:20] Encantamento: Veja Funcionando Antes de Entender

Cole este código completo e rode. Depois de ver funcionando, vamos entender cada parte.

```python
# ===============================
# PROJETO COMPLETO — RODE PRIMEIRO
# ===============================

!pip install agno groq gradio -q

from google.colab import userdata
from agno.agent import Agent
from agno.models.groq import Groq
import gradio as gr

agent = Agent(
    name="Triador de E-mails Executivos",
    role="Especialista em análise e triagem de comunicações corporativas",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=[
        "Você analisa e-mails corporativos recebidos por executivos.",
        "Para cada e-mail, responda SEMPRE neste formato:",
        "**Urgência:** ALTA (resposta em 2h) / MÉDIA (24h) / BAIXA (72h)",
        "**Tipo:** Solicitação | Informativo | Aprovação necessária | Reclamação | Outro",
        "**Resumo:** Uma frase descrevendo o assunto principal.",
        "**Resposta sugerida:** Rascunho profissional pronto para enviar.",
        "Seja objetivo. O executivo não tem tempo para análises longas.",
        "Se o e-mail não for corporativo, classifique como BAIXA e informe.",
    ],
    api_key=userdata.get("GROQ_API_KEY"),
    markdown=True,
)

def analisar_email(email):
    if not email.strip():
        return "Cole um e-mail no campo acima para analisar."
    resposta = agent.run(f"Analise este e-mail:\n\n{email}")
    return resposta.content

with gr.Blocks(title="Triador de E-mails") as interface:
    gr.Markdown("# Triador de E-mails Executivos")
    gr.Markdown("Cole o e-mail recebido e obtenha análise de urgência e rascunho de resposta.")
    email_input = gr.Textbox(
        label="E-mail recebido",
        placeholder="Cole aqui o corpo do e-mail...",
        lines=10,
    )
    analisar_btn = gr.Button("Analisar E-mail", variant="primary")
    output = gr.Markdown(label="Análise")
    analisar_btn.click(fn=analisar_email, inputs=email_input, outputs=output)

interface.launch(share=True)
```

Teste com este e-mail de exemplo:

```
De: Carlos Mendes <c.mendes@fornecedor.com>
Assunto: Contrato de renovação — prazo vence sexta-feira

Prezado,

Segue em anexo a proposta de renovação do nosso contrato de serviços. 
O prazo para assinatura é sexta-feira, dia 5. Caso não haja manifestação, 
vamos considerar o contrato encerrado e iniciar processo de transição com 
outro parceiro.

Atenciosamente,
Carlos Mendes
Diretor Comercial
```

---

### [00:20 – 00:50] Contexto de Negócio: O Custo do E-mail Mal Gerenciado

**O problema:**

Um executivo brasileiro recebe em média 120 e-mails por dia. Destes, estudos de produtividade mostram que 40% são informativos (não exigem ação), 35% são solicitações de baixa urgência e apenas 25% exigem resposta rápida. O problema: sem triagem, todos chegam com a mesma aparência de urgência.

**O que acontece sem triagem:**

| Situação | Consequência |
| --- | --- |
| E-mail urgente fica enterrado entre informativos | Prazo perdido, relacionamento danificado |
| Tempo gasto lendo e-mails de baixa prioridade | Até 2,5h/dia de trabalho desperdiçado |
| Respostas redigidas sem contexto claro | Comunicação imprecisa, retrabalho |

**Quem já usa isso:**

- Secretárias executivas em grandes consultorias usam critérios de triagem manual há décadas — o agente automatiza exatamente esse processo
- Plataformas como Superhuman e Microsoft Copilot já têm triagem por IA nos e-mails corporativos
- Escritórios de advocacia usam triagem de urgência para distribuição de casos entre advogados

> 💡 **Dica de Negócios:** Este agente não substitui o executivo — ele devolve 2 horas por dia para que o executivo use onde realmente importa. O rascunho de resposta não precisa ser enviado como está: ele reduz o esforço cognitivo de começar do zero.

---

### [00:50 – 01:10] Conceito Técnico: Prompt Engineering

**1 conceito novo nesta aula:** engenharia de prompt.

**Prompt** é tudo o que o modelo recebe como entrada: as `instructions`, o `role`, a mensagem do usuário. **Prompt engineering** é a habilidade de estruturar essa entrada para obter a saída que você quer.

É a habilidade mais valiosa deste curso. Código muda. Modelos mudam. A capacidade de instruir um agente com precisão é transferível para qualquer ferramenta de IA que você vai usar pelo resto da carreira.

#### As três alavancas do prompt

```
┌─────────────────────────────────────────────────────┐
│                    PROMPT COMPLETO                  │
│                                                     │
│  [SYSTEM]  ← instructions + role                   │
│  Define quem o agente é e como ele se comporta     │
│                                                     │
│  [USER]    ← mensagem que chega em agent.run()     │
│  O conteúdo que o agente vai processar             │
│                                                     │
│  [ASSISTANT] ← resposta gerada                     │
│  O que o agente devolve                            │
└─────────────────────────────────────────────────────┘
```

#### Técnicas que funcionam para público de negócios

| Técnica | Como aplicar | Efeito |
| --- | --- | --- |
| **Formato explícito** | `"Responda SEMPRE neste formato: X / Y / Z"` | Resposta previsível e fácil de ler |
| **Exemplos no prompt** | `"Exemplo de ALTA urgência: contrato vencendo hoje"` | Calibra o que "urgência" significa |
| **Persona de negócio** | `"Você é uma secretária executiva com 15 anos de experiência"` | Tom mais preciso e contextual |
| **Restrição explícita** | `"Nunca invente informações que não estão no e-mail"` | Reduz alucinações |

> ⚠️ **Importante:** O modelo não lembra de uma chamada para outra. Cada vez que o usuário envia um e-mail, o agente lê as `instructions` do zero. Por isso as instruções precisam ser completas e autossuficientes — o agente não aprende com os erros da sessão anterior.

---

### [01:10 – 03:00] Construção Bloco a Bloco

Abra um novo notebook. Vamos reconstruir o projeto entendendo cada decisão.

---

#### Bloco 1: Instalação e Chave

```python
# ===============================
# BLOCO 1 — INSTALAÇÃO E CHAVE
# ===============================

!pip install agno groq gradio -q

from google.colab import userdata
GROQ_API_KEY = userdata.get("GROQ_API_KEY")
print("Ambiente pronto.")
```

---

#### Bloco 2: Definindo as Instructions com Precisão

```python
# ===============================
# BLOCO 2 — INSTRUCTIONS
# ===============================
# Separamos as instructions para facilitar a edição e o versionamento
# Cada item da lista é uma regra independente

INSTRUCTIONS_TRIAGEM = [
    # Contexto e papel
    "Você analisa e-mails corporativos recebidos por executivos.",

    # Formato de saída — explícito e estruturado
    "Para cada e-mail, responda SEMPRE neste formato:",
    "**Urgência:** ALTA (resposta em 2h) / MÉDIA (24h) / BAIXA (72h)",
    "**Tipo:** Solicitação | Informativo | Aprovação necessária | Reclamação | Outro",
    "**Resumo:** Uma frase descrevendo o assunto principal.",
    "**Resposta sugerida:** Rascunho profissional pronto para enviar.",

    # Restrições e guardrails
    "Seja objetivo. O executivo não tem tempo para análises longas.",
    "Nunca invente informações que não estão no e-mail.",
    "Se o e-mail não for corporativo, classifique como BAIXA e informe o executivo.",
]

print(f"{len(INSTRUCTIONS_TRIAGEM)} instruções carregadas.")
```

Separar as instructions em uma variável tem duas vantagens: você pode ajustar sem tocar no restante do código, e pode versionar as mudanças (guardar qual versão funcionou melhor).

**Checkpoint 1:** Rodou sem erro? Avance.

---

#### Bloco 3: Criando o Agente

```python
# ===============================
# BLOCO 3 — O AGENTE
# ===============================

from agno.agent import Agent
from agno.models.groq import Groq

agent = Agent(
    name="Triador de E-mails Executivos",
    role="Especialista em análise e triagem de comunicações corporativas",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=INSTRUCTIONS_TRIAGEM,
    api_key=GROQ_API_KEY,
    markdown=True,
)

print(f"Agente '{agent.name}' pronto.")
```

**Checkpoint 2:** Apareceu `Agente 'Triador de E-mails Executivos' pronto.`?

---

#### Bloco 4: Teste Direto com E-mail Real

```python
# ===============================
# BLOCO 4 — TESTE DIRETO
# ===============================
# Antes da interface, valide que o agente funciona como esperado

email_teste = """
De: Carlos Mendes <c.mendes@fornecedor.com>
Assunto: Contrato de renovação — prazo vence sexta-feira

Prezado,

Segue em anexo a proposta de renovação do nosso contrato de serviços. 
O prazo para assinatura é sexta-feira, dia 5. Caso não haja manifestação, 
vamos considerar o contrato encerrado e iniciar processo de transição com 
outro parceiro.

Atenciosamente,
Carlos Mendes
Diretor Comercial
"""

resposta = agent.run(f"Analise este e-mail:\n\n{email_teste}")
print(resposta.content)
```

Leia a resposta com atenção. O agente classificou como ALTA? Identificou que há prazo envolvido? O rascunho de resposta tem o tom certo? Se algo ficou fora do esperado, a correção vai estar nas `instructions`.

**Checkpoint 3:** A urgência foi ALTA e o resumo mencionou prazo? Avance.

---

#### Bloco 5: Interface com Gradio Blocks

```python
# ===============================
# BLOCO 5 — INTERFACE GRADIO BLOCKS
# ===============================
# gr.Blocks dá mais controle sobre o layout do que gr.Interface
# É a evolução natural — mesmo princípio, mais flexibilidade

import gradio as gr

def analisar_email(email):
    if not email.strip():
        return "Cole um e-mail no campo acima para analisar."
    resposta = agent.run(f"Analise este e-mail:\n\n{email}")
    return resposta.content

with gr.Blocks(title="Triador de E-mails") as interface:
    gr.Markdown("# Triador de E-mails Executivos")
    gr.Markdown("Cole o e-mail recebido e obtenha análise de urgência e rascunho de resposta.")

    email_input = gr.Textbox(
        label="E-mail recebido",
        placeholder="Cole aqui o corpo do e-mail...",
        lines=10,
    )
    analisar_btn = gr.Button("Analisar E-mail", variant="primary")
    output = gr.Markdown(label="Análise")

    analisar_btn.click(fn=analisar_email, inputs=email_input, outputs=output)

interface.launch(share=True)
```

A diferença entre `gr.Interface` (Aula 1) e `gr.Blocks` (esta aula):

| | `gr.Interface` | `gr.Blocks` |
| --- | --- | --- |
| Simplicidade | Alta — ideal para começar | Média |
| Controle de layout | Limitado | Total |
| Múltiplos inputs/outputs | Difícil | Fácil |
| Botão customizado | Não | Sim |
| A partir de qual módulo | 0–1 | 1+ |

**Checkpoint 4:** A interface abriu com o botão "Analisar E-mail"? Teste com o e-mail de Carlos Mendes.

---

#### Bloco 6: Casos de Teste — Validando o Comportamento

```python
# ===============================
# BLOCO 6 — BATERIA DE TESTES
# ===============================

emails_teste = {
    "urgente": """
        De: jurídico@empresa.com
        Assunto: URGENTE — Liminar judicial exige resposta até 18h hoje

        A empresa recebeu liminar judicial exigindo suspensão imediata de 
        campanha publicitária. Prazo para resposta: hoje às 18h. 
        Necessito de aprovação da diretoria para acionar advogado externo.
    """,
    "informativo": """
        De: ti@empresa.com
        Assunto: Manutenção programada do servidor — sábado às 2h

        Informamos que haverá manutenção programada no servidor de e-mail 
        no próximo sábado às 2h. Previsão de conclusão: 4h. 
        Não haverá impacto para usuários finais.
    """,
    "guardrail": """
        De: amigo@gmail.com
        Assunto: Vai no churrasco sábado?

        Ei! Vai no churrasco lá em casa sábado? Vai ter cerveja gelada!
    """,
}

for tipo, email in emails_teste.items():
    print(f"\n{'='*60}")
    print(f"TESTE: {tipo.upper()}")
    print('='*60)
    resposta = agent.run(f"Analise este e-mail:\n\n{email}")
    print(resposta.content)
```

O teste `"guardrail"` é o mais importante: veja como o agente trata um e-mail pessoal. Com as instructions corretas, ele deve identificar que está fora do escopo corporativo e responder adequadamente — sem inventar urgência ou tipo.

---

### [03:00 – 03:30] Personalização Guiada: Adapte para o Seu Setor

Copie o Bloco 2 (instructions) e o Bloco 3 (agente) e substitua conforme o seu setor. **Não mude mais nada.**

**RH — Triagem de Solicitações de Colaboradores:**
```python
INSTRUCTIONS_TRIAGEM = [
    "Você analisa solicitações enviadas por colaboradores ao departamento de RH.",
    "Para cada solicitação, responda SEMPRE neste formato:",
    "**Prioridade:** ALTA (prazo legal envolvido) / MÉDIA (impacta colaborador) / BAIXA (informativo)",
    "**Categoria:** Férias | Benefícios | Reembolso | Desligamento | Dúvida Geral",
    "**Resumo:** Uma frase com o que o colaborador precisa.",
    "**Orientação:** O que o colaborador deve fazer ou qual documento precisa enviar.",
    "Seja empático mas objetivo. Não prometa prazos que não pode cumprir.",
    "Se envolver questão legal (rescisão, acidente), classifique como ALTA.",
]

agent = Agent(
    name="Triador de Solicitações de RH",
    role="Analista de RH especializado em triagem de solicitações de colaboradores",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=INSTRUCTIONS_TRIAGEM,
    api_key=GROQ_API_KEY,
    markdown=True,
)
```

**Jurídico — Triagem de Consultas Jurídicas:**
```python
INSTRUCTIONS_TRIAGEM = [
    "Você faz a triagem inicial de consultas jurídicas internas da empresa.",
    "Para cada consulta, responda SEMPRE neste formato:",
    "**Urgência:** ALTA (prazo processual ou contratual) / MÉDIA (análise necessária) / BAIXA (informativo)",
    "**Área:** Trabalhista | Contratual | Tributário | Regulatório | LGPD | Outro",
    "**Resumo:** Uma frase com o tema jurídico central.",
    "**Próximo passo:** O que precisa ser feito e por quem.",
    "Nunca emita parecer jurídico definitivo — você faz triagem, não assessoria.",
    "Se houver prazo processual mencionado, sempre classifique como ALTA.",
]

agent = Agent(
    name="Triador Jurídico",
    role="Assistente de triagem de demandas jurídicas corporativas",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=INSTRUCTIONS_TRIAGEM,
    api_key=GROQ_API_KEY,
    markdown=True,
)
```

**Marketing — Triagem de Briefings e Aprovações:**
```python
INSTRUCTIONS_TRIAGEM = [
    "Você analisa solicitações recebidas pelo departamento de marketing.",
    "Para cada solicitação, responda SEMPRE neste formato:",
    "**Prazo:** URGENTE (hoje/amanhã) / NORMAL (até 1 semana) / PLANEJADO (mais de 1 semana)",
    "**Tipo:** Aprovação de Peça | Briefing | Relatório | Ajuste | Informativo",
    "**Resumo:** Uma frase com o que está sendo solicitado.",
    "**O que falta:** Informações ausentes para processar a solicitação.",
    "Se faltarem informações essenciais (prazo, verba, objetivo), liste o que falta antes de tudo.",
    "Nunca aprove peças ou campanhas — você triaga, quem aprova é a gestão.",
]

agent = Agent(
    name="Triador de Marketing",
    role="Assistente de triagem de solicitações e aprovações de marketing",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=INSTRUCTIONS_TRIAGEM,
    api_key=GROQ_API_KEY,
    markdown=True,
)
```

> 💡 **Dica de Negócios:** Perceba que o único código que muda entre os três exemplos são as `instructions`, o `name` e o `role`. A arquitetura — modelo, interface, fluxo — é idêntica. Isso é o que torna o Agno poderoso: você muda o comportamento sem mudar o código.

---

## 6. O Resultado Final

Ao final desta aula você tem:

1. **Domínio da estrutura do Agent** — sabe o que cada parâmetro faz e por quê
2. **Projeto funcional** — triador de e-mails com interface visual para qualquer usuário
3. **Habilidade de prompt engineering** — sabe escrever instructions que produzem saída previsível
4. **Versão setorial** — agente adaptado para o vocabulário da sua área

---

## 7. Expandindo a Arquitetura

Este agente tem uma limitação importante: ele não lembra o contexto entre análises. Cada e-mail é processado de forma independente. Se você analisar 10 e-mails seguidos, o agente não consegue dizer "dos 10 e-mails de hoje, 3 eram urgentes".

**Versão 2.0 — sem mudar a arquitetura:**

| Melhoria | Como implementar | Em qual módulo |
| --- | --- | --- |
| Memória de sessão | Agente lembra e-mails analisados na mesma conversa | Módulo 2 |
| Histórico persistente | Log dos e-mails analisados salvos em banco | Módulo 2 |
| Busca de política interna | Agente consulta PDF de normas antes de responder | Módulo 3 |
| Relatório diário automático | Resume os e-mails do dia e envia por e-mail | Módulo 5 |

---

## 8. Próxima Aula

Na **Aula 3** vamos adicionar **guardrails formais e controle de temperatura** — e o projeto será um assistente de redação de comunicados internos com tom configurável (formal, neutro, empático). Você vai aprender a controlar não só o que o agente faz, mas como ele faz.

> **Leve para a Aula 3:** o agente desta aula adaptado para o seu setor. Vamos usar ele como base para adicionar os guardrails.

---

## Exercício Extraclasse

**Entregável:** Link do Colab rodando com o agente adaptado para o seu departamento

**Instruções:**
1. Use o notebook desta aula sem modificar a estrutura
2. Troque as `instructions`, `name` e `role` para o contexto real do seu departamento
3. Crie 5 e-mails ou solicitações fictícias (mas realistas) do seu setor
4. Analise os 5 e tire prints dos resultados
5. Identifique 1 resposta que ficou aquém e anote o que mudaria nas instructions

**Por setor:**

| Setor | Tipo de mensagem para testar |
| --- | --- |
| **RH** | Solicitação de férias, reembolso, atestado, rescisão, benefício |
| **Jurídico** | Consulta contratual, prazo processual, LGPD, rescisão de contrato |
| **Marketing** | Briefing de campanha, aprovação de peça, ajuste de copy, relatório |
| **Analytics** | Consulta de apólice, notificação de sinistro, renovação, cobertura |

---

## Gabarito: Código Completo

```python
# ===============================
# AULA 02 — PRIMEIRO AGENTE
# Triador de E-mails Executivos com Gradio Blocks
# ===============================

# BLOCO 1 — INSTALAÇÃO E CHAVE
!pip install agno groq gradio -q

from google.colab import userdata
GROQ_API_KEY = userdata.get("GROQ_API_KEY")

# BLOCO 2 — INSTRUCTIONS
INSTRUCTIONS_TRIAGEM = [
    "Você analisa e-mails corporativos recebidos por executivos.",
    "Para cada e-mail, responda SEMPRE neste formato:",
    "**Urgência:** ALTA (resposta em 2h) / MÉDIA (24h) / BAIXA (72h)",
    "**Tipo:** Solicitação | Informativo | Aprovação necessária | Reclamação | Outro",
    "**Resumo:** Uma frase descrevendo o assunto principal.",
    "**Resposta sugerida:** Rascunho profissional pronto para enviar.",
    "Seja objetivo. O executivo não tem tempo para análises longas.",
    "Nunca invente informações que não estão no e-mail.",
    "Se o e-mail não for corporativo, classifique como BAIXA e informe.",
]

# BLOCO 3 — O AGENTE
from agno.agent import Agent
from agno.models.groq import Groq

agent = Agent(
    name="Triador de E-mails Executivos",
    role="Especialista em análise e triagem de comunicações corporativas",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=INSTRUCTIONS_TRIAGEM,
    api_key=GROQ_API_KEY,
    markdown=True,
)

# BLOCO 4 — TESTE DIRETO (opcional)
# email_teste = "..."
# print(agent.run(f"Analise este e-mail:\n\n{email_teste}").content)

# BLOCO 5 — INTERFACE GRADIO BLOCKS
import gradio as gr

def analisar_email(email):
    if not email.strip():
        return "Cole um e-mail no campo acima para analisar."
    resposta = agent.run(f"Analise este e-mail:\n\n{email}")
    return resposta.content

with gr.Blocks(title="Triador de E-mails") as interface:
    gr.Markdown("# Triador de E-mails Executivos")
    gr.Markdown("Cole o e-mail recebido e obtenha análise de urgência e rascunho de resposta.")
    email_input = gr.Textbox(
        label="E-mail recebido",
        placeholder="Cole aqui o corpo do e-mail...",
        lines=10,
    )
    analisar_btn = gr.Button("Analisar E-mail", variant="primary")
    output = gr.Markdown(label="Análise")
    analisar_btn.click(fn=analisar_email, inputs=email_input, outputs=output)

interface.launch(share=True)
```
