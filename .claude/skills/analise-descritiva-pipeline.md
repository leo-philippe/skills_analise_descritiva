---
name: analise-descritiva-pipeline
description: >
  Pipeline completo de análise descritiva — do arquivo bruto ao dashboard interativo
  ou relatório analítico completo. Conduz levantamento de requisitos profissional,
  Data Profiling, ciclo EDA e Framework U-B-M, com seção final de metodologia em
  linguagem simples e lúdica.

  Ativar quando o usuário disser: "analise meus dados", "quero um dashboard",
  "análise descritiva", "faz uma EDA", "gera um relatório dos meus dados",
  "quero entender minha base", "cria um dashboard da minha área",
  "analise isso para mim", "explore esse arquivo", "preciso de um relatório analítico".
  Also trigger for: "analyze my data", "create a dashboard", "exploratory analysis",
  "data profiling", "descriptive report", "business analysis from data".
---

# Pipeline de Análise Descritiva

Pipeline modular em 9 fases que combina três metodologias acadêmicas —
Data Profiling (Rahm & Do, 2000), EDA (Tukey, 1977) e Framework U-B-M
(Peat & Barton, 2005) — com os 7 grupos de métricas e visualizações da
análise descritiva. Qualquer pessoa de qualquer área deve conseguir usar.

---

## Regras Globais de Eficiência

Seguir sempre, sem exceção:

1. **PDF com margem e padding zero** _(regra mais importante para geração de PDF)_:
   Todo PDF gerado via `reportlab` deve usar `topMargin=0, bottomMargin=0, leftMargin=0, rightMargin=0`
   no `SimpleDocTemplate`. O hero header usa `colWidths=[largura_total]` para ser full-bleed.
   O conteúdo interno aplica padding via `LEFTPADDING`/`RIGHTPADDING` dentro de `TableStyle`
   (valor padrão: 40 pts). Nunca usar margens no documento para "empurrar" conteúdo — sempre
   usar padding interno nas Tables.

