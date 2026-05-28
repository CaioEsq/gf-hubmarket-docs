# Produtos, Pedidos e Clientes

## Visao geral

Os serviços de domínio da lib foram pensados para quem quer trabalhar com operações de negócio, e não com chamadas HTTP
diretas.

As duas portas principais são:

- `br.com.herasistemas.domain.produto.ProdutoService`
- `br.com.herasistemas.domain.pedido.PedidoService`

Nos dois casos, a regra principal e a mesma:

1. Buscar no banco as credenciais das integrações do cliente.
2. Instanciar uma ou mais integrações.
3. Passar a lista de integrações no construtor do service.

## ProdutoService

`ProdutoService` centraliza as operações de produto entre as integrações configuradas.

### Instanciação

```java
List<Integracao> integracoes = Arrays.asList(nuvemshop, tray, bagy);
ProdutoService produtoService = new ProdutoService(integracoes);
```

### Metodos disponiveis

- `cadastrarProduto(ProdutoInput input)`
- `cadastrarListaProdutos(List<ProdutoInput> produtos)`
- `consultarProduto(Map<Integracoes, String> idsIntegracao)`
- `atualizarProduto(Map<Integracoes, String> idsIntegracao, ProdutoInput input)`
- `ativarProduto(Map<Integracoes, String> idsIntegracao)`
- `excluirProduto(Map<Integracoes, String> idsIntegracao)`

### Regras importantes

- Em `consultarProduto`, `atualizarProduto`, `ativarProduto` e `excluirProduto`, os IDs precisam ser informados por
  integracao.
- Quando um ID nao for informado para uma integracao, aquela plataforma e ignorada nessa operacao.
- Para NuvemShop, alguns fluxos de atualizacao usam identificadores no formato `productId:variantId`.
- Na NuvemShop, a atualizacao pode envolver `PUT /products/{productId}` para dados do produto e
  `PUT /products/{productId}/variants/{variantId}` para variacao, com `visible=true`.
- Na NuvemShop, a ativacao e logica e usa `PUT /products/{productId}/variants/{variantId}` com `visible=true`.
- Na NuvemShop, a exclusao e logica e usa `PUT /products/{productId}/variants/{variantId}` com `visible=false`.
- Na Tray, a atualizacao usa `PUT /products/{id}` com `available=1`.
- Na Tray, a ativacao e logica e usa `PUT /products/{id}` com `available=1`.
- Na Tray, a exclusao e logica e usa `PUT /products/{id}` com `available=0`.
- Na Tray, o campo `weight` e normalizado para inteiro positivo, com fallback minimo de `1`.
- Na Bagy, o cadastro cria uma variacao padrao e o ID pode usar o formato `productId:variationId`.
- Na Bagy, a atualizacao pode envolver `PUT /products/{productId}`, `PUT /variations/{variationId}` e `PUT /stocks`.
- Na Bagy, a exclusao e logica e usa `PUT /products/{productId}` com `active=false`.
- Na Pedizap, o cadastro usa `POST /api/produto` com `category_id` vindo de `ProdutoInput.pedizapCategoriaId`.
- Na Pedizap, os campos especificos `pedizapContent` e `pedizapResume` sao enviados como `content` e `resume`.
- Na Pedizap, `urlImagens` envia fotos apos o cadastro em `POST /api/produto/{id}/fotos` com `photo_url`.
- Na Pedizap, a exclusao de produto usa `DELETE /api/produto/{id}`.
- Em `consultarProduto`, quando a plataforma responder `404`, a lib trata isso como ausencia de cadastro e nao inclui
  aquela integracao na lista de retorno.
- O retorno sempre vem como lista de `ResultadoIntegracao`, permitindo tratar sucesso e falha por plataforma.

### Exemplo de cadastro

```java
ProdutoInput produto = new ProdutoInput();

produto.setReferencia("SKU-001");
produto.setNome("Produto de Teste");
produto.setValorVenda(new BigDecimal("19.90"));
produto.setQuantidadeEstoque(10);

List<ResultadoIntegracao<ProdutoOutput>> resultados = produtoService.cadastrarProduto(produto);
```

### Exemplo de consulta

