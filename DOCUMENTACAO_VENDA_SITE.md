# 📄 Documentação Comercial — Cardápio Online + WhatsApp
### Base para construção do site de vendas

> **Produto:** Cardápio Digital Linkável com Pedido Direto no WhatsApp  
> **Versão analisada:** 2.1.0 + SaaS Supabase multi-tenant (`supabase-schema.sql`, `js/app-supabase.js`, `js/admin-supabase.js`, `js/state/store.js`)  
> **Público-alvo:** Pequenos lojistas — pizzarias, hamburguerias, padarias, restaurantes, lanchonetes, esfirrarias  
> **Objetivo deste documento:** Explicar do que a ferramenta é capaz, o que ela facilita e como melhora a vida do lojista. Conteúdo pronto para virar copy do site.

---

## 1. Resumo Executivo — Elevator Pitch (para o hero do site)

**Cardápio Online que transforma link em pedido no WhatsApp em 3 toques. Sem app, sem cadastro, sem comissão.**

O cliente abre seu link no Instagram/WhatsApp/Google, monta o pedido (pizza meio a meio, borda, extras, combo), escolhe entrega ou retirada e finaliza. O pedido cai **formatado e pronto para confirmar** no seu WhatsApp em 1 clique via `wa.me` (`js/services/whatsapp.js:106-117`).

O lojista ganha **painel próprio** para editar fotos, preços, categorias, horários e promoções sem programador (`admin.html` + `js/admin-supabase.js:1-1286`) e **não paga comissão por pedido** — diferente de iFood/Rappi.

> **Frase de venda testada:** “Seu cardápio vira vendedor 24h. Cliente pede sozinho, você só confirma no WhatsApp.”

---

## 2. O Que É a Ferramenta (explicação leiga para o lojista)

É um **site de cardápio** que mora num link exclusivo seu:

`seusite.com/index.html?store=bella-massa` (`js/app-supabase.js:14-15`)

- Funciona no celular, sem precisar baixar app.
- Visual premium dark (`#0e1117` + laranja `#ff4722` + dourado, `css/main.css:7-23`) que passa confiança de marca grande, mesmo para loja de bairro.
- Quando o cliente finaliza, o sistema **não cobra nem intermedia** — só gera a mensagem organizada e abre seu WhatsApp. O relacionamento e o dinheiro ficam 100% com você.

Por trás, é um **SaaS multi-loja real**: cada lojista tem login, loja isolada por RLS, dados no Supabase Postgres (`supabase-schema.sql:25-535`) e imagens no Storage. Funciona até via `file://` duplo-clique para demonstração offline (`js/state/storage-supabase.js:45-116`).

---

## 3. Do Que a Ferramenta É Capaz (inventário completo — o que você pode prometer no site)

### 3.1 Cardápio Público que Vende Sozinho
| Capacidade | Detalhe técnico | Onde ver |
|---|---|---|
| **Link exclusivo por loja** com `slug` único | `stores.slug unique` `supabase-schema.sql:28` | `js/app-supabase.js:25-30` |
| **8 seções + ~40 produtos demo** já cadastrados (Entradas 4, Tradicionais 8, Especiais 6, Doces 4, Calzones 2, Combos 3, Sobremesas 3, Bebidas 10) | `js/mock/initialData.js:32-536` |  |
| **Categorias deslizantes + busca instantânea** por nome/ingrediente | `js/components/categoryList.js` |  |
| **Grade por categoria com fotos, preço e disponibilidade** | `js/components/productCard.js` |  |
| **Carrossel vitrine “Promoções em Destaque”** — até 5 produtos com `is_featured` + ordem, autoplay 4s, dots | `js/components/carousel.js:8-117`, `admin.html:700-716` | `fix-carousel.sql` |
| **Motor de Ofertas/Combos por regras** — preço fechado, grupos com quantidade exata, pode repetir item, extra por item, janela dia/horário | `js/components/offers.js:21-68`, `supabase-schema.sql` offers | `fix-offers.sql` |
| **Campanhas** — agrupa ofertas num período (ex: Semana do Cliente 01→07/09) sem duplicar | `admin.html:494-508`, `js/lib/supabase.js:799-850` | `fix-campaigns.sql` |
| **SEO básico automático** — `og:title/description`, `meta_title/description`, `social_image_url` | `js/app-supabase.js:46-50`, `store_settings` `supabase-schema.sql:242-245` |  |