2. **Scratchpad obrigatório**: No início, defina o SCRATCH path conforme o sistema operacional:
   - **Windows**: `%LOCALAPPDATA%\Temp\claude\analise_{slug}\`
     (equivalente Python: `os.path.join(os.environ['LOCALAPPDATA'], 'Temp', 'claude', f'analise_{slug}')`)
   - **macOS/Linux**: `/tmp/claude/analise_{slug}/`
   - **Alternativa universal**: `.scratch/analise_{slug}/` dentro do diretório do projeto

   Crie o diretório se não existir. Toda saída detalhada vai para arquivos nesse path.

3. **Verbosidade limitada**: As Fases 4, 5 e 6 imprimem no chat no máximo **20 linhas**.
   Todo output bruto, tabelas longas e dados intermediários vão para arquivo no SCRATCH.
   Cite apenas os 5–7 números mais relevantes da fase no chat.

4. **Checkpointing**: Cada fase termina gravando `SCRATCH/fase_{N}_ok.json`
   com as métricas-chave e um campo `"status": "completo"`.
   O arquivo substitui o dado no contexto para as fases seguintes —
   leia o arquivo, não reconstitua a partir do histórico.

5. **Modo retomada**: Ao iniciar, verifique se `SCRATCH/fase_3_ok.json` existe.
   Se sim: "Encontrei análise anterior. Deseja **retomar** a partir de qual fase
   ou **reiniciar** do zero?" Aguarde resposta antes de prosseguir.

6. **Subagente para grandes volumes**: Se o arquivo tiver > 50.000 linhas,
   use o `Agent` tool para executar as Fases 4, 5 e 6.
   O subagente recebe o caminho dos arquivos e o SCRATCH como instrução,
   executa toda a análise pesada, escreve os `fase_{N}_ok.json` e retorna
   apenas um parágrafo de sumário. O contexto principal recebe só o sumário.

7. **Modularidade**: Cada fase é independente. O usuário pode dizer
   "refaça a fase 5" ou "pule para a fase 7" e o pipeline deve obedecer,
   lendo os arquivos de checkpoint das fases anteriores para ter contexto.

---

## Fase 0 — Inicialização

1. Identifique o nome do projeto (da área, do arquivo, ou pergunte em uma linha).
2. Detecte o SO e defina o SCRATCH path (ver Regra 2).
3. Verifique se há checkpoints anteriores.
4. Se não houver, confirme: "Iniciando análise para [projeto]. Scratchpad em [SCRATCH]."

---

## Fase 1 — Levantamento de Requisitos

Use `AskUserQuestion` em **duas rodadas separadas**, aguardando as respostas da
primeira antes de fazer a segunda.

### Rodada 1 — Contexto e objetivo (4 perguntas simultâneas)

1. **"Qual é a sua área e o objetivo principal desta análise?"**
   Opções: Financeiro / Controladoria · Comercial / Vendas · RH / Pessoas ·
   Operações / Logística · Marketing / CRM · Outra área

2. **"Qual pergunta de negócio você quer que essa análise responda?"**
   Opções (para inspirar, aceitar Other para resposta livre):
   "Como está minha performance atual?" · "Onde estão os principais gargalos?" ·
   "Quem são meus melhores clientes/produtos?" · "O que mudou nos últimos meses?"

3. **"Quem vai receber esta análise?"**
   Opções: Só eu mesmo (uso pessoal) · Minha equipe · Gestores e diretores ·
   Apresentação para toda a empresa

4. **"Que tipo de dado você tem disponível?"**
   Opções: Planilha Excel ou CSV (vou anexar ou colar os dados) ·
   Dados exportados de um sistema · Vou descrever os dados verbalmente ·
   Outro formato

### Rodada 2 — Dados (3 perguntas, após Rodada 1)

5. **"Qual é o período coberto pelos seus dados?"**
   Opções: Último mês · Últimos 3 meses · Último ano · Mais de 1 ano · Não sei

6. **"Quantos registros (linhas) aproximadamente seus dados têm?"**
   Opções: Menos de 500 · Entre 500 e 5.000 · Entre 5.000 e 50.000 · Mais de 50.000

7. **"Quais são os campos (colunas) principais que você tem?"**
   Campo aberto — o usuário descreve os campos. Se preferir, leia o arquivo diretamente
   usando o Read tool e infira os campos sem perguntar.

Após a Rodada 2:
- Grave `SCRATCH/fase_1_ok.json` com: área, objetivo, audiência, tipo_dado, período, volume, campos.
- Confirme em uma linha: "Requisitos salvos. Montando arquitetura."

---

## Fase 2 — Arquitetura da Análise

Com base no `fase_1_ok.json`, produza e apresente um plano antes de executar.
**Adapte o escopo ao volume declarado:**

| Volume | Estratégia |
|---|---|
| < 5.000 linhas | Análise inline, todas as fases no contexto principal |
| 5.000–50.000 | Fases 4–6 com output resumido, checkpoints obrigatórios |
| > 50.000 | Fases 4–6 via Agent subagente; contexto principal recebe só sumário |

Use este template, preenchendo com as informações coletadas:

```
## Arquitetura da Análise — [Área / Projeto]

**Objetivo:** [A pergunta de negócio reformulada em uma frase clara]
**Audiência:** [Quem vai receber]
**Escopo:** [Período + volume estimado]
**Estratégia:** [Inline / Checkpoints / Subagente — conforme tabela acima]

### O que vamos analisar:

Etapa 1 · Data Profiling    → diagnóstico de qualidade dos dados
Etapa 2 · EDA (Exploração)  → padrões, anomalias, perguntas geradas
Etapa 3 · Análise U → B → M → do campo isolado ao conjunto

