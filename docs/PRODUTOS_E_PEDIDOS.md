# Produtos e Pedidos

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
List<Integracao> integracoes = Arrays.asList(nuvemshop, tray);
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

- Em `consultarProduto`, `atualizarProduto`, `ativarProduto` e `excluirProduto`, os IDs precisam ser informados por integracao.
- Quando um ID nao for informado para uma integracao, aquela plataforma e ignorada nessa operacao.
- Para NuvemShop, alguns fluxos de atualizacao usam identificadores no formato `productId:variantId`.
- Na NuvemShop, a atualizacao pode envolver `PUT /products/{productId}` para dados do produto e `PUT /products/{productId}/variants/{variantId}` para variacao, com `visible=true`.
- Na NuvemShop, a ativacao e logica e usa `PUT /products/{productId}/variants/{variantId}` com `visible=true`.
- Na NuvemShop, a exclusao e logica e usa `PUT /products/{productId}/variants/{variantId}` com `visible=false`.
- Na Tray, a atualizacao usa `PUT /products/{id}` com `available=1`.
- Na Tray, a ativacao e logica e usa `PUT /products/{id}` com `available=1`.
- Na Tray, a exclusao e logica e usa `PUT /products/{id}` com `available=0`.
- Na Tray, o campo `weight` e normalizado para inteiro positivo, com fallback minimo de `1`.
- Em `consultarProduto`, quando a plataforma responder `404`, a lib trata isso como ausencia de cadastro e nao inclui aquela integracao na lista de retorno.
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

List<ResultadoIntegracao<ProdutoOutput>> resultados = produtoService.consultarProduto(ids);
```