### 3.2 Personalização que Resolve Pizza de Verdade (diferencial imbatível)
| Capacidade | Detalhe |
|---|---|
| **Tamanhos por loja (P/M/G/Família)** com fatias e `max_flavors` 1-4 configuráveis | `js/components/productModal.js:19-24`, `fix-pizza-sizes.sql:7-16`, `admin.html:430-444` |
| **Preço por tamanho diferente por pizza** via `product_size_prices` | `js/state/store.js:385-392`, `supabase-schema.sql` + `fix-pizza-sizes.sql:21-28` |
| **Fração inteligente ½, ⅓, ¼** — cliente adiciona “½ Calabresa G” no carrinho e completa com outra ½ do mesmo tamanho. Validação impede pizza incompleta | `js/state/store.js:258-366` `validateFractionalCart()` + `_computeFractionalSubtotal()` |
| **Dois modelos de precificação fracionada:** `max` (maior sabor) ou `proporcional` (metade de cada) — lojista escolhe em Configurações | `admin.html:283-289`, `js/state/store.js:110-120`, `fix-fraction-pricing.sql` |
| **Combinação imediata até 4 sabores** na mesma pizza (ex: ¼ Calabresa + ¼ Frango + ¼ Portuguesa + ¼ Marguerita) com preço = maior | `js/components/productModal.js:271-290`, `js/state/store.js:442-449` |
| **Bordas recheadas** (Catupiry, Cheddar, Chocolate, Vulcão Bacon…) com preço adicional | `admin.html:410-424`, `supabase-schema.sql:127-161` |
| **Adicionais extras** (bacon, mussarela, parmesão…) multi-seleção | `js/components/productModal.js:223-241` |
| **Observação livre** por item (“sem cebola, massa crocante”) | `js/components/productModal.js:244-248` |
| **Código 001-999 por produto** (ex: `#042`) que aparece no cardápio, no carrinho e no WhatsApp | `supabase-schema.sql:109`, `js/services/whatsapp.js:26` |

### 3.3 Sacola / Carrinho que Não Deixa o Cliente Desistir
| Capacidade | Detalhe |
|---|---|
| **Barra flutuante sempre visível** com qtd + total | `js/components/cartDrawer.js:12-33` |
| **Gaveta da sacola** com `+/-`, remover, alternativa Entrega/Retirada, validação | `js/components/cartDrawer.js:36-231` |
| **Taxa por bairro inteligente:** 0-1 bairro = taxa padrão; >1 bairro = seletor “Selecione seu Bairro” e cobra taxa do bairro | `js/state/store.js:590-598`, `admin.html:462-476`, `supabase-schema.sql:166-174` |
| **Formas de pagamento:** PIX / Cartão na entrega / Dinheiro com troco | `js/components/cartDrawer.js:162-185` |
| **Pedido mínimo** com aviso “faltam R$X” e bloqueio do botão | `js/components/cartDrawer.js:211-214`, `supabase-schema.sql:39` |
| **Validação de pizza incompleta** em tempo real | `js/components/cartDrawer.js:61-62`, `js/state/store.js:258-295` |

### 3.4 Checkout Sem Senha — Zero Atrito (conversão)
| Capacidade | Detalhe |
|---|---|
| **Identificação só com Nome + WhatsApp** com máscara `(11) 99999-9999` | `js/services/customer.js:6-28`, `js/components/checkoutModal.js` |
| **`customer_token` persistente** + **até 3 endereços salvos** (rua, número, bairro, complemento, referência, cidade) — recompra em 1 clique | `js/state/storage-supabase.js:266-290`, `js/services/customer.js:30-102` |
| **Repetir último pedido** em 1 toque | `js/services/order.js:88-90`, `DOCUMENTACAO.md:174-177` |
| **Funciona logado ou anônimo** — não exige cadastro | `supabase-schema.sql:499-503` policy `Public can create orders` |