```java
Map<Integracoes, String> ids = new EnumMap<>(Integracoes.class);

ids.put(Integracoes.NUVEMSHOP, "334001881:1484964695");
ids.put(Integracoes.TRAY, "12345");
ids.put(Integracoes.BAGY, "163:173");

List<ResultadoIntegracao<ProdutoOutput>> resultados = produtoService.consultarProduto(ids);
```

## PedidoService

`PedidoService` centraliza as operacoes de pedido entre as integracoes configuradas.

### Instanciacao

```java
List<Integracao> integracoes = Arrays.asList(nuvemshop, tray, bagy);
PedidoService pedidoService = new PedidoService(integracoes);
```

### Metodos disponiveis

- `consultarListaPedidos()`
- `consultarListaPedidos(LocalDateTime inicio, LocalDateTime fim)`
- `consultarPedido(Map<Integracoes, String> idsIntegracao)`
- `alterarStatusPedido(Map<Integracoes, String> idsIntegracao, StatusPedido statusPedido)`
- `incluirProdutoPedido(Map<Integracoes, String> idsIntegracao, PedidoProdutoInput pedidoProdutoInput)`
- `excluirProdutoPedido(Map<Integracoes, String> idsIntegracao, String idProduto)`
- `validarNotaFiscalPedido(Map<Integracoes, String> idsIntegracao, PedidoNotaFiscalInput notaFiscalInput)`
- `cadastrarNotaFiscalPedido(Map<Integracoes, String> idsIntegracao, PedidoNotaFiscalInput notaFiscalInput)`

### Regras importantes

- `consultarListaPedidos(inicio, fim)` delega para a implementacao de cada integracao montar os filtros no formato
  esperado pela API externa.
- Nas integracoes que expõem paginação, a biblioteca faz chamadas sucessivas ate esgotar todas as paginas retornadas
  pela API.
- Esse fluxo paginado possui timeout total de protecao para evitar loops ou consultas presas por tempo indefinido.
- Se uma integracao nao encontrar pedidos para o filtro informado, a lib simplesmente nao adiciona resultados daquela
  plataforma.
- `consultarPedido` consulta apenas as integracoes para as quais um ID foi informado.
- `alterarStatusPedido` aplica o mesmo status de dominio para todas as integracoes informadas, e cada implementacao
  converte isso para o status nativo da plataforma.
- `incluirProdutoPedido` e `excluirProdutoPedido` hoje sao efetivos na Tray e na Bagy; quando solicitados para Nuvemshop
  a biblioteca ignora essa integracao sem erro.
- Na Bagy, inclusao e exclusao de produto consultam o pedido, recalculam o array `items` e enviam uma atualizacao
  parcial em `PUT /orders/{id}`. O ID de produto deve seguir `productId:variationId`, pois o trecho depois de `:` vira
  `variation_id`.
- Na Bagy, `FECHADO` usa fulfillment `delivered`, `EM_TRANSITO` usa fulfillment `shipped`, e `CANCELADO` usa
  `/orders/{id}/cancel`.
- Na Bagy, a nota fiscal e gravada no fulfillment com `nfe_number`, `nfe_token` e `nfe_series`.
- Na Pedizap, a listagem paginada usa `GET /api/pedido` com `limit`, `page` e `returnLayout=cliente,itens`.
- Na Pedizap, o filtro de periodo usa `data_inicio` e `data_fim` no formato `yyyy-MM-dd`.
- Na Pedizap, status nativos `1/7`, `2/6`, `3`, `4` e `5` mapeiam para `ABERTO`, `FECHADO`, `CANCELADO`, `PAUSADO` e `EM_TRANSITO`.
- Na Pedizap, a nota fiscal e gravada no pedido por `PUT /api/pedido/{id}` com campos `nfe_*`.
- `validarNotaFiscalPedido` consulta a integracao e devolve `dados.existente=true` quando a nota ja estiver vinculada ao
  pedido.
- Quando a validacao nao encontrar nota, a chamada continua valida e retorna `dados.existente=false` e
  `dados.criada=false`.
- `cadastrarNotaFiscalPedido` sempre valida antes de gravar para evitar duplicidade no pedido.

### Exemplo de consulta por periodo

```java
List<ResultadoIntegracao<PedidoOutput>> resultados =
        pedidoService.consultarListaPedidos(
                LocalDateTime.parse("2025-08-01T00:00:00"),
                LocalDateTime.parse("2025-11-01T00:00:00")
        );
```

### Exemplo de consulta por ID

