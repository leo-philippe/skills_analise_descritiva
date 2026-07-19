# Skills de Análise Descritiva

Este repositório contém skills para Claude Code que implementam um pipeline
completo de análise descritiva de dados — do arquivo bruto ao output final.

## Skills disponíveis

| Skill | Como invocar | O que faz |
|---|---|---|
| `analise-descritiva-pipeline` | `/analise-descritiva-pipeline` | Pipeline completo: requisitos → profiling → EDA → dashboard ou PDF |

## Skills bundled necessárias (já no Claude Code)

O pipeline usa internamente:
- `/dataviz` — princípios de visualização de dados (escolha de forma, marcas, paleta)
- `/artifact-design` — calibragem visual do output HTML

Essas skills fazem parte do Claude Code e não exigem instalação adicional.

## Pré-requisitos

- **Claude Code** instalado e autenticado
- **Python 3.x** com `reportlab` (necessário apenas para output PDF):
  ```
  pip install reportlab
  ```

## Como usar

1. Clone este repositório e abra com Claude Code
2. Digite `/analise-descritiva-pipeline` no chat
3. Siga o levantamento de requisitos (o Claude vai guiar você)
4. Forneça o arquivo de dados quando solicitado
5. Escolha o output: dashboard interativo ou relatório PDF

## Estrutura do projeto

```
.claude/
  skills/
    analise-descritiva-pipeline.md   ← skill principal
README.md                            ← quickstart completo
CLAUDE.md                            ← este arquivo (lido automaticamente)
```
