# Integracao Tray

## Quando usar

Use `br.com.herasistemas.service.tray.Tray` quando o cliente precisar integrar a API da Tray.

Essa classe representa a integracao concreta e deve ser instanciada com as credenciais carregadas do banco para o cliente corrente.

## Fluxo de geracao de credenciais

1. No painel da Tray, o cliente precisa localizar o aplicativo "Gestao Facil" e autorizar a integracao.
2. Nesse processo a Tray entrega a `URL` da loja e o `Codigo` de autorizacao. Esses dados devem ser persistidos.
3. Na aplicacao, use a `URL` para configurar `apiAddress` e o `Codigo` para gerar `token` e `refreshToken`.

```java
String urlTray = "https://1225878.commercesuite.com.br/web_api";
String codigoTray = "codigo-callback";

Tray tray = new Tray();
tray.setApiAddress(urlTray);

try {
    tray.gerarCredenciais(codigoTray);
} catch (Exception e) {
    System.err.println("Erro ao gerar credenciais para Tray: " + e.getMessage());
}

String token = tray.getToken();
String refreshToken = tray.getRefreshToken();
empresaController.persistirCredenciaisTray(token, refreshToken);

List<Integracao> integracoes = Arrays.asList(tray);
ProdutoService produtoService = new ProdutoService(integracoes);
PedidoService pedidoService = new PedidoService(integracoes);
```

## Validacao de credenciais

Para validar se as credenciais carregadas do banco parecem prontas para uso:

```java
boolean valido = tray.isCredenciaisValidas(
        tray.getToken(),
        tray.getRefreshToken(),
        tray.getApiAddress()
);
```

Se a API responder `401`, a implementacao da Tray tenta renovar o token usando o `refreshToken`.

## Fluxo recomendado

1. Validar `token`, `refreshToken` e `apiAddress` com `isCredenciaisValidas`.
2. Se as credenciais nao forem validas, tentar gerar novas usando o `code` persistido.
3. Se nao for possivel renovar, redirecionar o Cliente para um Link de Tutorial ou o site da Tray para Habilitar a Integração no painel.
4. Quando as credenciais estiverem validas, carregar `token`, `refreshToken` e `apiAddress` na integracao antes de usar os servicos.

## Produtos na Tray

Com a integracao pronta, use `ProdutoService`.

```java
ProdutoService produtoService = new ProdutoService(Arrays.asList(tray));
```

Operacoes suportadas:

- cadastrar produto
- consultar produto
- atualizar produto
- excluir produto

Os IDs usados pela Tray sao IDs da propria plataforma.

Observacoes importantes:

- A atualizacao de produto na Tray automaticamente ativa o produto na plataforma, caso esteja inativo ou indisponível.
- A ativacao de produto na Tray usa `PUT /products/{id}` com `available=1` em um payload minimo de disponibilidade.
- A exclusao de produto na Tray e logica: a biblioteca usa `PUT /products/{id}` com `available=0`.

Quando a Tray responder `404` em `consultarProduto`, a biblioteca trata isso como produto inexistente para o identificador informado e nao inclui aquela integracao no retorno.

## Pedidos na Tray

Com a integracao pronta, use `PedidoService`.

```java
PedidoService pedidoService = new PedidoService(Arrays.asList(tray));
```

Operacoes suportadas hoje:

- consultar lista de pedidos
- consultar pedido por ID
- alterar status do pedido
- incluir produto no pedido
- excluir produto do pedido
- validar nota fiscal do pedido
- cadastrar nota fiscal do pedido

### Como a biblioteca consulta o pedido

- Para `consultarPedido`, a implementacao da Tray tenta primeiro o endpoint completo `/orders/{id}/complete`.
- Se a loja nao suportar esse endpoint, a implementacao faz fallback para `/orders/{id}`.
- O `PedidoOutput` passa a ser preenchido com cliente, endereco e itens quando esses dados vierem no payload completo da Tray.
- Se a consulta por periodo vier sem pedidos, a biblioteca simplesmente nao adiciona resultados daquela integracao.
- Em `consultarListaPedidos`, a listagem da Tray e usada apenas para localizar os IDs dos pedidos; depois a biblioteca hidrata cada pedido com a consulta completa por ID para evitar retornar o payload parcial da listagem.
- A consulta de lista usa paginacao automatica com `page` e `limit`, repetindo chamadas ate esgotar as paginas retornadas pela Tray.
- Existe uma protecao de timeout total na paginacao para evitar loops ou sincronizacoes presas por tempo indefinido.

