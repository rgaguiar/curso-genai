# Exercícios Extraclasse — Módulo 0: Fundamentos e Ambiente

**Módulo:** 0 — Fundamentos e Ambiente  
**Aula de referência:** 01  
**Entregável:** Print da conversa com o agente respondendo 5 perguntas do seu setor  

---

## Instruções Gerais

1. Use o notebook da Aula 01 sem modificar a estrutura principal
2. Troque apenas `name`, `role` e `instructions` para o seu contexto real
3. Faça as 5 perguntas do seu setor listadas abaixo
4. Tire print de cada resposta
5. Prepare uma apresentação de 2 minutos para o início da Aula 2

---

## Exercício por Setor

### RH — Assistente de Políticas de Recursos Humanos

**Configure assim:**
```python
agent = Agent(
    name="Assistente de RH",
    role="Especialista em políticas de recursos humanos da empresa",
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

**Suas 5 perguntas:**
1. Como solicito minhas férias e qual o prazo de antecedência?
2. Qual é o processo de reembolso de despesas e em quanto tempo cai na conta?
3. Meu benefício de vale-alimentação não foi creditado. O que faço?
4. Posso parcelar minhas férias? Quantas vezes?
5. Qual é o prazo para entrega dos documentos de admissão de um novo colaborador?

---

### Jurídico — Triagem de Consultas Jurídicas

**Configure assim:**
```python
agent = Agent(
    name="Triagem Jurídica",
    role="Assistente de primeiro contato para consultas jurídicas internas",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=[
        "Você faz a triagem inicial de consultas jurídicas internas.",
        "Classifique sempre a urgência: alta (prazo vencendo), média (análise necessária), baixa (informação geral).",
        "Nunca dê pareceres definitivos — encaminhe para o advogado responsável.",
        "Sempre pergunte: qual é o prazo e qual o valor envolvido?",
        "Mantenha linguagem formal e precisa.",
    ],
    api_key=GROQ_API_KEY,
    markdown=True,
)
```

**Suas 5 perguntas:**
1. Preciso revisar um contrato de prestação de serviços. Por onde começo?
2. Um fornecedor quer incluir uma cláusula de reajuste automático. É comum?
3. Temos uma notificação extrajudicial com prazo de 5 dias. O que fazemos?
4. Como funciona o processo de due diligence para aquisição de empresa?
5. Precisamos adequar os contratos à LGPD. Quais cláusulas são essenciais?

---

### Marketing — Assistente de Briefing de Campanha

**Configure assim:**
```python
agent = Agent(
    name="Assistente de Briefing",
    role="Especialista em captação de informações para campanhas de marketing",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=[
        "Você coleta informações para briefings de campanha.",
        "Sempre pergunte: público-alvo, prazo, orçamento e objetivo principal.",
        "Organize as respostas em formato de briefing estruturado ao final.",
        "Se faltarem informações essenciais, pergunte antes de avançar.",
        "Use linguagem próxima, dinâmica e orientada a resultado.",
    ],
    api_key=GROQ_API_KEY,
    markdown=True,
)
```

**Suas 5 perguntas:**
1. Preciso de uma campanha para o lançamento de um produto B2B. Por onde começo o briefing?
2. Qual a diferença entre campanha de awareness e campanha de conversão?
3. Como defino o tom de voz de uma marca que quer parecer mais acessível?
4. Temos orçamento limitado: qual canal priorizo, redes sociais ou email marketing?
5. Como estruturo um briefing para uma agência externa sem perder controle criativo?

---

### Analytics — Assistente de Dúvidas do Dia a Dia de Dados

**Configure assim:**
```python
agent = Agent(
    name="Assistente de Analytics",
    role="Especialista em análise de dados e boas práticas de analytics",
    model=Groq(id="llama-3.3-70b-versatile"),
    instructions=[
        "Você é um assistente de analytics para times de dados e negócios.",
        "Explique conceitos técnicos de dados em linguagem acessível para áreas de negócio.",
        "Quando a pergunta envolver métricas, sempre pergunte: qual é o objetivo por trás do número?",
        "Diferencie claramente dados, métricas, indicadores e KPIs quando relevante.",
        "Nunca afirme que um dado está correto sem recomendar validação na fonte.",
    ],
    api_key=GROQ_API_KEY,
    markdown=True,
)
```

**Suas 5 perguntas:**
1. Qual a diferença entre média, mediana e moda? Quando usar cada uma?
2. O que é taxa de conversão e como calculo para uma campanha de e-mail?
3. Nosso dashboard mostra números diferentes do relatório do ERP. O que pode estar errado?
4. O que significa "granularidade" quando o time de dados pede para definir?
5. Preciso comparar o desempenho de dois períodos. Devo usar variação absoluta ou percentual?

---

## O Que Fazer com os Resultados

Ao final do exercício você deve ter:
- 5 prints de perguntas e respostas do agente configurado para o seu setor
- 1 observação sobre o que surpreendeu (positivo ou negativo) na resposta do agente

Na Aula 2, você vai apresentar em 2 minutos:
- A pergunta que mais te surpreendeu
- O que mudaria nas `instructions` para melhorar uma resposta que ficou aquém

---

## Dúvidas Frequentes

**O agente respondeu algo errado. O que faço?**  
Anote a pergunta e a resposta incorreta. Na Aula 2 vamos aprender como as `instructions` controlam e corrigem esse comportamento. Isso é material de aula, não um problema.

**Não consigo criar a conta no Groq.**  
Acesse o canal de suporte do grupo antes da aula. Resolvemos antes de sábado — não durante.

**O Colab desconectou e perdi o agente.**  
Role novamente todas as células do bloco 1 ao 5 em ordem. O agente é recriado em segundos.
