# Documentacao dos novos campos

Abaixo estao os campos adicionados as tabelas via script SQL. Todos os campos sao `NULL` por padrao.

1. Tabela: `lojacatalogodigital`
    - Campo: `nuvemshopIdLoja` - Tipo: `VARCHAR(20)`
      Descricao: Identificador da loja na Nuvemshop. Usado para mapear configuracoes por loja. Exemplo: `12345`.
    - Campo: `nuvemshopToken` - Tipo: `VARCHAR(255)`
      Descricao: Token de acesso da Nuvemshop (OAuth). Tratar como dado sensivel; armazenar/criptografar conforme
      politica de seguranca.
    - Campo: `trayUrl` - Tipo: `VARCHAR(255)`
      Descricao: URL base da integracao com Tray para esta configuracao.
    - Campo: `trayCode` - Tipo: `VARCHAR(255)`
      Descricao: Codigo de cliente/integracao para Tray.
    - Campo: `trayToken` - Tipo: `VARCHAR(255)`
      Descricao: Token de acesso para Tray. Tratar como sensivel.
    - Campo: `trayRefreshToken` - Tipo: `VARCHAR(255)`
      Descricao: Refresh token para renovar `trayToken`.
    - Campo: `bagyToken` - Tipo: `VARCHAR(255)`
      Descricao: Token Bearer da Bagy/Dooca. Tratar como dado sensivel.

2. Tabela: `pedidoscabecalhos`
    - Campo: `idNuvemshop` - Tipo: `VARCHAR(50)`
      Descricao: Identificador externo (Nuvemshop) associado ao documento/fatura no cabecalho.
    - Campo: `idTray` - Tipo: `VARCHAR(50)`
      Descricao: Identificador externo (Tray) associado ao documento/fatura no cabecalho.
    - Campo: `idBagy` - Tipo: `VARCHAR(50)`
      Descricao: Identificador externo (Bagy) associado ao documento/fatura no cabecalho.

3. Tabela: `produtoscatalogodigital`
    - Campo: `idNuvemshop` - Tipo: `VARCHAR(50)`
      Descricao: Identificador do produto na Nuvemshop para sincronizacao.
    - Campo: `idTray` - Tipo: `VARCHAR(50)`
      Descricao: Identificador do produto na Tray para sincronizacao.
    - Campo: `idBagy` - Tipo: `VARCHAR(50)`
      Descricao: Identificador do produto na Bagy para sincronizacao. Quando houver variacao, usar
      `productId:variationId`.

```sql
USE `database`;
ALTER TABLE `lojacatalogodigital`
    ADD COLUMN `nuvemshopIdLoja`  VARCHAR(20)  NULL,
    ADD COLUMN `nuvemshopToken`   VARCHAR(255) NULL,
    ADD COLUMN `trayUrl`          VARCHAR(255) NULL,
    ADD COLUMN `trayCode`         VARCHAR(255) NULL,
    ADD COLUMN `trayToken`        VARCHAR(255) NULL,
    ADD COLUMN `trayRefreshToken` VARCHAR(255) NULL,
    ADD COLUMN `bagyToken`        VARCHAR(255) NULL;

ALTER TABLE `pedidoscabecalhos`
    ADD COLUMN `idNuvemshop` VARCHAR(50) NULL,
    ADD COLUMN `idTray`      VARCHAR(50) NULL,
    ADD COLUMN `idBagy`      VARCHAR(50) NULL;
ALTER TABLE `produtoscatalogodigital`
    ADD COLUMN `idNuvemshop` VARCHAR(50) NULL,
    ADD COLUMN `idTray`      VARCHAR(50) NULL,
    ADD COLUMN `idBagy`      VARCHAR(50) NULL;
```
