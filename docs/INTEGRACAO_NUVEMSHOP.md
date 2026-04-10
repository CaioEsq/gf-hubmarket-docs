# Integracao NuvemShop

## Quando usar

Use `br.com.herasistemas.service.nuvemshop.Nuvemshop` quando o cliente precisar operar utilizando a API NuvemShop.

Essa classe deve ser instanciada com as credenciais carregadas do banco para o cliente corrente.

## Variaveis necessarias

- `nuvemshopToken`: token de acesso a API da NuvemShop. Deve ser persistido pela aplicacao apos o processo de autorizacao.
- `nuvemshopStoreId`: identificador da loja na NuvemShop, necessario para montar as rotas da API.
- `nuvemshopCodigo`: codigo retornado no processo de autorizacao da NuvemShop, necessario para gerar as credenciais.

## Fluxo de geracao de credenciais

```java
// Primeiro, levamos o cliente ate a tela de autorizacao da NuvemShop.
Nuvemshop nuvemshop = new Nuvemshop();
String url = nuvemshop.retornarUrlLoginCliente();

// O cliente acessa essa URL, autoriza a integracao e a aplicacao deve ouvir
// o callback configurado para capturar o parametro "code".
String codigo = getRequestParam("code");
nuvemshop.gerarCredenciais(codigo);

// Apos gerar as credenciais, token e storeId ficam disponiveis para persistencia.
String token = nuvemshop.getToken();
String storeId = nuvemshop.getStoreId();
empresaController.persistirCredenciaisNuvemshop(token, storeId);

List<Integracao> integracoes = Arrays.asList(nuvemshop);

ProdutoService produtoService = new ProdutoService(integracoes);
PedidoService pedidoService = new PedidoService(integracoes);
```

## Validacao de credenciais

```java
boolean valido = nuvemshop.isCredenciaisValidas(
        nuvemshop.getToken(),
        nuvemshop.getStoreId()
);
```

A NuvemShop nao usa refresh token nesse fluxo atual da lib. O `storeId` e obrigatorio para montar as rotas da API.

## Geracao de credenciais

Se a aplicacao estiver implementando o fluxo de autorizacao:

```java
try {
    nuvemshop.gerarCredenciais("codigo-retornado-no-callback");
} catch (Exception e) {
    System.err.println("Erro ao gerar credenciais para NuvemShop: " + e.getMessage());
}
```

Depois disso, os novos valores podem ser persistidos pela aplicacao em seu proprio banco:

- `nuvemshop.getToken()`
- `nuvemshop.getStoreId()`

## Fluxo ideal

1. Checar se as credenciais existem no banco e parecem validas usando `isCredenciaisValidas()`.
2. Se as credenciais nao forem validas, verificar se o `code` esta persistido no BD para tentar gerar novas credenciais.
3. Se nao for possivel gerar novas credenciais, o cliente deve ser redirecionado para a tela de autorizacao da NuvemShop para obter um novo codigo e repetir o processo.
4. Se as credenciais parecerem validas, carregar os valores de `token` e `storeId` na integracao para uso normal.

Exemplo:

```java
String tokenNuvemshop = empresaBean.getNuvemshopToken();
String storeIdNuvemshop = empresaBean.getNuvemshopStoreId();

Nuvemshop nuvemshop = new Nuvemshop();
if (!nuvemshop.isCredenciaisValidas(tokenNuvemshop, storeIdNuvemshop)) {
    String codigoNuvemshop = empresaBean.getNuvemshopCodigo();
    try {
        nuvemshop.gerarCredenciais(codigoNuvemshop);
    } catch (Exception e) {
        System.err.println("Erro ao gerar credenciais para NuvemShop: " + e.getMessage());
    }
} else {
    nuvemshop.setToken(tokenNuvemshop);
    nuvemshop.setStoreId(storeIdNuvemshop);
}
```

## Produtos na NuvemShop

O cliente usa `ProdutoService` para executar as operacoes de produto.

```java
ProdutoService produtoService = new ProdutoService(Arrays.asList(nuvemshop));
```

Observacoes importantes:

