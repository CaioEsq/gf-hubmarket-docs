# gf-hubmarket

Biblioteca Java para integracao com marketplaces, com foco no consumo via servicos de dominio.

O fluxo esperado de uso e:

1. Buscar no banco as credenciais configuradas para o cliente.
2. Instanciar uma integracao concreta, como `Tray`, `Nuvemshop`, `Bagy` ou `LojaIntegrada`.
3. Passar a lista de `Integracao` para `ProdutoService` e `PedidoService`.
4. Consumir os metodos de dominio, sem montar requests HTTP manualmente.

## Exemplo rapido

```java
Nuvemshop nuvemshop = new Nuvemshop();
nuvemshop.setToken(configuracaoCliente.getNuvemshopToken());
nuvemshop.setStoreId(configuracaoCliente.getNuvemshopStoreId());

PedidoService pedidoService = new PedidoService(Arrays.asList(nuvemshop));
ProdutoService produtoService = new ProdutoService(Arrays.asList(nuvemshop));
```

```java
LojaIntegrada lojaIntegrada = new LojaIntegrada();
lojaIntegrada.setChaveApi(configuracaoCliente.getLojaIntegradaChaveApi());
lojaIntegrada.setChaveAplicacao(configuracaoCliente.getLojaIntegradaChaveAplicacao());

PedidoService pedidoService = new PedidoService(Arrays.asList(lojaIntegrada));
ProdutoService produtoService = new ProdutoService(Arrays.asList(lojaIntegrada));
```

## Documentacao

- [Aplicacao](docs/APLICACAO.md)
- [Guia de campos a serem adicionados no BD](docs/NOVOS_CAMPOS_SQL.md)
- [Produtos e Pedidos](docs/PRODUTOS_E_PEDIDOS.md)
- [Integracao Tray](docs/INTEGRACAO_TRAY.md)
- [Integracao NuvemShop](docs/INTEGRACAO_NUVEMSHOP.md)
- [Integracao Bagy](docs/INTEGRACAO_BAGY.md)
- [Integracao LojaIntegrada](docs/INTEGRACAO_LOJAINTEGRADA.md)