```java
Map<Integracoes, String> idsPedido = new EnumMap<>(Integracoes.class);

idsPedido.put(Integracoes.NUVEMSHOP, "1800321711");
idsPedido.put(Integracoes.TRAY, "16045");
idsPedido.put(Integracoes.BAGY, "38");

List<ResultadoIntegracao<PedidoOutput>> resultados = pedidoService.consultarPedido(idsPedido);
```

### Exemplo de alteração de status

```java
Map<Integracoes, String> idsPedido = new EnumMap<>(Integracoes.class);
if (isTrayIntegrado()) {
    idsPedido.put(Integracoes.TRAY, "16045");
}

if (isNuvemshopIntegrado()) {
    idsPedido.put(Integracoes.NUVEMSHOP, "1800321711");
}

List<ResultadoIntegracao<PedidoOutput>> resultados =
        pedidoService.alterarStatusPedido(idsPedido, StatusPedido.ABERTO);
```

### Exemplo de validacao de nota fiscal

```java
PedidoNotaFiscalInput notaFiscal = new PedidoNotaFiscalInput();
notaFiscal.setDataEmissao(LocalDate.parse("2026-04-08"));
notaFiscal.setNumero("213123213213");
notaFiscal.setSerie("123");
notaFiscal.setValor(new BigDecimal("50.85"));
notaFiscal.setChave("55555555555555555555555555555");
notaFiscal.setLink("http://nfe.com.br/nsaasb");

List<ResultadoIntegracao<PedidoNotaFiscalOutput>> validacoes =
        pedidoService.validarNotaFiscalPedido(idsPedido, notaFiscal);
```

### Exemplo de cadastro de nota fiscal

```java
PedidoNotaFiscalInput notaFiscal = new PedidoNotaFiscalInput();
notaFiscal.setDataEmissao(LocalDate.parse("2026-04-08"));
notaFiscal.setNumero("213123213213");
notaFiscal.setSerie("123");
notaFiscal.setValor(new BigDecimal("50.85"));
notaFiscal.setChave("55555555555555555555555555555");
notaFiscal.setLink("http://nfe.com.br/nsaasb");
notaFiscal.setXmlDanfe("<xml>danfe</xml>");

List<ResultadoIntegracao<PedidoNotaFiscalOutput>> cadastros =
        pedidoService.cadastrarNotaFiscalPedido(idsPedido, notaFiscal);
```

## Tratamento de retorno

O consumo recomendado e sempre iterar sobre `ResultadoIntegracao`.

```java
for ( ResultadoIntegracao<PedidoOutput> resultado : resultados ) {
    if ( resultado.isSucesso()) {
        tratarSucesso(resultado.getIntegracao(), resultado)
    } else {
        tratarFalha(resultado.getIntegracao(), resultado.getErro());
    }
}
```

Esse formato evita acoplamento ao comportamento de uma unica plataforma e facilita a observabilidade quando a aplicacao
estiver integrada com mais de um marketplace.

## ClienteService

`ClienteService` centraliza operacoes de cliente para integracoes que suportam esse contrato.

### Metodos disponiveis

- `consultarListaClientes()`
- `cadastrarCliente(ClienteInput input)`
- `consultarCliente(Map<Integracoes, String> idsIntegracao)`
- `atualizarCliente(Map<Integracoes, String> idsIntegracao, ClienteInput input)`
- `excluirCliente(Map<Integracoes, String> idsIntegracao)`

Pedizap implementa o CRUD usando `/api/cliente`. Integracoes sem suporte sao ignoradas.

## ProdutoCategoriaService

`ProdutoCategoriaService` centraliza o CRUD de categorias de produto para integracoes que suportam esse contrato.

### Metodos disponiveis

- `consultarListaCategorias()`
- `cadastrarCategoria(ProdutoCategoriaInput input)`
- `consultarCategoria(Map<Integracoes, String> idsIntegracao)`
- `atualizarCategoria(Map<Integracoes, String> idsIntegracao, ProdutoCategoriaInput input)`
- `excluirCategoria(Map<Integracoes, String> idsIntegracao)`

Na Pedizap, o endpoint usado é `/api/categoria`. O campo `resumo` do input não é enviado como campo isolado, pois a API
documentada nao expoe `resume` para categorias; ele serve como fallback de `description` quando `descricao` estiver vazio.
