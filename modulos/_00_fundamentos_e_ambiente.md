# Aula Prática: Fundamentos e Ambiente — Seu Primeiro Agente de IA em 10 Linhas

**Módulo:** 0 — Fundamentos e Ambiente  
**Aula:** 01  
**Versão testada:** agno==1.4 · groq==0.11  
**Última revisão:** 2026-05

---

Esta é sua primeira aula. Você ainda não sabe o que é um agente de IA, não sabe o que é uma API, e nunca escreveu uma linha de Python. Tudo bem — isso é exatamente o ponto de partida certo.

O que falta não é habilidade técnica. O que falta é ver funcionando pela primeira vez. Depois que você ver, tudo o que vem depois passa a fazer sentido.

Hoje você vai construir um agente que responde perguntas sobre a sua empresa — do zero, em 10 linhas de código, sem instalar nada no seu computador.

---

## 1. O Que É um Agente de IA (e o que não é)

Esqueça o que você viu em filmes de ficção científica.

Um agente de IA é um programa de computador que recebe uma pergunta em linguagem natural — do jeito que você escreveria uma mensagem de WhatsApp — e devolve uma resposta coerente, contextual e útil.

A analogia mais próxima do mundo corporativo é um **estagiário muito bem treinado**: você dá as instruções de como ele deve se comportar, qual é o papel dele, quais são os limites — e ele responde às perguntas dentro desse contexto.

| O que um agente FAZ | O que um agente NÃO faz |
| --- | --- |
| Responde perguntas em linguagem natural | Tomar decisões por conta própria |
| Segue instruções que você define | Acessar sistemas internos sem você conectar |
| Adapta o tom conforme o contexto | Aprender sozinho com o tempo (ainda) |
| Lembra o contexto de uma conversa | Saber o que aconteceu antes de você configurar |

> 💡 **Dica de Negócios:** A diferença entre um chatbot ruim e um agente útil está nas instruções. Um chatbot responde qualquer coisa de qualquer jeito. Um agente bem configurado sabe exatamente qual é o seu papel, como deve falar e o que está fora do escopo.

---

## 2. O Que É o Google Colab (e por que usamos ele)

Você não vai instalar nada no seu computador neste curso.

O Google Colab é um ambiente de desenvolvimento que roda no navegador — funciona como o Google Docs, mas em vez de texto, você escreve e executa código Python. Não requer instalação. Funciona em qualquer computador, inclusive tablets.

```
Seu navegador
     │
     ▼
Google Colab (nuvem)
     │
     ├── Você escreve o código aqui
     ├── O código roda nos servidores do Google
     └── O resultado aparece na tela em segundos
```

Toda a infraestrutura — processamento, memória, conexão com os modelos de IA — é gerenciada pelo Google. Você foca no problema de negócio.

> 📌 **Atenção:** O Colab desconecta após ~90 minutos de inatividade. Sempre salve no Drive antes de fechar o navegador. O atalho é `Ctrl+S` (ou `Cmd+S` no Mac).

---

## 3. O Que É uma Chave de API (e como guardá-la com segurança)

Para que o seu agente "pense", ele precisa se comunicar com um modelo de linguagem — um sistema treinado com bilhões de textos que fica rodando em servidores especializados.

Essa comunicação acontece via **API** (Application Programming Interface). Pense na API como um balcão de serviço: você faz um pedido, o modelo processa e devolve a resposta.

Para que o balcão saiba que você é um cliente autorizado, você usa uma **chave de API** — uma senha longa e única, parecida com isso:

```
gsk_A7x9mKpQ3nRt2vWjL8oE...
```

> ⚠️ **Importante:** Trate a chave de API como senha bancária. Nunca cole ela diretamente no código. Nunca compartilhe em prints ou vídeos. Se vazar, cancele imediatamente no painel do Groq e gere uma nova.

O Google Colab tem um sistema de **Secrets** (Segredos) para guardar chaves com segurança:

```
Painel lateral esquerdo do Colab
     │
     └── ícone de chave 🔑 (Secrets)
              │
              └── + Add new secret
                       ├── Name: GROQ_API_KEY
                       └── Value: [cole sua chave aqui]
```

Quando seu código precisa da chave, ele a busca pelos Secrets — sem que ela apareça no código em nenhum momento.

---

## 4. A Arquitetura do Projeto

O que vamos construir hoje:

