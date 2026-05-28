# Integracao Pedizap

## Credenciais

Pedizap usa token externo enviado no header `Authorization`.

```java
Pedizap pedizap = new Pedizap("https://dominiodasualoja.com.br", "TOKEN");
```

`isCredenciaisValidas(token, apiAddress)` valida as credenciais com `GET /api/produto?limit=1`.

## Produtos

`ProdutoService` usa os endpoints:

- `POST /api/produto`
- `GET /api/produto/{id}`
- `PUT /api/produto/{id}`
- `DELETE /api/produto/{id}`
- `POST /api/produto/{id}/fotos`

Campos Pedizap em `ProdutoInput`:

- `pedizapCategoriaId`: enviado como `category_id` e obrigatorio no cadastro.
- `pedizapContent`: enviado como `content`.
- `pedizapResume`: enviado como `resume`.

Campos comuns reaproveitados:

- `referencia -> code`
- `nome -> name`
- `valorVenda -> price`
- `quantidadeEstoque -> estoque_quantidade`
- peso e dimensoes seguem os nomes da API Pedizap.

## Pedidos

`PedidoService` usa:

- `GET /api/pedido`
- `GET /api/pedido/{id}`
- `PUT /api/pedido/{id}`

A listagem e paginada com `limit=50`, `page` e `returnLayout=cliente,itens`.

Mapeamento de status:

- `1 Aberto` e `7 Carrinho -> ABERTO`
- `2 Confirmado` e `6 Finalizado -> FECHADO`
- `3 Cancelado -> CANCELADO`
- `4 Pendente -> PAUSADO`
- `5 Postado -> EM_TRANSITO`

Nota fiscal e gravada no pedido com os campos `nfe_situacao`, `nfe_dataemissao`, `nfe_numero`, `nfe_serie`,
`nfe_chaveacesso` e `nfe_linkimpressao`.

## Clientes

`ClienteService` implementa CRUD com `/api/cliente`.

`ClienteInput` aceita dados de pessoa, documento, contato, data de nascimento, tabela de preco e endereco basico.

## Categorias

`ProdutoCategoriaService` implementa CRUD com `/api/categoria`.

`ProdutoCategoriaInput` aceita `nome`, `descricao`, `resumo`, `parentId`, `friendlyUrl`, `published`, `priority`, `count`
e `resultOrder`. Como a API documentada nao expoe `resume` para categoria, `resumo` e usado como fallback para
`description`.
