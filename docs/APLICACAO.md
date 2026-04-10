# Aplicacao

## Objetivo

O `gf-hubmarket` e uma biblioteca Java para centralizar o consumo das integracoes de marketplace do software.

O contrato publico esperado para quem importar a lib e simples:

1. Buscar no banco as credenciais da integracao configuradas para o cliente.
2. Instanciar a integracao concreta com essas credenciais.
3. Montar a lista de `Integracao`.
4. Entregar essa lista para `ProdutoService` e `PedidoService`.

## Conceitos principais

- `Integracao`: contrato base que representa uma plataforma externa, como Tray ou NuvemShop.
- `ProdutoService`: fachada de dominio para operacoes de produto.
- `PedidoService`: fachada de dominio para operacoes de pedido.
- `ResultadoIntegracao<T>`: wrapper padrao do retorno, contendo integracao, sucesso, dados e erro.

## Fluxo recomendado por integração:

- [TRAY](INTEGRACAO_TRAY.md)
- [NUVEMSHOP](INTEGRACAO_NUVEMSHOP.md)
