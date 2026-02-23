# ColectorHub — Estratégia de Monetização e Escalabilidade

> Documento interno de estratégia de negócios. Não expor publicamente.

---

## 1. Princípios

- **ZERO anúncios** — jamais, em nenhuma circunstância
- **Valor real primeiro** — o app precisa ser excelente antes de monetizar
- **Respeito ao usuário** — monetização transparente, sem dark patterns
- **Custo de IA sustentável** — pipeline otimizado para minimizar chamadas pagas

---

## 2. Modelo Freemium — ColectorHub Pro

### Plano Gratuito (Free)

| Recurso | Limite |
|---|---|
| Plataformas | 1 (PS1 no lançamento) |
| Jogos na coleção | 50 |
| Scans por mês | 5 |
| Wishlist | Básica (sem alertas) |
| Busca no catálogo | Ilimitada |
| Adição manual | Ilimitada |

### Plano Pro (Assinatura)

| Recurso | Incluído |
|---|---|
| Plataformas | Todas disponíveis |
| Jogos na coleção | Ilimitado |
| Scans por mês | Ilimitados |
| Wishlist | Com alertas de preço e links de compra |
| Estatísticas avançadas | Valor da coleção, gráficos, tendências |
| Exportar coleção | CSV / PDF |
| Badge de apoiador | Visível no perfil |
| Suporte prioritário | Canal direto |

### Precificação por Região

| Região | Mensal | Anual | Economia |
|---|---|---|---|
| **Brasil** | R$ 4,90 | R$ 39,90 (~R$ 3,33/mês) | 32% |
| **América Latina** | US$ 1,99 | US$ 14,99 | 37% |
| **EUA / Europa** | US$ 4,99 | US$ 39,99 | 33% |
| **Japão** | ¥ 480 | ¥ 3.800 | 34% |

**Estratégia:** empurrar o plano anual com desconto agressivo. Maior retenção e previsibilidade de receita.

---

## 3. Programa de Afiliados

### Plataformas

| Marketplace | Comissão média | Mercado alvo |
|---|---|---|
| **Shopee** | 5-10% | Brasil |
| **Mercado Livre** | 4-12% | Brasil / LatAm |
| **Amazon BR** | 4-8% | Brasil |
| **Amazon US** | 4-8% | EUA / Global |
| **eBay** | Programa de parceiros | EUA / Europa |

### Onde exibir links de afiliados

1. **Wishlist** — botão "Encontrar para comprar" em cada jogo
2. **Detalhes do jogo** — seção "Onde comprar" com preços comparados
3. **Coleção** — sugestão "Complete sua coleção" com jogos relacionados
4. **Alerta de preço** (Pro) — notificação push: "Jogo X da sua wishlist por R$ Y"

### Regras

- Links sempre rotulados como "parceiro" ou "afiliado" (transparência)
- Nunca forçar o usuário a clicar — sempre opcional
- Priorizar lojas com melhor reputação e preço

### Sobre reproduções (repros)

- Permitido linkar repros **apenas se claramente identificados** como reprodução
- Nunca apresentar repro como original
- Tag visual diferenciada: "Reprodução" vs "Original"
- Aplicável principalmente a PS1, PS2, SNES, Mega Drive

### Projeção de receita (afiliados)

| Usuários ativos | Cliques/mês (5%) | Conversões (~10%) | Ticket médio | Comissão (6%) | Receita/mês |
|---|---|---|---|---|---|
| 5.000 | 250 | 25 | R$ 80 | R$ 4,80 | **R$ 120** |
| 20.000 | 1.000 | 100 | R$ 80 | R$ 4,80 | **R$ 480** |
| 50.000 | 2.500 | 250 | R$ 80 | R$ 4,80 | **R$ 1.200** |
| 50.000 (global) | 2.500 | 250 | US$ 30 | US$ 1,80 | **US$ 450** (~R$ 2.700) |

---

## 4. Valor Estimado da Coleção

Feature de alto engajamento e potencial viral.

### Como funciona

- Puxa preços de vendas recentes (Mercado Livre, eBay, PriceCharting)
- Calcula valor estimado individual e total da coleção
- Mostra tendência: valorizando ou desvalorizando

### Monetização

| Plano | Acesso |
|---|---|
| **Free** | Valor total estimado da coleção (número único) |
| **Pro** | Valor por jogo, histórico de preço, gráficos, tendência, alertas |

### Potencial viral

- "Minha coleção de PS1 vale R$ 12.000" → compartilha no Instagram/TikTok
- Gera curiosidade → download orgânico
- Influenciadores de retro gaming vão usar espontaneamente

---

## 5. Parcerias com Lojas

### Modelo

Lojas especializadas em retro gaming pagam mensalidade para aparecer como "Loja Recomendada" dentro do app.

### Benefícios para a loja

- Acesso direto ao público-alvo perfeito (colecionadores ativos)
- Destaque na seção "Onde comprar"
- Analytics: quantos usuários viram/clicaram

### Precificação sugerida

| Plano da loja | Preço/mês | Benefícios |
|---|---|---|
| Básico | R$ 99 | Listagem na seção "Lojas" |
| Destaque | R$ 249 | Listagem + badge "Recomendada" + prioridade nos resultados |
| Premium | R$ 499 | Destaque + push notifications de ofertas + analytics |

---

## 6. Custos de Infraestrutura

### Custo fixo mensal

| Serviço | Plano | Custo |
|---|---|---|
| **Supabase** | Free → Pro (a partir de 50K users) | R$ 0 → US$ 25/mês |
| **Conta Google Play** | Taxa única | US$ 25 (one-time) |
| **Conta Apple Developer** | Anual | US$ 99/ano |
| **Domínio** (site/landing page) | Anual | ~R$ 50/ano |

