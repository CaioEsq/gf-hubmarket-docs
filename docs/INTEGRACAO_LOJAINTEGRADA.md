# Integração LojaIntegrada

## Quando usar

Use `br.com.herasistemas.service.lojaintegrada.LojaIntegrada` quando o cliente precisar operar com a API da LojaIntegrada.

Essa classe usa a URL base `https://api.awsli.com.br/v1` e autentica as chamadas com query params:

```http
chave_api=...
chave_aplicacao=...
```

## Credenciais

A LojaIntegrada usa credenciais estáticas configuradas externamente. A aplicação consumidora deve persistir:

- `lojaIntegradaChaveApi`
- `lojaIntegradaChaveAplicacao`
- `lojaIntegradaApiAddress` opcional, quando quiser sobrescrever a URL base padrão

```java
LojaIntegrada lojaIntegrada = new LojaIntegrada();
lojaIntegrada.setChaveApi(configuracaoCliente.getLojaIntegradaChaveApi());
lojaIntegrada.setChaveAplicacao(configuracaoCliente.getLojaIntegradaChaveAplicacao());

ProdutoService produtoService = new ProdutoService(Arrays.asList(lojaIntegrada));
PedidoService pedidoService = new PedidoService(Arrays.asList(lojaIntegrada));
```

Para validar as credenciais carregadas:

```java
boolean valido = lojaIntegrada.isCredenciaisValidas(
        lojaIntegrada.getChaveApi(),
        lojaIntegrada.getChaveAplicacao()
);
```

`gerarCredenciais` e `atualizarCredenciais` não são suportados, porque o fluxo atual não usa OAuth nem refresh token.

## Produtos na LojaIntegrada

Operações suportadas:

- cadastrar produto em `POST /produto`
- consultar produto em `GET /produto/:id`
- atualizar dados cadastrais em `PUT /produto/:id`
- atualizar estoque em `PUT /produto_estoque/:id`
- atualizar preco em `PUT /produto_preco/:id`
- ativar produto com `ativo=true`
- excluir logicamente produto com `ativo=false`

Nesta primeira entrega o identificador salvo para a LojaIntegrada e simples, usando o `id` do produto retornado pela API.

## Pedidos na LojaIntegrada

Operações suportadas:

- consultar lista de pedidos em `GET /pedido`
- consultar pedido por ID em `GET /pedido/:id`
- filtrar lista por periodo usando `data_criacao__gte` e `data_criacao__lte`
- paginar a lista usando `page` e `limit`
- alterar status em `PUT /pedido/:id`

Mapeamentos de status implementados:

- `ABERTO` -> `aberto`
- `FECHADO` -> `faturado`
- `CANCELADO` -> `cancelado`
- `EM_TRANSITO` -> `enviado`
- `PAUSADO` -> `pausado`

## Nota fiscal na LojaIntegrada

Operações suportadas:

- consultar notas do pedido em `GET /pedido/:id/invoice`
- cadastrar nota fiscal em `POST /pedido/:id/invoice`

Mapeamento usado na gravação:

- `PedidoNotaFiscalInput.chave` -> `access_key`
- `PedidoNotaFiscalInput.numero` -> `invoice_number`
- `PedidoNotaFiscalInput.serie` -> `serie`
- `PedidoNotaFiscalInput.link` -> `url`
- `PedidoNotaFiscalInput.xml` -> `xml`
- `PedidoNotaFiscalInput.dataEmissao` -> `date`
- `PedidoNotaFiscalInput.valor` -> `value`
- `idPedido` -> `sale_number`

## Observacoes

- O suporte de `Cliente` continua indireto, apenas pelo mapeamento de `PedidoOutput`.
- Se a conta do cliente usar um wire format diferente em algum campo de invoice ou status, a implementacao deve ser ajustada conforme a documentacao oficial da LojaIntegrada vigente.
