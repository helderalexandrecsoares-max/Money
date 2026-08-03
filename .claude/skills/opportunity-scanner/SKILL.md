---
name: opportunity-scanner
description: "Scan de mercado para identificar produtos mais vendáveis, nichos de alta margem, e tendências emergentes. Usar quando o utilizador pede análise de oportunidades de negócio, produtos para vender, nichos lucrativos, ou trends de mercado. Invocado com /opportunity-scanner ou pedidos como 'faz um scan de oportunidades', 'que produtos posso vender', 'nichos com margem alta', 'o que está em trend'."
---

## Objetivo

Gerar um relatório de inteligência de mercado com oportunidades reais e acionáveis: produtos para vender, nichos pouco explorados, margens estimadas, e sinais de tendência. O output é um artifact HTML visualmente rico com estilo SITREP.

## Recolha de dados

Usa web search (tool web_search) para pesquisar cada dimensão em paralelo. Para cada dimensão, faz 1-2 pesquisas focadas com termos atuais (incluir o ano atual na query).

### Dimensões a pesquisar

**1. Produtos virais / mais vendáveis agora**
Query sugerida: `"trending products to sell 2026" OR "produtos mais vendidos online 2026" site:reddit.com OR site:ecommercenews.pt OR site:oberlo.com`

Encontrar: 4-5 produtos físicos ou digitais com tracção de vendas comprovada. Para cada um, estimar margem de lucro (%) com base em custo típico vs preço de venda médio.

**2. Nichos de alta margem**
Query sugerida: `"high margin niche products 2026" OR "nichos ecommerce maior margem lucro"`

Encontrar: 4-5 nichos onde margem bruta ≥ 40%. Priorizar nichos underserved em mercados lusófonos (Portugal, Brasil).

**3. Tendências emergentes (trends)**
Query sugerida: `"emerging trends 2026 consumer products" OR "tendências consumo 2026 europa"`

Encontrar: 4-5 tendências macro ou micro que ainda estão a crescer (não no pico). Incluir sinal de timing: se é cedo, ótimo, ou tarde para entrar.

**4. Oportunidades de arbitragem / gaps de mercado**
Query sugerida: `"products popular US not available portugal" OR "negócios que faltam em portugal 2026" OR "gap mercado europeu 2026"`

Encontrar: 3-4 conceitos ou produtos populares noutros mercados mas ausentes ou raros em Portugal/Brasil.

**5. Sinais de investimento e crescimento**
Query sugerida: `"fastest growing ecommerce categories 2026" OR "setores crescimento investimento 2026"`

Encontrar: 3-4 setores com capital a entrar ou crescimento acelerado documentado.

## Análise e scoring

Para cada item encontrado, atribui:
- **score** (1-5): estimativa de oportunidade (5 = muito forte)
- **margin** (string): ex. "40-60%", "~70%", ">80%"
- **timing** (string): `CEDO` | `ÓTIMO` | `TARDE` — fase da tendência
- **tag** (string): categoria em maiúsculas, ex. `VIRAL`, `HIGH MARGIN`, `NICHO`, `ARBITRAGEM`, `TREND`, `CRESCIMENTO`

## Output: Artifact HTML

Cria um único artifact HTML self-contained com o visual SITREP (dark theme, amber accent, IBM Plex Mono). O ficheiro deve ter `<title>Opportunity Scanner — SITREP</title>`.

### Layout

```
[HEADER: OPPORTUNITY SCAN // data e hora]
[Subtítulo: breve descrição do scan]

[GRID 2-col em desktop / 1-col mobile]
  Card: Produtos Mais Vendáveis
  Card: Nichos de Alta Margem
  Card: Trends Emergentes
  Card: Gaps de Mercado
  Card: Sinais de Investimento (largura total)

[FOOTER: disclaimer + timestamp]
```

### Cada card

Cada card tem cabeçalho com título e lista de items. Cada item tem:
- Tag colorida (teal border)
- Título do produto/nicho/trend em bold
- 1-2 frases de descrição com contexto
- Indicador de margem (se aplicável): `▮ MARGEM: 40-60%`
- Indicador de timing: `⏱ TIMING: ÓTIMO`
- Barras de score (5 barras, preenchidas até score)
- Link para fonte (se disponível)

### Paleta de cores

```css
--bg: #0C0A08
--surface: #181209
--surface-2: #211809
--line: #3a2e1a
--text: #F5EFE6
--muted: #8a7d68
--amber: #FFB000
--amber-dim: #8a5f00
--teal: #5EEAD4
--green: #4ade80
--danger: #ff6b5e
```

Usa a fonte `IBM Plex Mono` para labels/monospace e `IBM Plex Sans` para descrições (importar do Google Fonts no `<head>`).

### Estrutura CSS mínima

O HTML deve incluir no `<style>`:
- Reset box-sizing
- Grid responsivo (2 cols → 1 col em mobile)
- Cards com border `--line`, background `--surface`
- Item com `border-left: 2px solid var(--amber-dim)`
- Tags com `border: 1px solid rgba(94,234,212,.3)`, cor teal
- Score bars: `.bar` 4×9px, `.bar.on` background teal
- Animação de scan (dots pulse) para enquanto gera
- Sweep animation no header

## Modo interativo (com chave API)

Se o utilizador tiver a chave API Anthropic disponível (ou perguntar por ela), o artifact pode incluir botões de refresh por secção — cada botão re-corre a pesquisa via API do lado do browser (igual ao SITREP existente). Nesse caso, inclui campo de API key e lógica JavaScript igual à do `index.html` do projeto.

Se não houver chave disponível, gera o relatório completo diretamente no conteúdo estático do artifact.

## Regras de qualidade

- Fontes reais: cada item deve ter pelo menos um nome de fonte credível (mesmo que a URL esteja indisponível)
- Ser específico: nomes de produtos concretos, não categorias vagas
- Dados atuais: priorizar informação de 2025/2026
- Disclaimer no footer: "Este relatório é gerado por IA com pesquisa web em tempo real. Não é aconselhamento financeiro ou de negócio. Confirma sempre fontes antes de investir."
- Timestamp no header e footer
- Responsivo: funciona em mobile

## Variantes de invocação

- `/opportunity-scanner` → scan completo (todas as 5 dimensões)
- `/opportunity-scanner produtos` → só a dimensão de produtos virais
- `/opportunity-scanner nichos` → só nichos de alta margem
- `/opportunity-scanner trends` → só tendências emergentes
- `/opportunity-scanner [categoria]` → filtra pela categoria pedida

Se invocado com um argumento de país/mercado (ex. "para Portugal", "mercado brasileiro"), ajustar as queries para esse mercado.