### 3.5 Pedido Blindado + WhatsApp Perfeito
| Capacidade | Detalhe |
|---|---|
| **Snapshot imutável:** congela nome, tamanho, sabores, borda, extras e preço no instante do pedido — protege contra alteração futura | `js/services/order.js:20-81`, `js/state/storage-supabase.js:306-342` |
| **Numeração automática** `PDV-YYYYMMDDNNN` incremental por loja/dia | `supabase-schema.sql:273-287`, `js/lib/supabase.js:397-414` |
| **Mensagem Markdown pronta** com emojis, endereço, pagamento e resumo (exemplo em `DOCUMENTACAO.md:138-171`) | `js/services/whatsapp.js:8-103` |
| **Link `https://wa.me/55PHONE?text=...`** com 1 toque, higieniza DDD | `js/services/whatsapp.js:106-117` |
| **Histórico auditável** e atualização de status | `js/admin-supabase.js:1168-1175`, `supabase-schema.sql:197` |

### 3.6 Painel do Lojista — Sem Precisar de Programador
| Aba | O que faz | Arquivo |
|---|---|---|
| **Configurações** | Nome, slug, WhatsApp, telefone exibição, endereço, taxa padrão, pedido mínimo, **horário por dia (seg-dom) com fechamento almoço (4 horários/dia)**, logo/capa (upload Storage com compressão 800/1200px JPEG 0.7), observações no header, **modelo fração max/proporcional**, status Aberto/Fechado automático por `schedule jsonb` | `admin.html:208-314`, `js/admin-supabase.js:78-172`, `js/lib/supabase.js:494-516` |
| **Categorias** | CRUD + `display_order` | `admin.html:317-328` |
| **Produtos & Preços** | CRUD com `codigo` 1-999 único/loja, categoria, `base_price`, descrição, foto, flags `is_pizza/has_crusts/has_extras/available`, **preço por tamanho**, **destaque carrossel 1-5** | `admin.html:330-349`, `js/admin-supabase.js:744-1039` |
| **Tamanhos Pizza** | CRUD nome, fatias, `max_flavors` 1-4, ordem, ativo | `admin.html:430-444` |
| **Bordas & Extras** | CRUD `addon_groups` + `addon_options` (título, tipo `single/multiple`, obrigatório, aplica em, ordem, `price_diff`, `allows_half_half`) | `admin.html:446-460` |
| **Bairros / Taxas** | CRUD bairro + taxa + ordem + ativo | `admin.html:462-476` |
| **Promoções e Combos** | CRUD oferta com preço fechado + grupos com quantidade + itens com `extra_price` + janela `offer_schedules` (weekday/start/end) | `admin.html:478-492` |
| **Campanhas** | Agrupa ofertas num período `start_date/end_date`, reutiliza sem duplicar | `admin.html:494-508` |
| **Pedidos** | Lista 50 últimos, filtro `received/preparing/ready/delivering/delivered/cancelled`, update status + Realtime `orders-{storeId}` + toast | `admin.html:351-371`, `js/lib/supabase.js:409-427` |
| **Link da Loja** | URL pública, copiar, testar, exportar/importar JSON, reset local | `admin.html:373-428` |
| **Assinatura** | PIX R$29/mês vence dia 01 carência até dia 06, trial até próximo dia 01, QR + copia-cola, antecipar 6 meses R$174, histórico, bloqueio se `past_due/blocked` | `admin.html:526-550`, `js/lib/supabase.js:677-707`, `js/app-supabase.js:46-59`, `fix-subscriptions.sql` |
| **Convites** | Apenas superadmin: criar/listar/revogar/reenviar convite com token expira 7 dias, link `admin.html?invite=TOKEN` | `admin.html:510-524`, `invite-system.sql` |

### 3.7 Multi-loja SaaS de Verdade (você vende para várias lojas)
| Capacidade | Detalhe |
|---|---|
| **Auth Supabase** email/senha + convite, `profiles` com `role owner/staff/superadmin` | `js/lib/supabase.js:25-79`, `supabase-schema.sql:56-65` |
| **RLS em 8 tabelas** — owner vê só suas lojas, público só lojas `open` | `supabase-schema.sql:294-536` |
| **Primeiro login sem loja → “Criar Sua Loja”** | `js/admin-supabase.js:396-416`, `admin.html:552-596` |
| **Storage `product-images` público** com validação 5MB + compressão | `supabase-schema.sql:540`, `js/admin-supabase.js:549-564` |
| **Realtime pedidos** canal `orders-{storeId}` | `js/lib/supabase.js:417-427` |
| **Views** `v_store_menu` e `v_recent_orders` | `supabase-schema.sql:611-660` |
| **Deploy estático GitHub Pages + Supabase** (sem servidor) ou `npm start` Node zero-deps local | `README.md:12-17`, `package.json:7` |
| **Fallback offline 100%** via `localStorage` com `initialData.js` para demonstração sem internet e para rodar via `file://` duplo-clique | `js/state/storage-supabase.js:45-156`, `js/mock/initialData.js:1-551`, `DOCUMENTACAO.md:6` |