- O cadastro retorna um `id` que pode conter a combinacao de `productId:variantId`.
- Em atualizacoes, a biblioteca executa primeiro `PUT /products/{productId}` para os dados do produto.
- Para variacao, a biblioteca usa `PUT /products/{productId}/variants/{variantId}` com `visible=true`.
- Preco e estoque da NuvemShop passam a ser atualizados por esse mesmo endpoint de variacao, usando o `idIntegracao` no formato `productId:variantId`.
- A ativacao de produto na NuvemShop e logica: a biblioteca usa `PUT /products/{productId}/variants/{variantId}` com `visible=true`.
- A exclusao de produto na NuvemShop e logica: a biblioteca usa `PUT /products/{productId}/variants/{variantId}` com `visible=false`.
- O `PUT /products/{productId}` e o ponto de falha esperado para produto inexistente na atualizacao.
- Quando a API rejeita imagens remotas no cadastro, a biblioteca devolve uma mensagem amigavel indicando qual URL nao estava acessivel pela Nuvemshop.

## Pedidos na NuvemShop

O cliente usa `PedidoService` para executar as operacoes de pedido.

```java
PedidoService pedidoService = new PedidoService(Arrays.asList(nuvemshop));
```

Operacoes tipicas:

- consultar lista de pedidos
- consultar pedido por ID
- alterar status do pedido
- incluir ou excluir produto do pedido nao sao suportados pela API atual da Nuvemshop e a biblioteca ignora essa operacao quando ela for solicitada
- validar nota fiscal do pedido
- cadastrar nota fiscal do pedido
- consultar lista de pedidos com paginacao automatica ate esgotar o resultado da API
- aplicar timeout total no fluxo paginado para evitar loops ou consultas prolongadas indefinidamente

Nas consultas de produto ou pedido, caso a API retorne um `404` a integracao deixa de ser incluida no retorno dessa consulta.

## Nota fiscal na NuvemShop

A NuvemShop nao expõe uma API de nota fiscal dedicada. Para isso, a biblioteca usa o padrao oficial de `Metafields.Invoice`, gravando a lista de NFes do pedido em `namespace=nfe` e `key=list`.

Comportamento implementado:

- `validarNotaFiscalPedido` consulta `GET /metafields/orders` filtrando `owner_id`, `namespace=nfe` e `key=list`.
- Quando o metafield existir, a biblioteca decodifica o `value` e procura a nota pela `chave` e, se necessario, pelo `link`.
- `cadastrarNotaFiscalPedido` valida primeiro para evitar duplicidade.
- Se o metafield ainda nao existir, a biblioteca cria `POST /metafields`.
- Se o metafield ja existir, a biblioteca faz append da nova NFe no array atual e envia `PUT /metafields/{id}`.

Campos usados na gravacao da NFe:

- `chave`
- `link`

Os demais campos do input de nota fiscal continuam disponiveis para a aplicacao, mas a NuvemShop persiste oficialmente apenas os dados do array de NFes no metafield.

## Conversao de status

O cliente da lib sempre usa `StatusPedido`, e a implementacao da NuvemShop converte isso para os valores da API.

Mapeamentos relevantes hoje:

- `ABERTO` -> `open`
- `FECHADO` -> `closed`
- `CANCELADO` -> `cancelled`
- `EM_TRANSITO` -> mantem `open`
- `PAUSADO` -> mantem `open`

## Exemplo de alteracao de status

```java
Map<Integracoes, String> idsPedido = new EnumMap<>(Integracoes.class);

idsPedido.put(Integracoes.NUVEMSHOP, "1800321711");

PedidoService pedidoService = new PedidoService(Arrays.asList(nuvemshop));

List<ResultadoIntegracao<PedidoOutput>> resultados =
        pedidoService.alterarStatusPedido(idsPedido, StatusPedido.CANCELADO);
```

## Observacoes

- O `storeId` e obrigatorio para qualquer chamada da API.
- O token deve ser persistido pela aplicacao consumidora.
- O carregamento dessas credenciais deve ser feito pela aplicacao a partir do banco do cliente antes de instanciar a integracao.