### Métricas e grupos previstos:
- Grupo 1 (Tendência central): [campos numéricos relevantes]
- Grupo 2 (Dispersão): [campos com variação importante]
- Grupo 5 (Frequência): [campos categóricos relevantes]
- Grupo 6 (Negócio): [métricas adaptadas à área informada]
- Grupo 7 (Gráficos): [4-6 tipos adequados ao objetivo]
```

Pergunte: "Esse plano faz sentido? Posso ajustar antes de começar."
Grave `SCRATCH/fase_2_ok.json` após aprovação.

---

## Fase 3 — Escolha do Output

Use `AskUserQuestion` com uma única pergunta:

**"Como você quer receber o resultado desta análise?"**

- **Dashboard interativo (HTML / artifact)** — Painel visual com múltiplas abas,
  gráficos interativos e navegação. Abre como link compartilhável no navegador.
  Ideal para apresentações em reuniões ou exploração contínua dos dados.
  **Formato de entrega: artifact publicado (link).**

- **Relatório em PDF** — Documento exportável gerado com Python (reportlab),
  salvo em disco no diretório do projeto. Inclui gráficos, tabelas, narrativa
  e seção de metodologia. Ideal para enviar por e-mail, arquivar ou entregar
  para gestores. **Formato de entrega: arquivo .pdf salvo localmente.**

Grave `SCRATCH/fase_3_ok.json` com o campo `"output": "dashboard"` ou `"pdf"`.
Confirme a escolha e anuncie que a análise começará agora.

---

## Fase 4 — Data Profiling

**Se volume > 50.000 linhas:** use Agent tool (ver Regra 6). Instrua o agente a:
- Ler o(s) arquivo(s) com Read/Bash
- Apurar as dimensões abaixo
- Gravar `SCRATCH/fase_4_ok.json`
- Retornar apenas um parágrafo de diagnóstico (≤ 10 linhas)

**Se volume ≤ 50.000 linhas:** execute diretamente com Read/Bash.

| Dimensão | O que verificar |
|---|---|
| Volume | Nº de linhas e colunas |
| Tipos | Numérico, categórico, data, texto livre — por coluna |
| Completude | % de nulos por coluna |
| Cardinalidade | Valores únicos por categórica |
| Range | Min, max, exemplos por numérica |
| Consistência | Datas inválidas, valores impossíveis, duplicatas |

Grave `SCRATCH/fase_4_ok.json` com o resultado estruturado.
Imprima no chat apenas uma tabela resumida (≤ 15 linhas) e qualquer callout crítico:

> ⚠️ **Atenção:** [campo X] tem [N]% de valores nulos. Vamos prosseguir assumindo
> que ausências são legítimas e informar quando isso afeta os resultados.

---

## Fase 5 — EDA (Exploração)

**Se volume > 50.000:** já executado pelo subagente da Fase 4 no mesmo Agent call.
**Se volume ≤ 50.000:** execute e grave `SCRATCH/fase_5_ok.json`.

Aplique o ciclo de Tukey pelo menos **duas vezes por campo relevante**:
Olhe → Encontre → Interprete → Pergunte → repita.

No arquivo JSON, grave para cada campo relevante:
- média, mediana, desvio padrão, p25, p75, p90, p99, mín, máx
- categoria dominante e % (para categóricas)
- assimetria percebida e outliers relevantes
- lista de perguntas geradas pela exploração

No chat: apenas as 5–7 descobertas mais relevantes em formato de lista.

---

## Fase 6 — Análise U → B → M

Leia `SCRATCH/fase_5_ok.json` para contexto.
**Se volume > 50.000:** já executado pelo subagente; leia o arquivo de checkpoint.

### Univariada (Grupos 1–5)
Para cada campo relevante ao objetivo, interprete em linguagem de negócio:
- O que é o "típico"? (tendência central)
- Quanta variação existe? É preocupante? (dispersão)
- Como os valores se distribuem? (posição e forma)

### Bivariada (Grupo 7)
Para os 2–3 pares mais relevantes ao objetivo:
- Existe relação? Em que direção? Faz sentido para o negócio?

### Multivariada (Grupo 7)
Visão combinada. Identifique padrões invisíveis em análises isoladas.

Grave `SCRATCH/fase_6_ok.json` com as conclusões estruturadas por dimensão (U/B/M).
No chat: parágrafo único com os 3–5 achados principais.

---

## Fase 7 — Geração do Output

Leia `SCRATCH/fase_3_ok.json` para saber o formato.
Leia `SCRATCH/fase_4_ok.json`, `fase_5_ok.json`, `fase_6_ok.json` para os dados.

### Princípios de Visualização (embutidos)

**Escolha da forma — a pergunta é: qual é o trabalho do dado?**

| Trabalho do dado | Forma correta |
|---|---|
| Comparar magnitudes entre categorias | Barras horizontais (valores longos) ou verticais (≤ 7 cats) |
| Evolução no tempo | Linha de área — fill ~10% opacidade, linha 2px |
| Proporção de um todo (≤ 3 partes) | Barra segmentada horizontal |
| Distribuição de uma variável | Histograma ou box plot |
| Concentração / desigualdade | Curva de Lorenz com zona Gini (âmbar, ~15% opac.) |
| Regra 80/20 | Pareto — barras + linha acumulada no mesmo eixo Y |
| Um único número-chave | Stat tile / hero number — sem gráfico |
| Relação entre duas variáveis numéricas | Scatter plot |

**Nunca:** eixo duplo Y (duas escalas no mesmo gráfico — use dois gráficos separados). Nunca pizza com mais de 3 fatias. Nunca rainbow/arco-íris como paleta sequencial.

**Cor por função:**
- **Série única** (barras, área): azul sequencial — `#0369A1` claro → escuro por magnitude
- **Múltiplas categorias**: ordem fixa — azul `#0369A1`, violeta `#6D28D9`, âmbar `#B45309`, verde `#047857`. Nunca ciclar além dessas 4; a 5ª série vai para "Outros"
- **Positivo / Negativo**: verde `#047857` e vermelho `#B91C1C` — nunca como cor de série
- **Zona Gini / desigualdade**: âmbar com ~15% de opacidade
- **Área sob curva**: azul com ~10% de opacidade
- Em modo escuro, substitua pelos equivalentes: `#38BDF8`, `#A78BFA`, `#FBB024`, `#34D399`