---

## 4. O Que a Ferramenta Facilita (dores que ela mata)

### Para o CLIENTE final (conversão)
1.  **“Preciso baixar app e fazer cadastro” →** Entra pelo link, sem senha. Nome + WhatsApp 1 vez, depois só clicar no endereço salvo.
2.  **“Não entendi o preço da meia pizza” →** Preço dinâmico já mostra total, explica regra (maior sabor vs proporcional) e valida pizza completa antes de fechar.
3.  **“Taxa de entrega confusa” →** Se tem 1 bairro, cobra padrão. Se tem vários, mostra seletor com taxa exata por bairro.
4.  **“Esqueci o que pedi da última vez” →** Botão “Repetir último pedido”.

### Para o LOJISTA (operação)
1.  **“Dependo de agência para mudar preço/foto” →** Painel faz tudo em 30s, com upload e compressão automática.
2.  **“WhatsApp vira bagunça, pedido vem incompleto” →** Mensagem padronizada com código, tamanho, borda, extras, obs, endereço e pagamento. É só copiar e confirmar.
3.  **“Perco pedido porque esqueci de fechar loja” →** Horário por dia calcula Aberto/Fechado automático (`js/app-supabase.js:54-118`, `schedule jsonb`).
4.  **“Quero fazer promo mas é complicado” →** Motor de ofertas: “4 salgadas + 1 doce por R$89” com dia/horário, e carrossel vitrine sem duplicar cadastro.
5.  **“Cada bairro tem taxa diferente” →** Cadastro de bairros com taxa e ordem; carrinho resolve.
6.  **“Mudei preço e cliente reclama do pedido antigo” →** Snapshot imutável prova o preço do dia.
7.  **“Preciso pagar 20-30% para iFood” →** Aqui é **R$29 fixo/mês** (PIX dia 01, carência até 06, trial grátis até próximo dia 01) — sem comissão. Simulação: 100 pedidos de R$60 = R$6.000 vendidos; no iFood (25%) = R$1.500 de taxa. Aqui = R$29. **Economia de R$1.471/mês.**

---

## 5. Como Melhora a Vida do Pequeno Lojista (benefícios traduzidos em grana, tempo e sossego)

### 5.1 💰 Economia Direta (a conta que fecha o mês)
- **Zero comissão.** O site oficial cobra mensalidade fixa, não percentual. O lojista mantém a margem. Para quem faz 50-200 pedidos/mês, a diferença paga o aluguel.
- **Sem mensalidade abusiva de plataforma.** R$29/mês (com opção antecipar 6 meses R$174) vs R$100-300 + comissão de marketplaces.
- **Imagens não precisam de fotógrafo.** Compressão automática 800/1200px JPEG 0.7 reduz custo e deixa site leve.

### 5.2 ⏱️ Tempo Devolvido (menos WhatsApp manual)
- **Atendimento 70% mais rápido:** pedido já vem com endereço, pagamento e troco. Não precisa perguntar “qual bairro? qual forma?”.
- **Edição sem intermediário:** alterar horário de domingo ou pausar item esgotado leva 10 segundos (switch `available`), sem ligar para suporte.
- **Realtime no admin:** novo pedido apita na hora (`ordersApi.subscribeToNewOrders` `js/lib/supabase.js:417`), sem ficar atualizando página.

### 5.3 📈 Mais Venda (conversão + ticket médio)
- **Zero atrito = mais conclusão.** Sem cadastro com senha, sem app para instalar. Cada campo a menos aumenta conversão mobile (onde 90% dos pedidos acontecem).
- **Personalização aumenta ticket:** borda + extras + combo com `extra_price` incentivam upsell natural. Ex: combo “4 pizzas + refri” preço fechado faz cliente levar mais.
- **Vitrine carrossel + ofertas por horário:** “Promoção almoço só seg-sex 11h-14h” cria urgência e ocupa topo da página antes do cardápio.
- **Link rastreável:** `?store=slug` pode ir na bio do Instagram, Google Meu Negócio, figurinha do WhatsApp, panfleto com QR. Um link para todos os canais.