### Como a biblioteca altera status

- A biblioteca consulta `/orders/statuses` para localizar o `status_id` correspondente.
- Depois envia `PUT /orders/{id}` com o `status_id`.
- Se a Tray devolver o pedido completo no proprio `PUT`, esse retorno ja e usado.
- Se a Tray devolver apenas a resposta generica (`message`, `id`, `code`), a biblioteca reconsulta o pedido para montar o `PedidoOutput`.

Isso evita depender da API de clientes para o fluxo principal de pedidos, porque o endpoint completo do pedido ja traz `Customer` e `CustomerAddresses`.

### Como a biblioteca inclui ou exclui produto do pedido

- Para incluir item, a biblioteca envia `POST /orders/includeProduct/{order_id}` com o objeto `ProductsSold`.
- Para excluir item, a biblioteca envia `DELETE /orders/excludeProduct/{order_id}/{product_id}`.
- Em ambos os casos, apos o sucesso a biblioteca reconsulta o pedido completo e devolve `PedidoOutput`.

## Nota fiscal na Tray

A Tray possui a API propria de nota fiscal em `orders/:id/invoices`, e a biblioteca passa a usar esse recurso diretamente.

Comportamento implementado:

- `validarNotaFiscalPedido` consulta `GET /orders/{id}/invoices`.
- A validacao procura a nota pela `chave` e, como fallback, por `numero` + `serie` ou `link`.
- `cadastrarNotaFiscalPedido` sempre valida antes para nao duplicar a nota no pedido.
- Quando a nota ainda nao existir, a biblioteca envia `POST /orders/{id}/invoices` em JSON, seguindo o shape documentado do endpoint.
- Se a Tray responder que `number`, `serie` e `key` ja existem, a biblioteca reconsulta as notas do pedido e devolve a nota como existente em vez de manter erro de duplicidade.
- Se essa duplicidade existir em outro pedido, a biblioteca devolve uma mensagem funcional informando que a nota fiscal ja esta vinculada a outro pedido na Tray.
- Apos o cadastro, a biblioteca consulta `GET /orders/{order_id}/invoices/{invoice_id}` para retornar a nota completa com os dados de `ProductCfop`.

Campos esperados no cadastro da nota:

- `dataEmissao`
- `numero`
- `serie`
- `valor`
- `chave`
- `link`
- `xmlDanfe` opcional
- `produtosCfop` opcional

## Conversao de status

O cliente da lib sempre envia `StatusPedido`, e a implementacao converte para os nomes esperados pela Tray.

Mapeamentos relevantes:

- `ABERTO` -> `A ENVIAR`
- `FECHADO` -> `FINALIZADO`
- `CANCELADO` -> `CANCELADO`
- `EM_TRANSITO` -> `ENVIADO`
- `PAUSADO` -> `A ENVIAR`

## Exemplo de alteracao de status

```java
Map<Integracoes, String> idsPedido = new EnumMap<>(Integracoes.class);
idsPedido.put(Integracoes.TRAY, "16045");

PedidoService pedidoService = new PedidoService(Arrays.asList(tray));
List<ResultadoIntegracao<PedidoOutput>> resultados =
        pedidoService.alterarStatusPedido(idsPedido, StatusPedido.FECHADO);
```

## Observacoes

- A `apiAddress` precisa apontar para a URL base da loja na Tray.
- Sem `apiAddress` ou com a URL incorreta, as URLs nao conseguem ser montadas e a integracao nao funciona.
- Para uso em producao, persista `token`, `refreshToken`, `apiAddress` e dados da loja no banco da aplicacao.
