# Skills de Análise Descritiva para Claude Code

> Pipeline completo de análise descritiva — do arquivo bruto ao relatório final.
> Qualquer pessoa, de qualquer área, consegue usar.

---

## O que é isso?

Uma **skill para Claude Code** que guia você por um pipeline profissional de análise
de dados, combinando três metodologias acadêmicas consolidadas:

| Metodologia | Referência | O que faz |
|---|---|---|
| **Data Profiling** | Rahm & Do, 2000 (IEEE) | Diagnóstico de qualidade dos dados |
| **EDA** | Tukey, 1977 | Exploração em ciclos — padrões emergentes |
| **Framework U-B-M** | Peat & Barton, 2005 | Do simples ao complexo: Univariada → Bivariada → Multivariada |

O resultado é um dos dois:
- **Dashboard interativo HTML** — link compartilhável, gráficos com tooltip, tema claro/escuro
- **Relatório analítico PDF** — documento exportável com gráficos, tabelas e metodologia

---

## Quickstart

### 1. Pré-requisitos

- [Claude Code](https://claude.ai/code) instalado e autenticado
- Python 3.x (apenas para output PDF):
  ```bash
  pip install reportlab
  ```

### 2. Instalar a skill

**Opção A — Usar como projeto (recomendado)**

Clone este repositório e abra-o com Claude Code.
As skills ficam disponíveis automaticamente dentro deste projeto.

```bash
git clone https://github.com/leo-philippe/skills_analise_descritiva.git
cd skills_analise_descritiva
claude
```

**Opção B — Instalar globalmente**

Copie a skill para o diretório global do Claude Code.
Após isso ela estará disponível em **qualquer projeto**.

```bash
# macOS / Linux
cp .claude/skills/analise-descritiva-pipeline.md ~/.claude/skills/

# Windows (PowerShell)
Copy-Item ".claude\skills\analise-descritiva-pipeline.md" "$env:USERPROFILE\.claude\skills\"
```

### 3. Usar

Com Claude Code aberto, digite:

```
/analise-descritiva-pipeline
```

O Claude vai:
1. Fazer perguntas sobre seu objetivo e dados (2 rodadas rápidas)
2. Apresentar um plano e pedir aprovação
3. Ler e analisar seus dados
4. Gerar o output no formato que você escolheu

---

## O que o pipeline produz

### Dashboard interativo (HTML)

- KPIs principais com valores calculados
- Gráficos de barras, linhas de área, Pareto — interativos com tooltip
- Tema claro e escuro automático (segue o sistema)
- Link compartilhável via Claude Code artifacts
- Seção de metodologia em linguagem simples

### Relatório PDF

- Hero full-bleed (borda a borda, sem margem branca)
- Sumário executivo com achados em cards
- Gráficos vetoriais (sem dependências externas além de reportlab)
- Seção de metodologia com analogias lúdicas
- Salvo localmente no diretório dos dados

---

## Estrutura do pipeline (9 fases)

```
Fase 0  Inicialização           Detecta SO, define scratchpad, verifica retomada
Fase 1  Levantamento            8 perguntas em 2 rodadas — contexto e dados
Fase 2  Arquitetura             Plano aprovado antes de executar
Fase 3  Escolha do output       Dashboard HTML ou Relatório PDF
Fase 4  Data Profiling          Volume, tipos, nulos, cardinalidade, ranges
Fase 5  EDA                     Ciclo Tukey × 2 por campo — padrões emergentes
Fase 6  Análise U→B→M           Univariada → Bivariada → Multivariada
Fase 7  Geração do output       Dashboard ou PDF com dataviz + artifact-design
Fase 8  Metodologia             Seção obrigatória em linguagem simples
Fase 9  Encerramento            Checkpoint final + oferta de ajustes
```

Para grandes volumes (> 50 mil linhas), as Fases 4–6 rodam em subagente para
não exaurir o contexto — o resultado volta como resumo.

---

## Funcionalidades de resiliência

- **Checkpointing automático**: cada fase salva `fase_{N}_ok.json` no scratchpad
- **Modo retomada**: se a análise foi interrompida, o Claude detecta e pergunta de onde continuar
- **Verbosidade controlada**: outputs brutos vão para arquivo — o chat fica limpo
- **Cross-platform**: Windows, macOS e Linux

---

## Como contribuir

1. Fork este repositório
2. Melhore o skill em `.claude/skills/analise-descritiva-pipeline.md`
3. Abra um PR descrevendo o que mudou e por quê

Sugestões bem-vindas:
- Novas fases (ex: análise de correlação, clustering básico)
- Templates de output por área (RH, Financeiro, Marketing)
- Suporte a novos formatos de entrada (JSON, Parquet, SQL dump)

---

## Licença

MIT — use, adapte e compartilhe livremente.