### Custo variável — IA (Gemini Vision)

| Usuários ativos | Scans/mês | Fallback Gemini (25%) | Custo/chamada | Custo total |
|---|---|---|---|---|
| 1.000 | 5.000 | 1.250 | R$ 0,01 | **R$ 12,50** |
| 10.000 | 50.000 | 12.500 | R$ 0,01 | **R$ 125** |
| 50.000 | 250.000 | 62.500 | R$ 0,01 | **R$ 625** |
| 100.000 | 500.000 | 125.000 | R$ 0,01 | **R$ 1.250** |

### Por que o custo é baixo

1. **Google ML Kit (OCR):** roda no dispositivo do usuário → custo ZERO
2. **Fuzzy matching:** algoritmo local → custo ZERO
3. **Gemini Vision:** só é chamado como fallback (~25% dos scans)
4. **Cache de resultados:** jogo já identificado não chama Gemini novamente
5. **Rate limiting:** Edge Function limita chamadas por usuário/minuto

### Margem por usuário Pro

```
Receita Pro (BR):    R$ 4,90/mês
Custo IA por user:  ~R$ 0,10/mês (média)
Custo Supabase:     ~R$ 0,01/mês (rateado)
────────────────────────────────────
Margem:              R$ 4,79/mês (~98%)
```

---

## 7. Projeção de Crescimento — 24 Meses

### Usuários

| Período | Usuários totais | Novos/mês | Como |
|---|---|---|---|
| Mês 1-3 | 200-500 | ~150 | Comunidades, amigos, grupos Facebook/Discord |
| Mês 3-6 | 500-2.000 | ~400 | Reddit, TikTok, boca a boca |
| Mês 6-12 | 2.000-8.000 | ~800 | Multi-plataforma, YouTube retro |
| Mês 12-18 | 8.000-20.000 | ~1.500 | Versão inglês, mercado global |
| Mês 18-24 | 20.000-40.000 | ~2.500 | Social features, viral orgânico |

### Receita mensal (mês 24)

| Fonte | Conservador | Otimista |
|---|---|---|
| Pro Brasil (5% de ~15K BR) | R$ 3.675 | R$ 7.350 |
| Pro Global (5% de ~10K int.) | R$ 15.000 | R$ 30.000 |
| Afiliados (BR + global) | R$ 2.000 | R$ 5.000 |
| Parcerias com lojas (3-5 lojas) | R$ 1.000 | R$ 3.000 |
| **Total mensal** | **~R$ 21.000** | **~R$ 45.000** |
| **Total anual (projetado)** | **~R$ 250.000** | **~R$ 540.000** |

### Custos no mês 24

| Item | Custo |
|---|---|
| Supabase Pro | R$ 150 |
| Gemini Vision | R$ 500-1.250 |
| Apple Developer | R$ 50 (rateado) |
| **Total custos** | **~R$ 700-1.500/mês** |
| **Margem líquida** | **~96-97%** |

---

## 8. Estratégia por Mercado

### Brasil — Volume + Afiliados

```
Preço baixo (R$ 4,90) → máxima adoção
  → base grande de usuários
    → receita forte de afiliados (Shopee, ML, Amazon BR)
      → colecionador brasileiro compra muito online
```

### Global (EUA, Europa, Japão) — Premium + Assinatura

```
Preço mais alto (US$ 4,99) → margem alta
  → colecionador gringo gasta US$ 50-200 por jogo sem pensar
    → US$ 4,99/mês é irrelevante pra eles
      → afiliados Amazon US/eBay pagam melhor
```

### Chave do crescimento global

- App em **inglês** desde o lançamento (PT-BR como segundo idioma)
- ASO (App Store Optimization) para "game collection tracker", "retro game scanner"
- Presença no Reddit: r/gamecollecting (700K+ membros), r/retrogaming (500K+)

---

## 9. Futuro — Marketplace (v4.0+)

### Conceito

Colecionadores compram e vendem entre si dentro do app. ColectorHub fica com uma porcentagem por transação.

### Modelo

| Item | Valor |
|---|---|
| Taxa por venda | 5-8% do valor |
| Listagem básica | Gratuita |
| Listagem destacada | R$ 2,99 |

### Por que funciona

- O usuário já tem o catálogo dos jogos dentro do app
- "Vender" é um botão na tela do jogo
- Comprador já sabe a condição (mint/good/fair/poor)
- Confiança: perfil com coleção verificada por scan

### Projeção

Com 50K usuários e 2% vendendo 1 jogo/mês:
- 1.000 vendas × R$ 80 ticket médio × 6% taxa = **R$ 4.800/mês**

Isso é o **endgame**: virar o marketplace de referência do retro gaming.

---

## 10. Métricas Chave (KPIs)

| Métrica | Meta mês 6 | Meta mês 12 | Meta mês 24 |
|---|---|---|---|
| Downloads totais | 3.000 | 10.000 | 50.000 |
| Usuários ativos mensais (MAU) | 1.500 | 5.000 | 30.000 |
| Taxa de conversão Pro | 3% | 5% | 5-7% |
| Receita mensal (MRR) | R$ 500 | R$ 5.000 | R$ 20.000+ |
| Custo por aquisição (CPA) | R$ 0 (orgânico) | R$ 0-2 | R$ 2-5 |
| Churn mensal (Pro) | < 10% | < 8% | < 5% |
| NPS (satisfação) | > 40 | > 50 | > 60 |

---

## 11. Resumo

```
Receita = Freemium (Pro) + Afiliados + Parcerias + Marketplace (futuro)
Custos  = Supabase (~R$ 150) + Gemini (~R$ 500) + Apple Dev (~R$ 50)
Margem  = ~96-97%

Sem anúncios. Nunca.
```
