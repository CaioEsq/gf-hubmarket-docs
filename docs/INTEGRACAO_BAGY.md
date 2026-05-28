# Integracao Bagy

## Quando usar

Use `br.com.herasistemas.service.bagy.Bagy` quando o cliente precisar operar com a API Bagy/Dooca.

Essa classe usa a URL base `https://api.dooca.store` e autentica as chamadas com:

```http
Authorization: Bearer token
```

## Credenciais

A Bagy usa token Bearer ja emitido externamente. A aplicacao consumidora deve persistir o token no banco do cliente e
carregar esse valor antes de usar os servicos.

```java
Bagy bagy = new Bagy();
bagy.setToken(configuracaoCliente.getBagyToken());

ProdutoService produtoService = new ProdutoService(Arrays.asList(bagy));
PedidoService pedidoService = new PedidoService(Arrays.asList(bagy));
```

Para validar se o token carregado parece pronto para uso:

```java
boolean valido = bagy.isCredenciaisValidas(bagy.getToken());
```

`gerarCredenciais` e `atualizarCredenciais` nao sao suportados nesta integracao, porque o fluxo atual nao usa OAuth nem
refresh token.

## Produtos na Bagy

Operacoes suportadas:

- cadastrar produto em `POST /products`
- consultar produto em `GET /products/:id`
- atualizar dados do produto em `PUT /products/:id`
- atualizar variacao em `PUT /variations/:id`
- atualizar estoque em `PUT /stocks`
- ativar produto com `active=true`
- excluir logicamente produto com `active=false`

Quando o produto tiver variacao, o identificador retornado e salvo deve usar o formato `productId:variationId`. Esse
formato permite atualizar dados do produto, variacao e estoque nas chamadas futuras.

## Pedidos na Bagy

Operacoes suportadas:

- consultar lista de pedidos em `GET /orders`
- consultar pedido por ID em `GET /orders/:id`
- filtrar lista por periodo usando `created_at=inicio--fim`
- paginar a lista usando `page`
- cancelar pedido com `PUT /orders/:id/cancel`
- marcar pedido em transito com `PUT /orders/:id/fulfillment/shipped`
- marcar pedido fechado com `PUT /orders/:id/fulfillment/delivered`
- incluir produto em pedido existente com `PUT /orders/:id`, atualizando o array `items`
- excluir produto de pedido existente com `PUT /orders/:id`, removendo o item do array `items`

`ABERTO` e `PAUSADO` nao possuem alteracao direta documentada para a Bagy neste momento. Quando solicitados, a
biblioteca retorna erro funcional informando que a alteracao nao e suportada.

Para inclusao e exclusao de itens, a biblioteca consulta o pedido atual, monta novamente apenas o array `items` e envia
a atualizacao parcial documentada em `PUT /orders/:id`. Na inclusao, `PedidoProdutoInput.idProduto` deve usar
`productId:variationId`; o `variationId` vira `items[].variation_id`. Na exclusao, o item e localizado por
`productId:variationId`, por `variation_id` ou pelo ID do item do pedido.

## Nota fiscal na Bagy

A Bagy grava dados de nota fiscal no fulfillment do pedido:

- cria fulfillment com `POST /orders/:id_order/fulfillment`, quando ainda nao existir
- grava a NFe com `PUT /orders/:id_order/fulfillment/invoiced`

Mapeamento usado:

- `PedidoNotaFiscalInput.numero` -> `nfe_number`
- `PedidoNotaFiscalInput.chave` -> `nfe_token`
- `PedidoNotaFiscalInput.serie` -> `nfe_series`

`link` e `xml` continuam disponiveis nos DTOs de dominio, mas nao sao persistidos na Bagy por falta de campo
documentado.

## Exemplo de uso

```java
Bagy bagy = new Bagy();
bagy.setToken(configuracaoCliente.getBagyToken());

Map<Integracoes, String> idsProduto = new EnumMap<>(Integracoes.class);
idsProduto.put(Integracoes.BAGY, "163:173");

ProdutoService produtoService = new ProdutoService(Arrays.asList(bagy));
produtoService.consultarProduto(idsProduto);

Map<Integracoes, String> idsPedido = new EnumMap<>(Integracoes.class);
idsPedido.put(Integracoes.BAGY, "38");

PedidoService pedidoService = new PedidoService(Arrays.asList(bagy));
pedidoService.consultarPedido(idsPedido);
```
