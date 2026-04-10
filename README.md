# gf-hubmarket

Biblioteca Java para integracao com marketplaces, com foco no consumo via servicos de dominio.

O fluxo esperado de uso e:

1. Buscar no banco as credenciais configuradas para o cliente.
2. Instanciar uma integracao concreta, como `Tray` ou `Nuvemshop`.
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

## Documentação

- [Aplicação](docs/APLICACAO.md)
- [Guia de campos a serem adicionados no BD](docs/NOVOS_CAMPOS_SQL.md)
- [Produtos e Pedidos](docs/PRODUTOS_E_PEDIDOS.md)
- [Integração Tray](docs/INTEGRACAO_TRAY.md)
- [Integração NuvemShop](docs/INTEGRACAO_NUVEMSHOP.md)
