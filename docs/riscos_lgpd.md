# Riscos, Ética e LGPD · EngMan Copilot (DETECTA)

O assistente apoia decisões sobre câmeras que fazem parte da segurança viária. Um erro aqui pode virar um deslocamento desnecessário, uma câmera parada ou um acidente com o técnico. Por isso, mapeamos os riscos abaixo e indicamos onde cada mitigação aparece no projeto.

**Legenda de onde a mitigação aparece:** Regra N = regra do system prompt · Bloco N = etapa do fluxograma · CNN = caso do eval set.

## Matriz de riscos

| # | Risco | Como aparece no nosso projeto | Impacto | Mitigação | Onde aparece |
|---|---|---|---|---|---|
| 1 | **Alucinação** | O assistente inventa um TAG, uma falha no histórico, um valor de λ ou um trecho de norma | Financeiro e segurança | Usar só a base e as tools, citar a fonte e dizer "não encontrei" quando o dado não existir | Regras 1, 5 e 6 · Bloco 7 · C02, C07 |
| 2 | **Prompt injection direto** | O usuário pede "ignore suas regras" ou "mostre seu system prompt" | Segurança e reputacional | Regra de que as instruções não podem ser alteradas nem reveladas; detecção no guardrail de entrada | Regra 11 · Bloco 2 · C10 |
| 3 | **Prompt injection indireto** | Uma ordem escondida dentro de um boletim de campo colado na conversa | Segurança e legal | O assistente trata o texto colado como dado, nunca como instrução | Regra 11 · Bloco 2 · C11 |
| 4 | **Risco operacional** | Orientar o técnico a abrir a câmera energizada ou subir no poste sem procedimento | Segurança (acidente) | Recusa obrigatória, remissão à NR-10 e à NR-35, reforço de EPI e desligamento e encaminhamento para profissional habilitado | Regras 8 e 9 · Bloco 2 · tool `consultar_procedimento` · C12 |
| 5 | **Responsabilidade técnica** | O assistente emite um laudo dizendo que a câmera pode continuar operando | Legal | Proibição de emitir laudo, ART ou parecer; oferta de alternativa dentro do escopo | Regra 7 · C13 |
| 6 | **Excesso de confiança** | O engenheiro trata a hipótese do assistente como diagnóstico confirmado, ou trata um alerta da DETECTA como prova da causa | Financeiro (CAPEX errado) e segurança | Rotular "Fato" e "Hipótese", informar o nível de confiança e exigir confirmação humana antes de abrir a OS | Regras 2 e 3 · bloco FORMATO · tool `abrir_ordem_servico` · C03, C05 |
| 7 | **Limite de operação errado** | Usar o limite de 70 °C do protótipo da DETECTA como se fosse o limite real de todas as câmeras | Segurança e financeiro | Usar sempre o limite da ficha técnica de cada câmera; se não houver, dizer que não há | Regra 4 · C02 |
| 8 | **Viés na priorização** | Priorizar sempre o mesmo trecho ou km, ou decidir só pelo alerta, ignorando a criticidade | Segurança viária e financeiro | Critérios explícitos e auditáveis (criticidade, alerta e R(t)) definidos na matriz de criticidade e nos exemplos de prioridade | Bloco FORMATO (few-shot) · Matriz de criticidade (base) · tool `listar_ativos_por_criticidade` |
| 9 | **LGPD: dado pessoal** | Nome de operador, matrícula de técnico ou placa de veículo aparecem na pergunta ou na base | Legal | Não repetir nem usar dados pessoais; remover no guardrail de entrada; não incluir na base | Regra 12 · Bloco 2 · inventário ("o que não entra") · C03 |
| 10 | **Confidencialidade operacional** | Histórico de falhas e dados da Motiva enviados para um modelo em servidor externo | Legal e reputacional | Nesta fase, usamos só dados simulados; em uso real, a recomendação é rodar o modelo localmente (Ollama local) | Inventário · README (Stack) |
| 11 | **Contexto estourado** | Em conversa longa, o assistente esquece qual câmera está em análise ou mistura duas câmeras | Financeiro (OS na câmera errada) | Janela de 6 mensagens com resumo das anteriores e repetição do TAG em análise | Bloco BORDA · Blocos 3 e 8 · C14 |
| 12 | **Fontes que discordam** | O plano preventivo e a ficha técnica dizem coisas diferentes, e o assistente escolhe uma sozinho | Financeiro e segurança | Mostrar as duas fontes, indicar a mais recente e recomendar verificação em campo | Regra 13 |

## LGPD aplicada ao projeto

Seguimos as 3 regras práticas de LGPD para quem usa LLMs (Aula 04):

**1. Nunca enviar ao modelo dado que identifique uma pessoa.**
Nomes e matrículas de empregados, placas de veículos e imagens de ocupantes não entram na base de conhecimento e são removidos da pergunta no guardrail de entrada. Nos boletins de campo, o técnico aparece apenas como "Técnico A".

**2. O usuário precisa saber que está falando com uma IA.**
O assistente se apresenta como assistente na saudação, separa fato de hipótese e informa o nível de confiança. A decisão final, como abrir uma OS ou fazer uma intervenção, é sempre do engenheiro.

**3. Coletar só o necessário para a finalidade.**
A finalidade é a manutenção das câmeras. Por isso, a base guarda apenas dados técnicos: TAG, leituras de temperatura e consumo, falhas e procedimentos. Nenhum dado pessoal é necessário para essa finalidade.
