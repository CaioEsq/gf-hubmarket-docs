# Documentação dos novos campos

Abaixo estão os campos adicionados às tabelas via script SQL. Todos os campos são `NULL` por padrão.

1. Tabela: `configuracaobean`  
   - Campo: `nuvemshopIdLoja` — Tipo: `VARCHAR(20)`  
     Descrição: Identificador da loja na Nuvemshop. Usado para mapear configurações por loja. Exemplo: `12345`.
   - Campo: `nuvemshopToken` — Tipo: `VARCHAR(255)`  
     Descrição: Token de acesso da Nuvemshop (OAuth). Tratar como dado sensível; armazenar/criptografar conforme política de segurança.
   - Campo: `trayUrl` — Tipo: `VARCHAR(255)`  
     Descrição: URL base da integração com Tray para esta configuração.
   - Campo: `trayCode` — Tipo: `VARCHAR(255)`  
     Descrição: Código de cliente/integração para Tray.
   - Campo: `trayToken` — Tipo: `VARCHAR(255)`  
     Descrição: Token de acesso para Tray. Tratar como sensível.
   - Campo: `trayRefreshToken` — Tipo: `VARCHAR(255)`  
     Descrição: Refresh token para renovar `trayToken`.

2. Tabela: `davcabecalho`  
   - Campo: `idNuvemshop` — Tipo: `VARCHAR(50)`  
     Descrição: Identificador externo (Nuvemshop) associado ao documento/fatura no cabeçalho.
   - Campo: `idTray` — Tipo: `VARCHAR(50)`  
     Descrição: Identificador externo (Tray) associado ao documento/fatura no cabeçalho.

3. Tabela: `produotos`  
   - Campo: `idNuvemshop` — Tipo: `VARCHAR(50)`  
     Descrição: Identificador do produto na Nuvemshop para sincronização.
   - Campo: `idTray` — Tipo: `VARCHAR(50)`  
     Descrição: Identificador do produto na Tray para sincronização.


```sql
USE `database`;
ALTER TABLE `configuracaobean`
    ADD COLUMN `nuvemshopIdLoja`  VARCHAR(20)  NULL,
    ADD COLUMN `nuvemshopToken`   VARCHAR(255) NULL,
    ADD COLUMN `trayUrl`          VARCHAR(255) NULL,
    ADD COLUMN `trayCode`         VARCHAR(255) NULL,
    ADD COLUMN `trayToken`        VARCHAR(255) NULL,
    ADD COLUMN `trayRefreshToken` VARCHAR(255) NULL;

ALTER TABLE `davcabecalho`
    ADD COLUMN `idNuvemshop` VARCHAR(50) NULL,
    ADD COLUMN `idTray`      VARCHAR(50) NULL;
ALTER TABLE `produotos`
    ADD COLUMN `idNuvemshop` VARCHAR(50) NULL,
    ADD COLUMN `idTray`      VARCHAR(50) NULL;

```