**Marcas e anatomia:**
- Linhas: **2px** de espessura
- Pontos em linhas: **≥ 8px**, somente em pontos-chave (início, fim, pico, mínimo)
- Barras: bordas arredondadas **4px** nas pontas livres, ancoradas na baseline
- Labels diretos: **apenas no ponto final ou no maior valor** — nunca número em cada ponto
- Grid: linhas **1px** cor `var(--border)`, sem borda no eixo Y
- Tooltip: obrigatório em charts HTML — crosshair + valor ao hover

---

### Princípios Visuais do Output HTML (embutidos)

**Sistema de tokens CSS — copie exatamente:**

```css
:root {
  --bg:#F0F4F8; --surface:#FFFFFF; --surface-2:#E4ECF4; --border:#C8D6E5;
  --text:#0F1B2D; --text-2:#4A6077; --text-3:#7A94AA;
  --accent:#0369A1; --accent-light:#DBEAFE;
  --violet:#6D28D9; --violet-light:#EDE9FE;
  --amber:#B45309; --amber-light:#FEF3C7;
  --green:#047857; --green-light:#D1FAE5;
  --shadow-sm:0 1px 2px rgba(15,27,45,.06);
  --shadow:0 2px 6px rgba(15,27,45,.08);
}
@media(prefers-color-scheme:dark){:root{
  --bg:#07101E; --surface:#0F1D30; --surface-2:#172840; --border:#223450;
  --text:#DDE8F4; --text-2:#7A9ABB; --text-3:#456077;
  --accent:#38BDF8; --accent-light:#0B2A42;
  --violet:#A78BFA; --violet-light:#1C1035;
  --amber:#FBB024; --amber-light:#281A04;
  --green:#34D399; --green-light:#052E1E;
}}
:root[data-theme="light"]{/* repete tokens light */}
:root[data-theme="dark"]{/* repete tokens dark */}
```

