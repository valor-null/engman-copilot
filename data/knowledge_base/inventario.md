# Inventário da Base de Conhecimento · EngMan Copilot (DETECTA)

Como não temos acesso aos dados reais da Motiva, vamos criar os documentos marcados como **simulado** com dados fictícios, mas realistas, e identificar cada um deles como fictício. Os marcados como **real** foram tirados do briefing da Motiva ou de fontes públicas.

## Documentos

| # | Documento | Conteúdo e campos | Origem | Quantidade | Acesso |
|---|---|---|---|---|---|
| 1 | Catálogo de câmeras CFTV PTZ | TAG, fabricante, modelo, km, praça/pórtico, data de instalação, vida útil, criticidade, limite de temperatura, potência nominal | simulado | 30+ câmeras | tool `consultar_ativo` |
| 2 | Histórico de falhas do FSM | TAG, data, sintoma, causa identificada, tipo de intervenção, tempo de reparo (h), peça trocada, reincidência | simulado | 50+ registros | tool `consultar_historico_falhas` |
| 3 | Registro de alertas da DETECTA | TAG, data e hora, temperatura (°C), corrente (A), tensão (V), consumo (W), leituras consecutivas fora do limite, alerta confirmado (sim/não) | simulado | 30+ alertas | RAG |
| 4 | Plano de manutenção preventiva | Tarefa, periodicidade, responsável, tempo estimado, recursos, condição de execução e gatilhos de CBM (temperatura, consumo e R(t)) | simulado | 1 | RAG |
| 5 | Procedimentos operacionais (POPs) | Passo a passo e pré-requisitos de segurança de: inspeção preventiva (temperatura, ventilação e fonte), verificação de consumo elétrico e teste de comunicação | simulado | 3 | tool `consultar_procedimento` |
| 6 | Fichas técnicas das câmeras | Potência típica e máxima, faixa de temperatura de operação, grau de proteção (IP), alimentação | real + simulado | todos os modelos do catálogo | RAG |
| 7 | Matriz de criticidade | Câmera ou trecho × impacto na segurança viária e na disponibilidade do monitoramento → prioridade | simulado | 1 | RAG |
| 8 | Glossário de siglas | CFTV, PTZ, CCO, CBM, RCM, WCM, FSM, RGA, LDA, λ, MTBF, MTTR, R(t), OS | real | 1 | RAG |
| 9 | Normas de segurança | Trechos aplicáveis da NR-10 (eletricidade) e da NR-35 (trabalho em altura) e regras de trabalho em via | real (resumo) | 1 | RAG |
| 10 | Briefing da Motiva | Contexto do desafio, ondas de inovação e descrição do Tema 2 | real | 1 | RAG |
| 11 | Boletins de ocorrência de campo | Relato do técnico no atendimento: TAG, data, o que foi encontrado, condição do ambiente (calor, chuva) | simulado | 10+ | RAG |

**Fonte única de verdade:** o catálogo (1), o histórico (2) e os POPs (5) são lidos pelas tools, não pelo RAG, para que o mesmo dado não venha de dois lugares com versões diferentes.

**Exemplo de dado real usado na ficha técnica (6):** câmera PTZ para trânsito com consumo de 20 W em operação básica e 33 W com infravermelho ligado, operando de −40 °C a 70 °C (ficha técnica Dahua SD8A440FA-HNT).

## Como a base é usada no RAG

1. Os documentos são divididos em trechos (*chunks*) e transformados em vetores (*embeddings*).
2. Quando o engenheiro faz uma pergunta, o sistema busca os **4 trechos mais parecidos** e, quando houver TAG na pergunta, filtra pelos trechos daquela câmera.
3. Esses trechos entram no contexto junto com o system prompt, e o assistente responde citando de qual documento cada informação veio.

## Tamanho dos trechos (chunk)

- **Documentos de texto** (POPs, plano, normas, briefing, boletins): trechos de **300 a 500 tokens**, com sobreposição de 50 tokens.
- **Documentos em tabela** (alertas, fichas técnicas): **1 registro por trecho**, para a busca trazer exatamente a câmera certa.

**Por quê:** 1 página tem cerca de 500 tokens (Aula 02). Com 4 trechos de até 500 tokens, o contexto recuperado fica em torno de 2.000 tokens, sobrando espaço na janela de contexto para o system prompt e o histórico da conversa. Trechos muito grandes misturariam câmeras diferentes; trechos muito pequenos cortariam um passo do procedimento ao meio.

## Metadados de cada trecho

Guardados junto com cada trecho, para permitir busca filtrada:

`tag` · `tipo_documento` · `data` · `km` · `criticidade` · `origem` (real ou simulado)

## O que NÃO entra na base (LGPD)

- Nome, matrícula ou qualquer dado pessoal de empregados e técnicos (nos boletins, o técnico aparece só como "Técnico A").
- Placas de veículos e imagens de ocupantes captadas pelas câmeras.
- Contratos, valores de arrecadação e qualquer dado comercial sensível da Motiva.
- Dados reais da Motiva apresentados como reais: todo dado operacional é simulado e identificado como tal.