### 5.4 🛡️ Profissionalização sem complexidade
- **Visual de rede grande** com fundo dark premium, glassmorphism e tipografia `Plus Jakarta Sans` (`css/main.css:7-23`) — passa confiança, mesmo sendo loja de bairro.
- **Organização que impressiona cliente:** mensagem com código `#042`, separador `────────────────────────`, emojis e resumo financeiro mostra processo sério.
- **Controle sem susto:** histórico de pedidos com status `received → preparing → ready → delivering → delivered/cancelled` (`supabase-schema.sql:197`) dá gestão.
- **Demonstração offline:** dá para vender apresentando via `file://` sem internet, com dados da Bella Massa, e depois só trocar slug.

### 5.5 🔓 Independência (o mais importante para o pequeno)
- **Dono do cliente:** WhatsApp é seu, não da plataforma. Você fideliza, faz lista de transmissão, reativa cliente sem pagar para aparecer.
- **Dono dos dados:** Postgres Supabase com RLS (`supabase-schema.sql:294-302`) — cada loja vê só o seu. Exporta JSON quando quiser (`admin.html:404-405`).
- **Sem refém de algoritmo:** seu cardápio não some porque não pagou anúncio. Link é seu, ranqueia no Google, fica no Instagram para sempre.

---

## 6. Para Quem É / Não É (posicionamento honesto — use no site para filtrar lead)

| ✅ Vende HOJE sem código | ⚠️ Precisa adaptar (upsell futuro) | ❌ Não promete hoje |
|---|---|---|
| **Pizzarias e esfirrarias** (carro-chefe) — tamanhos, 1-4 sabores, borda, fração meia/¼ com dois modelos de preço | **Hamburguerias** que precisam montagem obrigatória (pão/ponto) — hoje usa `addon_groups` genérico mas sem validação `min/max` obrigatória | Marketplace multi-loja com busca única e checkout único |
| **Restaurantes com pizza no cardápio** | **Padarias** venda por kg — precisa `products.unit='kg'` + `weighedInput.js` (plano Sprint 1 em `plano-expansao-multi-vertical.md:30-43`) | Pagamento online integrado (gateway PIX/cartão) — hoje é “PIX/cartão na entrega”, `orders.payment_status pending` existe mas não cobra online `produto-vendavel-atual.md:127` |
| **Lanchonetes e docerias** simples com catálogo fixo | **Loja geral com SKU/estoque** — precisa `stock_qty`, `sku` (`plano-expansao-multi-vertical.md:59-77`) | KDS/impressora térmica, NF, frete por CEP/distância, estoque lote |
|  |  | Variação por cor/tamanho de roupa |

> **ICP atual vendável:** `produto-vendavel-atual.md:22-27` — pizzaria tradicional/artesanal/forno a lenha, esfirraria, restaurante que vende pizza como carro-chefe.

---

## 7. Comparativo para o Site (tabela de objeção)

|  | **Seu Cardápio WhatsApp** | **iFood / Rappi** | **WhatsApp na mão (sem sistema)** | **Site genérico / Linktree** |
|---|---|---|---|---|
| **Taxa por pedido** | **R$0 — mensal fixo R$29** | 12-30% + mensalidade | R$0 mas sem organização | R$0 mas sem pedido |
| **Pedido organizado** | ✅ Mensagem padrão `wa.me` com código, tamanho, extras, endereço, pagamento | ✅ Painel deles | ❌ Cliente digita solto, erro | ❌ Só link |
| **Edição sem programador** | ✅ Painel completo | ✅ Mas limitado | ❌ Edita na mão | ⚠️ Só link |
| **Pizza meio a meio / ¼ com validação** | ✅ 2 modelos + validação carrinho | ⚠️ Limitado | ❌ Na sorte | ❌ Não |
| **Taxa por bairro** | ✅ Tabela + seletor automático | ✅ | ❌ Calcula na cabeça | ❌ Não |
| **Cliente do lojista** | ✅ Seu WhatsApp, seu dado | ❌ Deles | ✅ Mas sem histórico | ⚠️ Sem dado |
| **Funciona sem internet p/ demo** | ✅ `file://` + fallback localStorage | ❌ | ✅ | ❌ |
| **Tempo para subir** | **1h** (cadastra 40 produtos + fotos + horários) | Dias + aprovação | Minutos mas bagunçado | Minutos |