**Tipografia:** `font-family: system-ui, -apple-system, 'Segoe UI', sans-serif` — **nunca** carregar fontes externas (URLs bloqueadas em artifacts). Títulos: `font-weight: 800`, `letter-spacing: -0.025em`, `text-wrap: balance`. Labels e badges em UPPERCASE com `letter-spacing: 0.08em`.

**Layout:** Container principal: `max-width: 1080px; margin: 0 auto; padding: 48px 24px`. Use `display: flex; gap:` ou `display: grid; gap:` para espaçamento entre siblings — nunca `margin` individual entre cards do mesmo grupo. Tabelas e código com `overflow-x: auto` em container próprio (o `<body>` nunca deve rolar horizontalmente).

**Componentes padrão:**

```css
/* Card */
.card { background:var(--surface); border:1px solid var(--border);
        border-radius:8px; padding:18px 20px;
        box-shadow:0 1px 2px rgba(15,27,45,.06); }

/* Badge */
.badge { font-size:10.5px; font-weight:700; letter-spacing:.08em;
         text-transform:uppercase; padding:3px 8px; border-radius:4px; }

/* Nav sticky */
nav { position:sticky; top:0; z-index:100; background:var(--surface);
      border-bottom:1px solid var(--border); }

/* Tabela */
table { border-collapse:collapse; width:100%; }
thead { background:var(--surface-2); }
th { font-size:10.5px; font-weight:700; text-transform:uppercase;
     letter-spacing:.07em; color:var(--text-3); padding:10px 16px;
     border-bottom:1px solid var(--border); }
td { padding:10px 16px; border-bottom:1px solid var(--border); }
```

**Responsivo:** breakpoint `860px` — grades de 2+ colunas viram 1 coluna. Breakpoint `560px` — padding reduzido para 14px lateral.

**Ambos os temas obrigatórios:** implementar `@media(prefers-color-scheme:dark)` + overrides `:root[data-theme="dark"]` e `:root[data-theme="light"]`. O tema escuro não é inversão automática — redefinir os tokens com valores escolhidos e validar contraste.

---

### Se DASHBOARD:

Construa um artifact HTML com **abas de navegação** (tabs via JavaScript puro):

- **Aba: Visão Geral** — KPIs principais, período, totais, destaques
- **Aba: Distribuições** — Histogramas e box plots dos campos principais
- **Aba: Relações** — Gráficos bivariados mais relevantes para o objetivo
- **Aba: Frequências** — Barras e Pareto das variáveis categóricas
- **Aba: Metodologia** — Seção obrigatória (ver Fase 8)

Cada aba deve ter:
- Um título e uma frase explicando o que o usuário está vendo
- Os gráficos com legendas em linguagem de negócio (não estatística)
- Um parágrafo de interpretação abaixo de cada gráfico ou conjunto

### Se RELATÓRIO (PDF):

Gere um script Python usando `reportlab` e execute-o com:
- **Windows**: `py -3 script.py`
- **macOS/Linux**: `python3 script.py`

Salve o PDF em `{diretório_dos_dados}/relatorio_{slug_projeto}.pdf`.

**Estrutura obrigatória do PDF:**
1. **Header** — Bloco navy full-bleed com título, área, audiência, período e data
2. **Sumário Executivo** — 3 a 5 achados em cards com borda azul lateral
3. **Contexto e Dados** — Tabela de Data Profiling + callout de qualidade
4. **Resultados** — Um bloco por dimensão: gráfico (Drawing/Polygon/PolyLine) + interpretação
5. **Conclusões** — Cards com número + ação recomendada
6. **Metodologia** — Seção obrigatória (ver Fase 8)

