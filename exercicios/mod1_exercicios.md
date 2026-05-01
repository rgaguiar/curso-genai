# Exercícios Extraclasse — Módulo 1: Primeiro Agente

**Módulo:** 1 — Primeiro Agente  
**Aulas de referência:** 02 e 03  
**Entregável:** Link do Colab rodando com o agente adaptado para o seu departamento  

---

## Exercício da Aula 2 — Triador de Mensagens do Seu Setor

Use o notebook da Aula 2 sem modificar a estrutura. Troque apenas `instructions`, `name` e `role`.

### RH — Triador de Solicitações de Colaboradores

**5 situações para testar:**
1. Colaborador solicita adiantamento de férias com 10 dias de prazo
2. Gestor pede orientação sobre processo de desligamento sem justa causa
3. Colaborador relata assédio moral por parte de colega
4. Novo contratado pergunta sobre prazo para receber o crachá
5. Gestor solicita relatório de horas extras do time para fecha mento de folha

**O que observar:**
- O agente classificou a situação de assédio como ALTA?
- A orientação foi empática sem prometer o que não pode cumprir?
- O agente pediu mais informações quando o contexto era insuficiente?

---

### Jurídico — Triador de Consultas Internas

**5 situações para testar:**
1. Área comercial pergunta se pode usar logotipo de cliente em material de vendas
2. Fornecedor notifica rescisão contratual com prazo de 30 dias
3. Colaborador pergunta sobre regras de LGPD para envio de newsletter
4. Gestor solicita análise de cláusula de não-concorrência em contrato de trabalho
5. Departamento financeiro pergunta prazo de guarda de documentos fiscais

**O que observar:**
- O agente se absteve de emitir parecer definitivo?
- Identificou corretamente quais situações têm prazo legal?
- Indicou claramente o próximo passo (quem deve resolver)?

---

### Marketing — Triador de Solicitações da Área

**5 situações para testar:**
1. Agência envia briefing de campanha para aprovação sem informar o orçamento
2. Gerente comercial pede ajuste urgente em banner digital para hoje
3. Diretoria solicita relatório de performance da campanha do último trimestre
4. Designer pergunta qual versão do logo usar em fundo escuro
5. Parceiro pede co-branding sem descrever o contexto da peça

**O que observar:**
- O agente listou as informações faltantes antes de classificar?
- Distinguiu corretamente o que é urgente do que é apenas marcado como urgente?
- Manteve tom adequado para comunicação interna?

---

### Analytics — Triador de Solicitações de Dados

**5 situações para testar:**
1. Gestor comercial pede um dashboard de vendas por região "para ontem" sem especificar métricas
2. Área de marketing solicita acesso à base de clientes completa sem justificar o uso
3. Diretoria pede análise ad-hoc comparando dois períodos, mas não define quais indicadores comparar
4. Analista reporta que os números do relatório semanal não batem com o sistema de origem
5. RH solicita cruzamento de dados de turnover com avaliação de desempenho sem informar a granularidade

**O que observar:**
- O agente pediu as informações faltantes antes de classificar a urgência?
- Tratou a inconsistência de dados como alta prioridade?
- Sinalizou a solicitação de acesso à base completa como algo que precisa de validação?

---

## O Que Entregar na Aula 3

1. Link do Colab com o agente rodando (ou print da interface com resultados)
2. Anotação de 1 resposta que ficou aquém e o que você mudaria nas `instructions`
3. Preparado para apresentar em 2 minutos: "o que funcionou e o que não funcionou"

---

## Dúvidas Frequentes

**O agente está inventando informações que não estavam no e-mail.**  
Adicione nas instructions: `"Nunca invente informações que não estão na mensagem recebida."` Se persistir, adicione: `"Se a informação não estiver explícita, diga que não é possível determinar com base no que foi enviado."`

**O agente sempre classifica tudo como ALTA urgência.**  
As instructions precisam dar critérios mais claros para cada nível. Adicione exemplos: `"ALTA: prazo legal ou contratual mencionado. MÉDIA: impacta operação mas sem prazo. BAIXA: informativo ou sem ação necessária."`

**A interface do Gradio não abre.**  
Verifique se o Colab ainda está conectado (o círculo no canto superior direito deve estar verde). Se desconectou, rode todas as células novamente na ordem.
