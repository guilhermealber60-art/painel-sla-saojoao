# Painel de Entregas — Farmácias São João

Painel de nível de serviço (SLA) do e-commerce, para acompanhamento diário e cobranças direcionadas. É um site estático de um único arquivo (`index.html`) que faz todos os cálculos no navegador — nenhum dado é enviado para servidores.

**Fontes de dados:** VTEX (Report) · Abbiamo (Orders) · Abbiamo (Events).

---

## Como funciona o histórico e o acesso compartilhado

O painel carrega automaticamente os arquivos versionados em `data/`:

- `data/snapshot.json` — todos os pedidos já acumulados (o que todos veem ao abrir o link).
- `data/lojas.json` — a base fixa de Coordenadores, Distritais, Diretores e Regionais por filial.

**Cada "Atualizar dados" soma os pedidos do dia aos que já estavam publicados** (deduplicados pelo número do pedido — se o mesmo pedido aparecer de novo, a versão mais recente vence). Ou seja, você pode enviar só os CSVs do dia, sem precisar reexportar o período inteiro toda vez; o painel acumula sozinho.

Cada atualização é um **commit** no Git, então você tem histórico completo (pode ver e voltar a qualquer versão) e qualquer pessoa com o link enxerga sempre a última publicada.

Se precisar começar um período do zero (ex.: virada de mês), use o botão **Apagar dados acumulados** dentro de "Atualizar dados" — ele some com todos os pedidos carregados na tela (não dá pra desfazer localmente; só recuperando um `snapshot.json` antigo pelo histórico do Git) e depois é só publicar de novo.

---

## Publicar no GitHub Pages (uma vez)

1. Crie um repositório novo no GitHub (ex.: `painel-sla-saojoao`).
2. Envie o conteúdo desta pasta para o repositório. Pela web: **Add file → Upload files**, arraste tudo (incluindo a pasta `data/`) e confirme o commit. Pelo terminal:
   ```bash
   git init
   git add .
   git commit -m "Painel SLA — versão inicial"
   git branch -M main
   git remote add origin https://github.com/SEU_USUARIO/painel-sla-saojoao.git
   git push -u origin main
   ```
3. No repositório: **Settings → Pages**. Em **Build and deployment**, escolha **Deploy from a branch**, selecione a branch `main` e a pasta `/ (root)`. Salve.
4. Em ~1 minuto o site fica disponível em `https://SEU_USUARIO.github.io/painel-sla-saojoao/`. Compartilhe esse link.

> **Privacidade:** um repositório público deixa o site (e o `snapshot.json`) acessível a qualquer pessoa com o link. O snapshot contém apenas indicadores operacionais (pedido, filial, modalidade, plataforma, entregador e horários das etapas) — **nenhum dado de cliente** (nome, endereço, contato). Se precisar de acesso restrito, mantenha o repositório privado e distribua o `index.html` por outro meio, ou use GitHub Pages em uma organização com controle de acesso.

---

## Atualizar os dados do dia (rotina)

1. Abra o site publicado (ou o `index.html` local).
2. Clique em **Atualizar dados** e envie os 3 CSVs do dia (VTEX Report, Abbiamo ORDERS, Abbiamo EVENTS). A base de gestores (`Lojas`) já vem carregada; só reenvie se a estrutura mudar.
3. Clique em **Publicar (snapshot)**. O navegador baixa `snapshot.json` (e `lojas.json`, se você tiver editado a base).
4. Substitua esses arquivos na pasta `data/` do repositório e faça commit. Pela web: **Add file → Upload files** dentro da pasta `data/`, envie os arquivos e confirme. Pelo terminal:
   ```bash
   git add data/snapshot.json data/lojas.json
   git commit -m "Dados de AAAA-MM-DD"
   git push
   ```
5. O site atualiza sozinho para todos em alguns minutos.

---

## Editar a base de gestores

Use o botão **Base de gestores** no painel para corrigir Coordenador/Distrital/Diretor/Regional de uma filial (vinculado pelo **código** da loja). Depois clique em **Publicar (snapshot)** e faça commit do `lojas.json` para valer para todos.

---

## Exportações

- **Exportar pedidos em atraso** — planilha `.xlsx` com todos os pedidos em atraso do filtro aplicado no momento (pedido, filial, gestores, plataforma, entregador, modalidade, horários e tempos de separação/coleta/entrega).
- **Imagem (slides)** — cada seção do painel como PNG, no tamanho de slide.

## Estrutura

```
index.html          Painel (todo o código: layout + cálculos)
data/snapshot.json  Dados processados (versionados)
data/lojas.json     Base fixa de gestores por filial
.nojekyll           Garante que o GitHub Pages sirva a pasta data/
```

## Premissas dos cálculos

- SLA = No prazo ÷ Sucessos. Atraso quando prazo prometido < entrega.
- Separação = Criado Abbiamo − Criado VTEX; meta 15 min (No Prazo até 15 min).
- Coleta: `IN_TRANSIT` para Uber/99, `DISPATCHED` para as demais; entrega = `SUCCESSFUL`.
- Pedidos considerados: `SUCCESSFUL` com origem VTEX / API-V2 / API Abbiamo.
- Fluxo e "tempo de separação" da Entrega Rápida excluem a "Entrega Rápida São João".
- Finalizações suspeitas são **indícios** (rajada, quase simultâneas, ocioso→rajada), não conclusões.