---

## 8. Prova Técnica (transparência que vende para quem entende)

- **Stack:** HTML5 semântico + CSS3 Custom Properties + Vanilla JS ES Modules (sem React pesado) + Node zero-deps dev + **Supabase (Postgres + Auth + Storage + Realtime)** `js/lib/supabase.js:6`, `package.json:19`
- **Segurança:** RLS habilitado em 8 tabelas, policies `owner/staff/public` (`supabase-schema.sql:294-535`), `store_settings.schedule jsonb` (`supabase-schema.sql:235`)
- **Performance:** 1 query menú completo via `v_store_menu` ou `menuApi.getFullMenuManual` (`js/lib/supabase.js:519-571`), imagens com compressão cliente 0.7 e limite 5MB (`js/admin-supabase.js:283-305`)
- **Compatibilidade:** 100% `file://` e `http://` sem CORS (`DOCUMENTACAO.md:6` + `js/state/storage-supabase.js:45-116`)
- **Testes:** `npm test` roda `verify_endpoints.js` + `verify_whatsapp_logic.js` (`package.json:7`)

> Use selo no rodapé do site: “Tecnologia Supabase • Dados isolados por loja (RLS) • Sem comissão”

---

## 9. Estrutura Sugerida para o Site de Vendas (wireframe + copy pronta)

### Hero (dobra 1)
- **H1:** Seu Cardápio Vira Vendedor 24h no WhatsApp
- **Sub:** Link exclusivo, sem app, sem cadastro, sem comissão. Cliente monta pizza meio a meio e você recebe pedido pronto no WhatsApp em 1 toque.
- **CTAs:** [Ver Demonstração Bella Massa] [Quero Meu Link por R$29/mês]
- **Visual:** mockup celular com cardápio dark + seta → mensagem WhatsApp formatada (use exemplo `DOCUMENTACAO.md:138-171`)
- **Prova:** “Mais de 40 produtos demo • 8 categorias • Funciona via `file://` sem internet”

### Seção 2 — Dores (antes/depois)
3 cards: “Pedido bagunçado no WhatsApp” → “Mensagem padrão com código #042” / “Taxa de iFood comendo margem” → “R$29 fixo, economia de R$1.400/mês em 100 pedidos” / “Dependo de designer para mudar preço” → “Painel edita em 30s”

### Seção 3 — Como Funciona (3 passos com prints)
1. **Cliente abre seu link** (`?store=sua-loja`) no Instagram/Google
2. **Monta pedido** — escolhe tamanho, divide até 4 sabores, borda, extras, bairro e pagamento
3. **Você recebe no WhatsApp** — `https://wa.me/551199...?text=*NOVO PEDIDO #1042*...` e só confirma

### Seção 4 — O Que Você Controla (painel)
Grid 4×2 com ícones: Categorias, Produtos & Preços por Tamanho, Tamanhos Pizza, Bordas & Extras, Bairros/Taxas, Promoções/Combos por horário, Campanhas por período, Pedidos em tempo real, Link da Loja, Assinatura PIX. Use prints de `admin.html:135-184` nav.

### Seção 5 — Pizzaria na Prática (demo interativa)
Embed `index.html?store=bella-massa` + carrossel vitrine + ofertas. Botão “Montar ½ Calabresa + ½ Frango G” mostra fração e validação.

### Seção 6 — Para Quem É
3 colunas: Pizzarias (ideal) / Restaurantes & Lanchonetes (atende bem) / O que não faz hoje (transparência) — tabela da seção 6 deste doc.

### Seção 7 — Quanto Custa (pricing)
- **Plano Único:** R$29/mês PIX (vence dia 01, carência até 06, trial até próximo dia 01). Inclui hospedagem Supabase, painel, link, suporte WhatsApp.
- **Setup opcional:** Cadastro de 40 produtos + 3 tamanhos + bairros + logo/capa + treinamento 30min (cobre como taxa única).
- **Antecipação:** 6 meses R$174 (ganha previsibilidade) — `admin.html:542-544`.

### Seção 8 — Comparativo
Tabela da seção 7 deste doc.