**Regras técnicas para reportlab:**
- Use `SimpleDocTemplate` com **margens zeradas** (ver Regra 1 — PDF zero-margin obrigatório)
- Gráficos como `Drawing` com `Rect`, `PolyLine`, `Polygon` — sem bibliotecas externas
- Coordenadas em Drawing são bottom-up (y=0 na base)
- `PolyLine` e `Polygon` recebem lista plana: `[x1,y1,x2,y2,...]`
- Nunca use `wkhtmltopdf`, `weasyprint` ou Chrome headless
- Informe o caminho completo do PDF ao usuário ao final

---

## Fase 8 — Seção de Metodologia (obrigatória)

Esta seção é **obrigatória** nos dois formatos de output. Tom: simples, claro,
levemente lúdico — qualquer pessoa sem formação técnica deve entender.

---

**Como esta análise foi feita**

*Você não precisa saber o nome técnico de cada método — o que importa é entender
o que fizemos e por que fizemos dessa forma.*

**Passo 1 — Conhecemos os dados antes de qualquer conclusão**
*(Data Profiling — Rahm & Do, 2000, IEEE Data Engineering Bulletin)*

Antes de tirar qualquer conclusão, verificamos se os dados eram confiáveis.
Contamos os registros, mapeamos os campos, encontramos valores ausentes e
identificamos inconsistências. É como conferir todos os ingredientes antes
de cozinhar: se você pular essa etapa, a receita pode dar errado de formas
que você só descobre no final.

**Passo 2 — Conversamos com os dados em ciclos**
*(EDA — Tukey, 1977, Exploratory Data Analysis)*

Exploramos os dados fazendo perguntas. Cada resposta gerava uma nova pergunta.
Não fomos em linha reta — fomos em espiral, indo cada vez mais fundo.
Assim encontramos os padrões que não estavam óbvios na planilha.
O método foi criado pelo estatístico John Tukey em 1977 e é usado até hoje
como padrão mundial de exploração de dados.

**Passo 3 — Analisamos do simples ao complexo**
*(Framework U-B-M — Peat & Barton, 2005, Medical Statistics)*

Primeiro entendemos cada número sozinho (Univariada). Depois vimos como dois
campos se relacionam (Bivariada). Por fim, olhamos para tudo junto (Multivariada).
É como montar um quebra-cabeça: você precisa entender cada peça antes de
tentar ver a imagem completa. Pular etapas leva a conclusões erradas.

**O que esta análise não faz**

Esta é uma Análise *Descritiva* — ela descreve o que aconteceu, com precisão e
embasamento. Para entender *por que* aconteceu, o próximo passo é uma
Análise Diagnóstica. Para prever o que vai acontecer, o passo seguinte é
uma Análise Preditiva.

**Métricas e gráficos utilizados**
[Inserir aqui quais grupos (1–7) foram aplicados, com uma frase simples
explicando o papel de cada um na análise realizada.]

---

## Fase 9 — Encerramento

Após gerar o artifact:
1. Grave `SCRATCH/fase_9_ok.json` com `{"status": "completo", "artifact_tipo": "..."}`.
2. Informe o usuário: "Análise concluída. Artifact disponível acima."
3. Ofereça: "Posso refazer alguma seção, ajustar a visualização ou aprofundar
   algum dos achados?"

---

## Inputs necessários

- **Obrigatório:** respostas do levantamento de requisitos (Fase 1)
- **Obrigatório:** dados em qualquer formato (arquivo, colagem ou descrição)
- **Opcional:** contexto adicional de negócio, metas ou benchmarks do usuário
- **Opcional:** preferências visuais ou de identidade visual

## Output

- **Dashboard**: artifact HTML interativo com abas — link compartilhável
- **Relatório**: PDF salvo localmente gerado com reportlab — zero margem, full-bleed
- Ambos incluem obrigatoriamente a Fase 8 (Metodologia)
- A arquitetura é apresentada e aprovada pelo usuário antes de qualquer análise
- Checkpoints gravados no SCRATCH para retomada em caso de interrupção