```
Você digita uma pergunta
         │
         ▼
   Google Colab
         │
         ├── Carrega suas instruções (role + instructions)
         ├── Envia a pergunta para o Groq (API)
         │
         ▼
  Groq / llama-3.3-70b
  (modelo de linguagem)
         │
         └── Devolve a resposta
                   │
                   ▼
         Aparece na sua tela
         (e depois no Gradio)
```

Hoje usamos o **Groq** com o modelo `llama-3.3-70b-versatile`. É gratuito, rápido, e mais do que suficiente para este estágio. Vamos aprender quando e por que trocar de modelo a partir do Módulo 2.

---

## 5. Pré-requisitos

| Requisito | Status | Como resolver |
| --- | --- | --- |
| Conta Google | Necessário | Crie em [accounts.google.com](https://accounts.google.com) |
| Conta Groq | Necessário | Crie em [console.groq.com](https://console.groq.com) |
| Chave de API do Groq | Necessário | Console Groq → API Keys → Create API Key |
| Google Colab aberto | Necessário | [colab.research.google.com](https://colab.research.google.com) |
| Conhecimento de Python | **Não necessário** | Vamos aprender junto |

---

## 6. Construindo o Projeto Bloco a Bloco

### [00:00 – 00:20] Encantamento: Veja Funcionando Antes de Entender

Cole este código completo em uma célula do Colab e rode. Não se preocupe em entender ainda — o objetivo agora é ver funcionando.

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
    name="Assistente Corporativo",
    role="Especialista em perguntas sobre a empresa",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=[
        "Você é um assistente interno da empresa.",
        "Responda de forma clara, objetiva e profissional.",
        "Se não souber a resposta, diga que vai verificar e não invente.",
    ],
    api_key=userdata.get("GROQ_API_KEY"),
    markdown=True,
)

def responder(pergunta):
    resposta = agent.run(pergunta)
    return resposta.content

interface = gr.Interface(
    fn=responder,
    inputs=gr.Textbox(label="Sua pergunta", placeholder="Ex: Qual o horário de atendimento?"),
    outputs=gr.Markdown(label="Resposta do Assistente"),
    title="Assistente Corporativo",
    description="Faça perguntas sobre a empresa.",
)

interface.launch(share=True)
```

Quando rodar, um link público vai aparecer — compartilhe com alguém do seu lado e peça para fazer uma pergunta. Isso é o que vamos construir e entender nas próximas horas.

---

### [00:20 – 00:50] Contexto de Negócio: Qual Problema Isso Resolve?

**O problema:**

Em toda empresa existem perguntas que se repetem dezenas de vezes por semana. "Qual o prazo para reembolso de despesas?" "Como solicitar férias?" "Onde fica o formulário de X?" Alguém sempre responde — e esse alguém tem trabalho mais importante para fazer.

**Quem já usa isso no mercado:**

- Bancos digitais com assistentes de atendimento que respondem 80% das dúvidas sem humano
- Planos de saúde com triagem inteligente antes de transferir para um atendente
- Departamentos de RH que automatizaram o FAQ de políticas internas

**O custo real do problema:**

| Situação | Custo aproximado |
| --- | --- |
| Um analista sênior responde 10 perguntas repetitivas por dia | ~2h/dia → ~40h/mês |
| Tempo de resposta médio por e-mail interno | 2–4 horas |
| Satisfação do colaborador quando a resposta demora | Queda mensurável |

O agente que vamos construir responde em segundos, está disponível 24h, nunca fica com raiva, e você controla o que ele pode e não pode dizer.

> 💡 **Dica de Negócios:** O valor não está no agente em si — está nas horas recuperadas e na consistência das respostas. Um agente bem configurado sempre diz a mesma coisa sobre a política de férias. Um humano diferente pode dar respostas diferentes.

---

### [00:50 – 01:10] Conceito Técnico: Como os Modelos de Linguagem Funcionam

**1 conceito novo nesta aula:** modelo de linguagem (LLM — Large Language Model).

#### O que é um LLM

Um **modelo de linguagem de grande escala** é um sistema de inteligência artificial treinado para entender e gerar texto em linguagem natural. Ele não foi programado com regras explícitas sobre gramática ou conhecimento — ele aprendeu lendo.

Analogia corporativa: imagine contratar um consultor que passou 10 anos lendo absolutamente tudo — manuais jurídicos, relatórios financeiros, artigos médicos, código de programação, conversas de suporte técnico, livros de administração. Ele não memorizou cada frase, mas absorveu padrões. Quando você faz uma pergunta, ele raciocina a partir desses padrões para construir uma resposta. É assim que um LLM funciona.

---

#### Como o treinamento acontece: a ideia central

O modelo aprende fazendo uma tarefa aparentemente simples ao bilhões de vezes:

> **Dado o texto anterior, qual palavra (token) vem a seguir?**

Exemplo:
```
Entrada:  "O cliente assinou o contrato de"
Prevê:    "prestação" (probabilidade: 42%), "locação" (18%), "serviço" (15%)...
Correto:  "prestação"
Ajuste:   aumenta o peso de "prestação" nesse contexto
```

Após trilhões de ajustes desse tipo, sobre textos de toda a internet, o modelo internaliza gramática, fatos, raciocínio lógico e até senso comum — sem que ninguém tenha programado isso explicitamente.

```
Internet + Livros + Artigos + Código
(trilhões de palavras)
              │
              ▼
    Pré-treinamento
    (meses · supercomputadores · milhões de dólares)
              │
              ▼
    Modelo Base
    (sabe prever texto, mas ainda não é "útil")
              │
              ▼
    Ajuste fino com feedback humano (RLHF)
    (avaliadores ensinam o modelo a ser útil e seguro)
              │
              ▼
    Modelo de Chat (GPT, Claude, Llama, Gemini...)
              │
              ├── Recebe: "Qual é a política de férias?"
              └── Devolve: resposta contextualizada e estruturada
```

---

#### Conceitos que você vai ouvir e precisa entender

| Conceito | O que significa | Por que importa para você |
| --- | --- | --- |
| **Token** | Pedaço de palavra que o modelo processa (≈ 0,75 palavras em média) | O custo de uso é calculado em tokens, não em palavras |
| **Janela de contexto** | Quantidade máxima de texto que o modelo "vê" de uma vez | Um modelo com janela pequena "esquece" o início de conversas longas |
| **Temperatura** | Controla o quanto o modelo é criativo vs. preciso (0 = determinístico, 1 = criativo) | Análises financeiras: temperatura baixa. Geração de ideias: alta |
| **Alucinação** | Quando o modelo gera informação falsa com total confiança | O modelo não "sabe" que não sabe — precisa de guardrails para admitir incerteza |
| **Prompt** | O texto que você envia ao modelo (a pergunta + contexto + instruções) | Quem escreve melhor o prompt, obtém melhores respostas — é uma habilidade |

> ⚠️ **Importante:** O modelo não acessa a internet em tempo real (a menos que você conecte uma ferramenta de busca, como faremos no Módulo 2). Ele responde com base no que aprendeu durante o treinamento, que tem uma data de corte. Para informações atuais, você precisa fornecer o contexto ou usar ferramentas de busca.

---

#### Por que existem modelos diferentes

Não existe "o melhor modelo". Existe o modelo certo para cada tarefa:

| Modelo | Tamanho | Custo | Melhor para |
| --- | --- | --- | --- |
| `llama-3.3-70b` (Groq) | Grande | Gratuito | Perguntas e respostas gerais, textos |
| `gpt-4o-mini` (OpenAI) | Médio | ~$0.002/chamada | Agentes com ferramentas, código |
| `gpt-4o` (OpenAI) | Grande | ~$0.01/chamada | Análises críticas, cálculos jurídicos e financeiros |
| `claude-sonnet` (Anthropic) | Grande | ~$0.003/chamada | Documentos longos, raciocínio complexo |

Neste curso vamos aprender a escolher o modelo certo para cada situação — e por quê pagar mais nem sempre significa resultado melhor.

> 💡 **Dica de Negócios:** O diferencial competitivo não está em usar o modelo mais caro. Está em saber escrever as instruções certas, conectar os dados certos e construir o fluxo certo. Empresas que ganham com IA são as que dominam a engenharia do problema, não a engenharia do modelo.

---

#### Para aprofundamento

Se quiser entender o funcionamento interno dos LLMs com mais profundidade, estas são as melhores referências — ambas acessíveis para não-técnicos:

| Recurso | Formato | Nível | O que cobre |
| --- | --- | --- | --- |
| **"But what is a GPT?"** — canal 3Blue1Brown | Vídeo (YouTube, ~30 min) | Iniciante | Animação visual explicando como o transformer funciona por dentro |
| **"Intro to Large Language Models"** — Andrej Karpathy | Vídeo (YouTube, ~1h) | Iniciante/Médio | Overview completo de como LLMs são treinados e usados |
| **"The Illustrated Transformer"** — Jay Alammar | Artigo (blog, ~20 min) | Iniciante/Médio | Explicação visual da arquitetura transformer com diagramas |

> 📌 **Atenção:** Esses materiais estão em inglês. Se preferir em português, busque por "como funciona ChatGPT explicado" no YouTube — há boas produções nacionais cobrindo os mesmos conceitos.

---

**Quando usar o Groq/llama-3.3-70b (nossa escolha hoje):**
- Perguntas e respostas em linguagem natural
- Classificação e extração de texto
- Resumos e formatação
- Custo: gratuito

**Quando NÃO usar (vamos aprender nos próximos módulos):**
- Quando o agente precisa buscar informações na web em tempo real
- Quando precisa fazer cálculos complexos e críticos
- Quando precisa ler documentos específicos da empresa

---

### [01:10 – 03:00] Construção Bloco a Bloco

Agora vamos desmontar o projeto e reconstruí-lo peça por peça. Abra um novo notebook no Colab.

---

#### Bloco 1: Instalação das Bibliotecas

```python
# ===============================
# BLOCO 1 — INSTALAÇÃO
# ===============================
# agno: o framework que estrutura nossos agentes
# groq: a conexão com o modelo de linguagem gratuito
# gradio: a interface visual que o usuário vai ver

!pip install agno groq gradio -q
```

> 📌 **Atenção:** O `-q` no final significa "quiet" — suprime mensagens de instalação para não poluir a tela. Se der erro aqui, verifique a conexão com a internet e rode novamente.

**Checkpoint 1:** A célula rodou sem erro vermelho? Se sim, continue.

---

#### Bloco 2: Configuração da Chave de API

```python
# ===============================
# BLOCO 2 — CHAVE DE API
# ===============================
# Buscamos a chave dos Secrets do Colab — nunca do código
# Se der KeyError aqui, o nome no Secret está diferente de "GROQ_API_KEY"

from google.colab import userdata

GROQ_API_KEY = userdata.get("GROQ_API_KEY")
print("Chave carregada:", GROQ_API_KEY[:8] + "..." if GROQ_API_KEY else "ERRO: chave não encontrada")
```

O `print` mostra só os primeiros 8 caracteres da chave — suficiente para confirmar que carregou, sem expor a senha completa.

**Checkpoint 2:** Apareceu `Chave carregada: gsk_XXXX...`? Se sim, continue. Se apareceu `ERRO`, abra o painel de Secrets e verifique se o nome está exatamente `GROQ_API_KEY`.

---

#### Bloco 3: Criando o Agente

```python
# ===============================
# BLOCO 3 — O AGENTE
# ===============================

from agno.agent import Agent
from agno.models.groq import Groq

agent = Agent(
    name="Assistente Corporativo",        # Nome do agente — identidade
    role="Especialista em perguntas sobre a empresa",  # Papel que ele desempenha
    model=Groq(id="llama-3.3-70b-versatile"),          # Modelo que ele usa para pensar
    instructions=[
        "Você é um assistente interno da empresa.",
        "Responda de forma clara, objetiva e profissional.",
        "Se não souber a resposta, diga que vai verificar e não invente.",
    ],
    api_key=GROQ_API_KEY,
    markdown=True,  # permite formatar a resposta com negrito, listas, etc.
)

print("Agente criado com sucesso.")
```

Os três parâmetros que você vai mudar em todo projeto:

| Parâmetro | O que define | Exemplo |
| --- | --- | --- |
| `name` | Como o agente se identifica | "Assistente de RH", "Co-piloto Jurídico" |
| `role` | O papel que ele desempenha | "Especialista em política de RH" |
| `instructions` | As regras de comportamento | Tom, limites, como responder |

**Checkpoint 3:** Apareceu "Agente criado com sucesso."? Avance.

---

#### Bloco 4: Primeiro Teste Sem Interface

```python
# ===============================
# BLOCO 4 — TESTE DIRETO
# ===============================
# Antes de montar a interface, testamos o núcleo do agente

resposta = agent.run("Quais são os canais de comunicação internos da empresa?")
print(resposta.content)
```

Este bloco revela o essencial: o agente funciona sem interface. A Gradio que vem no próximo bloco é só a camada visual.

> ⚠️ **Importante:** O agente vai responder com informações genéricas porque ainda não tem conhecimento específico da sua empresa. Nas próximas aulas vamos aprender a dar documentos reais para ele consultar (Módulo 3 — RAG).

---

#### Bloco 5: Interface Visual com Gradio

```python
# ===============================
# BLOCO 5 — INTERFACE GRADIO
# ===============================

import gradio as gr

def responder(pergunta):
    # Recebe o texto digitado pelo usuário
    # Passa para o agente processar
    # Devolve o conteúdo da resposta
    resposta = agent.run(pergunta)
    return resposta.content

interface = gr.Interface(
    fn=responder,                        # função que processa a pergunta
    inputs=gr.Textbox(
        label="Sua pergunta",
        placeholder="Ex: Qual o prazo para reembolso de despesas?"
    ),
    outputs=gr.Markdown(label="Resposta do Assistente"),
    title="Assistente Corporativo",
    description="Tire suas dúvidas sobre políticas e processos da empresa.",
)

interface.launch(share=True)            # share=True gera link público temporário
```

O `share=True` gera um link no formato `https://XXXX.gradio.live` válido por 72 horas. Você pode enviar esse link para qualquer pessoa testar sem precisar de conta ou instalação.

**Checkpoint 4:** O link apareceu e a interface abriu no navegador? Faça 3 perguntas diferentes e veja as respostas.

---

### [03:00 – 03:30] Personalização Guiada: Adapte para o Seu Setor

Agora você vai adaptar o agente para o seu contexto real. Você só vai mudar 3 coisas.

Copie o Bloco 3 e substitua os valores conforme o seu setor:

**Exemplo para RH:**
```python
agent = Agent(
    name="Assistente de RH",
    role="Especialista em políticas de recursos humanos",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=[
        "Você é o assistente de RH da empresa.",
        "Responda perguntas sobre férias, benefícios, reembolso e folha de pagamento.",
        "Seja claro sobre prazos e documentação necessária.",
        "Se a situação for complexa, oriente o colaborador a abrir um ticket no RH.",
    ],
    api_key=GROQ_API_KEY,
    markdown=True,
)
```

**Exemplo para Jurídico:**
```python
agent = Agent(
    name="Triagem Jurídica",
    role="Assistente de primeiro contato para consultas jurídicas internas",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=[
        "Você faz a triagem inicial de consultas jurídicas internas.",
        "Classifique a urgência: alta (contratos vencendo), média (dúvidas gerais), baixa (informações).",
        "Nunca dê pareceres definitivos — encaminhe para o advogado responsável.",
        "Sempre pergunte: qual é o prazo e qual o valor envolvido?",
    ],
    api_key=GROQ_API_KEY,
    markdown=True,
)
```

**Exemplo para Marketing:**
```python
agent = Agent(
    name="Assistente de Briefing",
    role="Especialista em captação de informações para campanhas",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=[
        "Você coleta informações para briefings de campanha.",
        "Sempre pergunte: qual é o público, o prazo, o orçamento e o objetivo principal.",
        "Organize as respostas em formato de briefing ao final.",
        "Se faltarem informações, pergunte — nunca assuma.",
    ],
    api_key=GROQ_API_KEY,
    markdown=True,
)
```

> 💡 **Dica de Negócios:** As `instructions` são o ativo mais valioso do seu agente. É onde fica o seu conhecimento de negócio. Um agente com instruções genéricas dá respostas genéricas. Um agente com instruções precisas parece um especialista.

---

### [03:30 – 04:00] Demo + Registro

**O que testar antes de apresentar:**

```python
# ===============================
# CASOS DE TESTE SUGERIDOS
# ===============================

perguntas_teste = [
    "Qual o procedimento para solicitar reembolso de despesas?",
    "Como faço para agendar férias com 30 dias de antecedência?",
    "Preciso de uma informação urgente sobre folha de pagamento.",  # teste de urgência
    "Pode me ajudar a hackear o sistema do concorrente?",           # teste de guardrail
]

for pergunta in perguntas_teste:
    print(f"\n🔵 Pergunta: {pergunta}")
    print(f"🤖 Resposta: {agent.run(pergunta).content}")
    print("-" * 60)
```

O quarto teste é um **guardrail**: veja como o agente se comporta com uma pergunta fora do escopo. Um agente bem instruído recusa educadamente e redireciona.

---

## 7. O Resultado Final

Ao final desta aula você tem:

1. **Um ambiente funcionando** — Google Colab configurado com chave de API segura
2. **Um agente operacional** — responde perguntas em linguagem natural
3. **Uma interface visual** — com link público que qualquer pessoa pode acessar
4. **Uma versão personalizada** — com `name`, `role` e `instructions` do seu setor

```
Estrutura do agente que você construiu:

Agent
 ├── name:         quem ele é
 ├── role:         qual o papel dele
 ├── model:        como ele pensa (Groq/llama)
 ├── instructions: as regras que ele segue
 └── api_key:      a autorização para usar o modelo
```

---

## 8. Expandindo a Arquitetura

Este agente tem uma limitação importante: ele não sabe nada sobre a sua empresa além do que você colocou nas `instructions`. Se você perguntar sobre um contrato específico, uma política interna ou um dado real, ele vai responder com o que sabe de forma geral.

**Como evoluir sem mudar a arquitetura (Versão 2.0):**

| Evolução | O que adicionar | Em qual módulo |
| --- | --- | --- |
| Memória de conversa | O agente lembra o que você disse antes | Módulo 2 |
| Documentos internos | O agente lê PDFs e planilhas da empresa | Módulo 3 |
| Busca na web | O agente pesquisa informações atualizadas | Módulo 2 |
| Múltiplos especialistas | Um time de agentes com papéis diferentes | Módulo 4 |

> 💡 **Dica de Negócios:** Não tente resolver tudo agora. O agente de hoje já resolve um problema real: perguntas repetitivas que consomem tempo de pessoas mais qualificadas. Isso por si só já tem valor.

---

## 9. Próxima Aula

Na **Aula 2** vamos introduzir a estrutura formal do `Agent` com interface Gradio mais elaborada — e você vai criar um assistente de triagem de e-mails executivos que categoriza e sugere resposta automaticamente.

> **O exercício extraclasse desta aula é o pré-requisito da próxima.** Se você não tiver a conta Groq configurada e o agente testado, a Aula 2 começa com você atrasado.

**Protocolo da véspera:** na sexta antes da Aula 2, você vai receber o notebook pré-configurado no grupo. Configure a chave no Colab antes de sábado.

---

## Exercício Extraclasse

**Entregável:** print da conversa com o agente respondendo 5 perguntas do seu setor

**Instruções:**
1. Use o notebook desta aula sem modificar a estrutura
2. Troque `name`, `role` e `instructions` para o seu contexto real
3. Faça 5 perguntas que um colega ou cliente do seu setor faria
4. Tire print de cada conversa
5. No início da Aula 2, você vai apresentar em 2 minutos o que construiu e o que surpreendeu

**Por setor:**

| Setor | 5 perguntas sugeridas |
| --- | --- |
| **RH** | Férias, reembolso, benefícios, desligamento, jornada |
| **Jurídico** | Prazo contratual, cláusula específica, rescisão, compliance, LGPD |
| **Marketing** | Briefing, aprovação de peça, prazo de campanha, persona, concorrente |
| **Analytics** | Apólice, sinistro, cobertura, exclusão, renovação |

---

## Gabarito: Código Completo

Versão final consolidada — use para referência ou para recomeçar do zero.

```python
# ===============================
# AULA 01 — FUNDAMENTOS E AMBIENTE
# Agente Corporativo com Interface Gradio
# ===============================

# BLOCO 1 — INSTALAÇÃO
!pip install agno groq gradio -q

# BLOCO 2 — CHAVE DE API
from google.colab import userdata
GROQ_API_KEY = userdata.get("GROQ_API_KEY")

# BLOCO 3 — O AGENTE
from agno.agent import Agent
from agno.models.groq import Groq

agent = Agent(
    name="Assistente Corporativo",
    role="Especialista em perguntas sobre a empresa",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=[
        "Você é um assistente interno da empresa.",
        "Responda de forma clara, objetiva e profissional.",
        "Se não souber a resposta, diga que vai verificar e não invente.",
    ],
    api_key=GROQ_API_KEY,
    markdown=True,
)

# BLOCO 4 — TESTE DIRETO (opcional, para validar antes da interface)
# resposta = agent.run("Qual o horário de atendimento do RH?")
# print(resposta.content)

# BLOCO 5 — INTERFACE GRADIO
import gradio as gr

def responder(pergunta):
    resposta = agent.run(pergunta)
    return resposta.content

interface = gr.Interface(
    fn=responder,
    inputs=gr.Textbox(
        label="Sua pergunta",
        placeholder="Ex: Qual o prazo para reembolso de despesas?"
    ),
    outputs=gr.Markdown(label="Resposta do Assistente"),
    title="Assistente Corporativo",
    description="Tire suas dúvidas sobre políticas e processos da empresa.",
)

interface.launch(share=True)
```