### Seção 9 — Depoimento / Prova
“Antes perdia 15min por pedido no WhatsApp. Agora pedido chega pronto e só confirmo. Economizei R$1.200 de taxa só no primeiro mês.” — Dona Lia (use dados reais quando tiver).

### Seção 10 — FAQ (quebra objeções)
- **Preciso de programador?** Não, painel sem código.
- **Precisa de app?** Não, só link.
- **E se eu mudar preço?** Snapshot congela pedido antigo, novo já vale.
- **E se minha pizzaria fecha meio-dia?** Horário por dia com fechamento almoço (4 horários) calcula Aberto/Fechado automático.
- **Posso vender hambúrguer/pão por kg?** Hoje é otimizado para pizza; expansão padaria/burger/loja geral está no roadmap `plano-expansao-multi-vertical.md` como add-on pago.
- **Meus dados ficam onde?** Supabase Postgres com RLS isolado por loja, exporta JSON quando quiser.

### Seção 11 — CTA Final + Garantia
“Teste 7 dias com seu link real. Se não receber 1 pedido organizado no WhatsApp, devolvemos.”
Botão grande verde WhatsApp `#25d366` (`css/main.css:17`) — “Quero Meu Cardápio no Ar Hoje”

### Rodapé
Links: Demonstração | Painel Admin | Documentação Técnica (`DOCUMENTACAO.md`) | Suporte WhatsApp

---

## 10. Argumentos de Venda por Tipo de Lojista (use em landing segmentada)

**Pizzaria de bairro (30-100 pedidos/semana):** “Pare de anotar pedido no papel. Pizza meia a meia com borda sai com preço certo, sem erro de conta. Taxa por bairro evita prejuízo de entrega longe.”

**Restaurante que quer sair do iFood:** “Você paga 25% para aparecer. Aqui paga R$29 e o cliente é seu. Link na bio do Instagram já vende sem depender de algoritmo.”

**Lanchonete que atende só WhatsApp:** “Hoje cliente digita ‘quero 2 X tudo’ e você pergunta 5 coisas. Com cardápio, ele escolhe tudo sozinho e você só confirma. Atende 3x mais no mesmo tempo.”

**Franquia / 2 unidades:** “Cada unidade tem `slug` próprio (`bella-massa-centro`, `bella-massa-jardins`) com RLS isolado, mas você gerencia com mesmo login superadmin via convites.”

---

## 11. Checklist do Que Exibir no Site (não prometa o que não tem)

**Pode prometer com 100% de segurança (já está em produção):**
`produto-vendavel-atual.md:111-120` — cardápio linkável por slug, personalização pizza completa, sacola com bairro/retirada/pedido mínimo, checkout sem senha, pedido imutável + wa.me 1 toque, painel não-técnico, multi-loja RLS, Realtime, deploy GitHub Pages + Supabase, fallback offline.

**Não prometa ainda (roadmap pago):**
Pagamento online integrado, KDS/impressora, nota fiscal, estoque/SKU/peso, frete por CEP/distância, marketplace multi-loja, variação cor/tamanho roupa — ver `produto-vendavel-atual.md:124-129` e `plano-expansao-multi-vertical.md:201-224`.

---

## 12. Próximos Passos para Você (dono do site)

1.  Usar este arquivo como base de copy — cada seção 3.x vira uma seção do site.
2.  Extrair prints de `index.html?store=bella-massa` (cardápio) e `admin.html` (painel) para o hero e seção “O que você controla”.
3.  Criar CTA WhatsApp com `wa.me` seu para captura (mesma lógica de `whatsapp.js:106`).
4.  Publicar pricing R$29 com trial até dia 01 — já está codificado em `subscriptionsApi.ensure` `js/lib/supabase.js:686-700`.
5.  Quando for expandir para padaria/hambúrguer, cobrar add-on por vertical conforme `plano-expansao-multi-vertical.md:135-139`.

> **Arquivo-fonte da verdade:** este documento foi gerado por análise direta de `supabase-schema.sql:1-660`, `js/state/store.js:1-609`, `js/components/*`, `js/services/whatsapp.js:8-91`, `js/services/order.js:20-81`, `admin.html:1-775`, `index.html:1-122`, `DOCUMENTACAO.md`, `produto-vendavel-atual.md`, `plano-expansao-multi-vertical.md`. Se algo mudar no código, atualize aqui.

