# Teste de Conhecimento SQL com SQL Server

### Teste para apidão de conhecimentos solicitado pela empresa CSJ Sistemas

Primeiramente iremos criar o banco de dados que foi solicitado para os 11 exercícios solicitados na documentação, o nosso cliente possui um pequeno sistema de vendas, cujo banco de dados tem as tabelas abaixo, com suas respectivas estruturas, tabelas solicitadas tbProdutos, tbClientes, tbPedidos, tbPedidosItens. 

``` sql
    CREATE DATABASE dbCSJSistemas
```



Após a criação do banco de dados temos que selecionar o banco de dados para realizar a criação das entidades e posteriormente realizar os exercícios.

``` sql
    USE dbCSJSistemas
```



Agora vamos criar as tabelas solicitada:

**Tabelas Extras**
``` sql
  CREATE TABLE tbPaises(
    ID INT IDENTITY(1,1),
    SIGLA VARCHAR(2) NOT NULL,
    DESCRICAO VARCHAR(60) NOT NULL,
    CONSTRAINT  PK_PAIS PRIMARY KEY CLUSTERED (ID) 
  )

  CREATE TABLE tbEstados(
    ID INT IDENTITY(1,1),
    SIGLA VARCHAR(2) NOT NULL,
    DESCRICAO VARCHAR(60) NOT NULL,
    FK_PAIS INT NOT NULL,
    CONSTRAINT  PK_ESTADO PRIMARY KEY CLUSTERED (ID),
    CONSTRAINT  FK_PAIS__ESTADO FOREIGN KEY (FK_PAIS) REFERENCES tbPaises (ID) 
                ON DELETE CASCADE
                ON UPDATE CASCADE
  )

  CREATE TABLE tbCidades(
    ID INT IDENTITY(1,1),
    SIGLA VARCHAR(5) NOT NULL,
    DESCRICAO VARCHAR(100),
    FK_ESTADO INT NOT NULL,
    CONSTRAINT  PK_CIDADE PRIMARY KEY CLUSTERED (ID),
    CONSTRAINT  FK_ESTADO__CIDADE FOREIGN KEY (FK_ESTADO) REFERENCES tbEstados (ID)
                ON DELETE CASCADE
                ON UPDATE CASCADE
  )

  CREATE TABLE tbBairros(
    ID INT IDENTITY(1,1),
    SIGLA VARCHAR(5) NOT NULL,
    DESCRICAO VARCHAR(150) NOT NULL,
    FK_CIDADE INT NOT NULL,
    CONSTRAINT  PK_BAIRRO PRIMARY KEY CLUSTERED (ID),
    CONSTRAINT  FK_CIDADE__BAIRRO FOREIGN KEY (FK_CIDADE) REFERENCES tbCidades (ID)
                ON DELETE CASCADE
                ON UPDATE CASCADE
  )

  CREATE TABLE tbCep(
    ID INT IDENTITY(1,1),
    CEP INT NOT NULL,
    FK_BAIRRO INT NOT NULL,
    RUA VARCHAR(150) NOT NULL,
    CONSTRAINT  PK_ID PRIMARY KEY CLUSTERED (ID),
    CONSTRAINT  FK_BAIRRO__CEP FOREIGN KEY (FK_BAIRRO) REFERENCES tbBairros (ID)
                ON DELETE CASCADE
                ON UPDATE CASCADE
  )

  CREATE TABLE tbCategoria(
    ID INT IDENTITY(1,1),
    CODIGO VARCHAR(20) NOT NULL,
    DESCRICAO VARCHAR(50) NOT NULL,
    ESTOQUE_MIN DECIMAL(18,2) NOT NULL DEFAULT 0.0,
    ESTOQUE_MAX DECIMAL(18,2) NOT NULL DEFAULT 0.0,
    GRAU_RISCO_PERDA DECIMAL(18,2) NOT NULL DEFAULT 0.0,
    CONSTRAINT  PK_CATEGORIA PRIMARY KEY CLUSTERED (ID),
    CONSTRAINT  UQ_CODIGO_CATEGORIA UNIQUE NONCLUSTERED (CODIGO)
  )
```



1. Tabela de Produtos: **tbProdutos**

|CHAVE|CAMPO|DESCRIÇÃO|TIPO|TAM|
|---|---|---|---|---|
|PK|ID|Registro|int|Identity|
|UNQ|Codigo|Código Produto|char|20|
|---|Descricao|Descricao Produto|char|100|
|---|Grupo|Grupo de Estoque|char|10
|---|Preco|Preço da Venda|money|

```sql
  CREATE TABLE tbProdutos(
    ID INT IDENTITY(1,1),
    CODIGO VARCHAR(20) NOT NULL,
    DESCRICAO VARCHAR(100) NOT NULL,
    FK_CATEGORIA INT NOT NULL,
    QTD_ESTOQUE DECIMAL(18,2) NOT NULL DEFAULT 0.0,
    PRECO DECIMAL(18,2) NOT NULL DEFAULT 0.0,
    CONSTRAINT  PK_PRODUTOS PRIMARY KEY CLUSTERED (ID), 
    CONSTRAINT  UQ_CODIGO_PRODUTO UNIQUE NONCLUSTERED (CODIGO),
    CONSTRAINT  FK_REFERENCES FOREIGN KEY (FK_CATEGORIA) REFERENCES tbCategoria (ID)
                ON DELETE CASCADE
                ON UPDATE CASCADE
  )
```


2. Tabela de Clientes: **tbClientes**

|CHAVE|CAMPO|DESCRIÇÃO|TIPO|TAM|
|---|---|---|---|---|
|PK|CPF|CPF do Cliente|char|20|
|---|Nome|Nome do Cliente|char|10|
|---|Cidade|Cidade do Cliente|char|30|
|---|Bairro|Bairro do cliente|char|30|

```sql
  CREATE TABLE tbClientes(
    ID INT IDENTITY(1,1),
    CPF VARCHAR(11) NOT NULL,
    NOME VARCHAR(10) NOT NULL,
    SOBRENOME VARCHAR(10) NOT NULL,
    FK_CEP INT NOT NULL,
    NUMERO INT NOT NULL,
    CONSTRAINT  PK_CPF PRIMARY KEY CLUSTERED (CPF),
    CONSTRAINT  CK_CPF CHECK (LEN(CPF) = 11),
    CONSTRAINT  FK_CEP__CLIENTES FOREIGN KEY (FK_CEP) REFERENCES tbCep (ID)
                ON DELETE CASCADE
                ON UPDATE CASCADE
  )
```


3. Tabela de Vendas **tbVendas**

|CHAVE|CAMPO|DESCRIÇÃO|TIPO|TAM|
|---|---|---|---|---|
|PK|ID|Registro|Int|Identity|
|UNQ|Pedido|Numero do pedido|char|20|
|---|Data|Data do Pedido|Date|---|
|FK|ClienteCPF|Identificação do Cliente|char|20|

```sql
   CREATE TABLE tbPedidos(
    ID INT IDENTITY(1,1),
    PEDIDO VARCHAR(20) NOT NULL,
    DATA_HORA DATETIME DEFAULT CURRENT_TIMESTAMP,
    FK_CLIENTE VARCHAR(11) NOT NULL,
    VALOR_VENDA DECIMAL(18,2) DEFAULT 0.0,
    CONSTRAINT  PK_PEDIDOS PRIMARY KEY CLUSTERED (ID),
    CONSTRAINT  UQ_PEDIDO UNIQUE NONCLUSTERED (PEDIDO),
    CONSTRAINT  FK_CLIENTE__PEDIDO FOREIGN KEY (FK_CLIENTE) REFERENCES tbClientes (CPF)
                ON DELETE CASCADE
                ON UPDATE CASCADE
  )
```


4. Tabela de Itens da Venda **tbPedidosItens**

|CHAVE|CAMPO|DESCRIÇÃO|TIPO|TAM|
|---|---|---|---|---|
|PK/FK|PedidoID|ID do Pedido|Int|---|
|PK/FK|ProdutoID|ID do Produto|Int|---|
|---|Qtde|Quantidade do Produto|Int|---|
|---|PrecoUnitario|Valor Unitario|money|---|
|---|Desconto|Valor do Desconto total item|money|---|

```sql
  CREATE TABLE tbPedidosItens(
    FK_PEDIDO_ID INT NOT NULL,
    FK_PRODUTO_ID INT NOT NULL,
    QUANTIDADE DECIMAL(18,2) NOT NULL DEFAULT 1.0,
    UNITARIO DECIMAL(18,2) NOT NULL DEFAULT 0.0,
    DESCONTO DECIMAL(18,2) NOT NULL DEFAULT 0.0,
    TOTAL_BRUTO DECIMAL(18,2) NOT NULL DEFAULT 0,
    TOTAL_LIQUIDO DECIMAL(18,2) NOT NULL DEFAULT 0,
    CONSTRAINT  PK_PEDIDO_ITENS PRIMARY KEY CLUSTERED (FK_PEDIDO_ID, FK_PRODUTO_ID),
    CONSTRAINT  FK_PEDIDO_ID__PEDIDO_ITENS FOREIGN KEY (FK_PEDIDO_ID) REFERENCES tbPedidos (ID)
                ON DELETE CASCADE
                ON UPDATE CASCADE,
    CONSTRAINT  FK_PRODUTO_ID__PEDIDO_ITENS FOREIGN KEY (FK_PRODUTO_ID) REFERENCES tbProdutos (ID)
                ON DELETE CASCADE
                ON UPDATE CASCADE
  )
```

### Dados Iniciais a populados antes dos exercícios solicitados:

```sql
  -- Categorias
  INSERT INTO tbCategoria (CODIGO, DESCRICAO, ESTOQUE_MAX) VALUES ('CEL', 'CELULARES', 100)
  INSERT INTO tbCategoria (CODIGO, DESCRICAO, ESTOQUE_MAX) VALUES ('CAPCEL', 'CAPAS PARA CELULARES', 300)
  INSERT INTO tbCategoria (CODIGO, DESCRICAO, ESTOQUE_MAX) VALUES ('PELCEL', 'PELICULAS PARA CELULARES', 500)

  -- Países
  INSERT INTO tbPaises (SIGLA, DESCRICAO) VALUES ('BR', 'BRASIL')
  
  -- Estados
  INSERT INTO tbEstados (SIGLA, DESCRICAO, FK_PAIS) VALUES ('SP', 'SAO PAULO', 1)
  
  -- Cidades
  INSERT INTO tbCidades (SIGLA, DESCRICAO, FK_ESTADO) VALUES ('SJC', 'SAO JOSE DOS CAMPOS', 1)
  
  -- Bairros
  INSERT INTO tbBairros (SIGLA, DESCRICAO, FK_CIDADE) VALUES ('JDAQU', 'JARDIM AQUARIUS', 1)
  
  -- Cep
  INSERT INTO tbCep (CEP, FK_BAIRRO, RUA) VALUES (12242000, 1, 'AV. SAO JOAO')
  
  -- Criando um cliente de teste
  INSERT INTO tbClientes (CPF, NOME, SOBRENOME, FK_CEP, NUMERO)
            VALUES ('00000000001', 'CLIENTE', 'TESTE 01', 1, 2405 )
  INSERT INTO tbClientes (CPF, NOME, SOBRENOME, FK_CEP, NUMERO)
            VALUES ('00000000002', 'CLIENTE', 'TESTE 02', 1, 2405 )
  INSERT INTO tbClientes (CPF, NOME, SOBRENOME, FK_CEP, NUMERO)
              VALUES ('00000000003', 'CLIENTE', 'TESTE 03', 1, 2405 )
  INSERT INTO tbClientes (CPF, NOME, SOBRENOME, FK_CEP, NUMERO)
              VALUES ('00000000004', 'CLIENTE', 'TESTE 04', 1, 2405 )
  INSERT INTO tbClientes (CPF, NOME, SOBRENOME, FK_CEP, NUMERO)
              VALUES ('00000000005', 'CLIENTE', 'TESTE 05', 1, 2405 )
  
```

### Criando procedures e triggers para auxiliar nos exercícios

``` sql
  -- Procedure para criar um pedido completo 
  CREATE PROC PROC_ADICIONAR_PEDIDO_COMPLETO
      @PEDIDO VARCHAR(20),
      @CLIENTE VARCHAR(11),
      @PRODUTO INT,
      @QUANTIDADE DECIMAL(18,2),
      @DESCONTO DECIMAL(18,2)
   AS
   BEGIN
      DECLARE @ID_PEDIDO INT;
      DECLARE @PRECO DECIMAL(18,2);
      DECLARE @QTD_ESTOQUE DECIMAL(18,2);

      SELECT @PRECO = PRECO, @QTD_ESTOQUE = QTD_ESTOQUE FROM tbProdutos WHERE ID = @PRODUTO;

      IF @QUANTIDADE < @QTD_ESTOQUE
      BEGIN
          IF @PRECO > 0
          BEGIN
              INSERT INTO tbPedidos (PEDIDO, FK_CLIENTE) VALUES (@PEDIDO, @CLIENTE);
              SELECT @ID_PEDIDO = SCOPE_IDENTITY();

              INSERT INTO tbPedidosItens (FK_PEDIDO_ID, FK_PRODUTO_ID, QUANTIDADE, UNITARIO, DESCONTO)
                          VALUES (@ID_PEDIDO, @PRODUTO, @QUANTIDADE, @PRECO, @DESCONTO)
          END
          ELSE
              PRINT('PRECO INVALIDO');
      END
      ELSE
          PRINT('QUANTIDADE INFORMADA ACIMA DO QUE TEMOS NO ESTOQUE');
  END;
  
  -- Procedure para adicionar um item no ultimo pedido criado
  CREATE PROC PROC_ADD_ITEM_ULT_PEDIDO
    @PRODUTO INT,
    @QUANTIDADE DECIMAL(18,2),
    @DESCONTO DECIMAL(18,2)
  AS
  BEGIN
      DECLARE @ID_ULTIMO_PEDIDO INT;
      DECLARE @PRECO DECIMAL(18,2);
      DECLARE @QTD_ESTOQUE DECIMAL(18,2);

      SELECT @PRECO = PRECO, @QTD_ESTOQUE = QTD_ESTOQUE FROM tbProdutos WHERE ID = @PRODUTO;

      IF @QUANTIDADE < @QTD_ESTOQUE
      BEGIN
          IF @PRECO > 0
          BEGIN
              SELECT @ID_ULTIMO_PEDIDO = MAX(ID) FROM tbPedidos;

              INSERT INTO tbPedidosItens (FK_PEDIDO_ID, FK_PRODUTO_ID, QUANTIDADE, UNITARIO, DESCONTO)
                          VALUES (@ID_ULTIMO_PEDIDO, @PRODUTO, @QUANTIDADE, @PRECO, @DESCONTO);
          END
          ELSE
              PRINT('PRECO INVALIDO');
      END
      ELSE
          PRINT('QUANTIDADE INFORMADA ACIMA DO QUE TEMOS NO ESTOQUE');
  END
  
  -- Procedure para adicionar um item passando o pedido que deseja adicionar o item
  CREATE PROC PROC_ADD_ITEM_PEDIDO
    @PRODUTO INT,
    @QUANTIDADE DECIMAL(18,2),
    @DESCONTO DECIMAL(18,2),
    @ID_PEDIDO INT
  AS
  BEGIN
      DECLARE @ID_VERIFICACAO INT;
      DECLARE @PRECO DECIMAL(18,2);
      DECLARE @QTD_ESTOQUE DECIMAL(18,2);

      SELECT @PRECO = PRECO, @QTD_ESTOQUE = QTD_ESTOQUE FROM tbProdutos WHERE ID = @PRODUTO;
      SELECT @ID_VERIFICACAO = ID FROM tbPedidos WHERE ID = @ID_PEDIDO;

      IF @ID_VERIFICACAO IS NOT NULL OR @ID_VERIFICACAO > 0
      BEGIN
          IF @QUANTIDADE < @QTD_ESTOQUE
          BEGIN
              IF @PRECO > 0
              BEGIN
                  INSERT INTO tbPedidosItens (FK_PEDIDO_ID, FK_PRODUTO_ID, QUANTIDADE, UNITARIO, DESCONTO)
                              VALUES (@ID_VERIFICACAO, @PRODUTO, @QUANTIDADE, @PRECO, @DESCONTO);
              END
              ELSE
                  PRINT('PRECO INVALIDO');
          END
          ELSE
              PRINT('QUANTIDADE INFORMADA ACIMA DO QUE TEMOS NO ESTOQUE');
      END
      ELSE
          PRINT('ID DO PEDIDO INVALIDO');
  END
  
  -- Trigger para somar o total bruto, total liquido e o valor da venda
  CREATE TRIGGER TR_INS_ITENS_PEDIDOS_UTUALIZA_TOTAL
  ON tbPedidosItens
  AFTER INSERT AS
  BEGIN 
      DECLARE @ID_PEDIDO INT;
      DECLARE @ID_PRODUTO INT;
      DECLARE @PRECO_UNITARIO DECIMAL(18,2);
      DECLARE @QUANTIDADE DECIMAL(18,2);
      DECLARE @DESCONTO DECIMAL(18,2);
      DECLARE @TOTAL_BRUTO DECIMAL(18,2);
      DECLARE @TOTAL_LIQUIDO DECIMAL(18,2);
      DECLARE @TOTAL_VENDA DECIMAL(18,2);

      SELECT  @ID_PEDIDO = FK_PEDIDO_ID, @ID_PRODUTO = FK_PRODUTO_ID,
              @PRECO_UNITARIO = UNITARIO, @QUANTIDADE = QUANTIDADE,
              @DESCONTO = DESCONTO
      FROM INSERTED;

      SET @TOTAL_BRUTO = @QUANTIDADE * @PRECO_UNITARIO;
      SET @TOTAL_LIQUIDO = @TOTAL_BRUTO * @DESCONTO / 100;

      UPDATE tbPedidosItens SET TOTAL_BRUTO = @TOTAL_BRUTO, TOTAL_LIQUIDO = @TOTAL_LIQUIDO
      WHERE FK_PEDIDO_ID = @ID_PEDIDO AND FK_PRODUTO_ID = @ID_PRODUTO;

      SELECT @TOTAL_VENDA = SUM(TOTAL_LIQUIDO) FROM tbPedidosItens WHERE FK_PEDIDO_ID = @ID_PEDIDO;

      UPDATE tbPedidos SET VALOR_VENDA = @TOTAL_VENDA;
  END
  
  
  CREATE TRIGGER TR_UPD_ITENS_PEDIDOS_UTUALIZA_TOTAL
  ON tbPedidosItens
  AFTER UPDATE AS
  BEGIN 
      DECLARE @ID_PEDIDO INT;
      DECLARE @ID_PRODUTO INT;
      DECLARE @PRECO_UNITARIO DECIMAL(18,2);
      DECLARE @QUANTIDADE DECIMAL(18,2);
      DECLARE @DESCONTO DECIMAL(18,2);
      DECLARE @TOTAL_BRUTO DECIMAL(18,2);
      DECLARE @TOTAL_LIQUIDO DECIMAL(18,2);
      DECLARE @TOTAL_VENDA DECIMAL(18,2);

      SELECT  @ID_PEDIDO = FK_PEDIDO_ID, @ID_PRODUTO = FK_PRODUTO_ID,
              @PRECO_UNITARIO = UNITARIO, @QUANTIDADE = QUANTIDADE,
              @DESCONTO = DESCONTO
      FROM UPDATED;

      SET @TOTAL_BRUTO = @QUANTIDADE * @PRECO_UNITARIO;
      SET @TOTAL_LIQUIDO = @TOTAL_BRUTO * @DESCONTO / 100;

      UPDATE tbPedidosItens SET TOTAL_BRUTO = @TOTAL_BRUTO, TOTAL_LIQUIDO = @TOTAL_LIQUIDO
      WHERE FK_PEDIDO_ID = @ID_PEDIDO AND FK_PRODUTO_ID = @ID_PRODUTO;

      SELECT @TOTAL_VENDA = SUM(TOTAL_LIQUIDO) FROM tbPedidosItens WHERE FK_PEDIDO_ID = @ID_PEDIDO;

      UPDATE tbPedidos SET VALOR_VENDA = @TOTAL_VENDA;
  END
  
  
  CREATE TRIGGER TR_DEL_ITENS_PEDIDOS_UTUALIZA_TOTAL
  ON tbPedidosItens
  AFTER DELETE AS
  BEGIN
      DECLARE @ID_PEDIDO INT;
      DECLARE @ID_PRODUTO INT;
      DECLARE @PRECO_UNITARIO DECIMAL(18,2);
      DECLARE @QUANTIDADE DECIMAL(18,2);
      DECLARE @DESCONTO DECIMAL(18,2);
      DECLARE @TOTAL_BRUTO DECIMAL(18,2);
      DECLARE @TOTAL_LIQUIDO DECIMAL(18,2);
      DECLARE @TOTAL_VENDA DECIMAL(18,2);

      SELECT  @ID_PEDIDO = FK_PEDIDO_ID, @ID_PRODUTO = FK_PRODUTO_ID,
              @PRECO_UNITARIO = UNITARIO, @QUANTIDADE = QUANTIDADE,
              @DESCONTO = DESCONTO
      FROM DELETED;

      SET @TOTAL_BRUTO = @QUANTIDADE * @PRECO_UNITARIO;
      SET @TOTAL_LIQUIDO = @TOTAL_BRUTO * @DESCONTO / 100;

      UPDATE tbPedidosItens SET TOTAL_BRUTO = @TOTAL_BRUTO, TOTAL_LIQUIDO = @TOTAL_LIQUIDO
      WHERE FK_PEDIDO_ID = @ID_PEDIDO AND FK_PRODUTO_ID = @ID_PRODUTO;

      SELECT @TOTAL_VENDA = SUM(TOTAL_LIQUIDO) FROM tbPedidosItens WHERE FK_PEDIDO_ID = @ID_PEDIDO;

      UPDATE tbPedidos SET VALOR_VENDA = @TOTAL_VENDA;
  END
```


### Exercicios solicitados:

1.Precisamos na nossa tabela de produtos 5 produtos:

- Codigo = “0001”, Descrição  = “Produto Teste 01”, Unitario = R$ 30,00 e Grupo=”CELULARES”
- Codigo = “0002”, Descrição  = “Produto Teste 02”, Unitario = R$ 35,00 e Grupo=”CELULARES”
- Codigo = “0003”, Descrição  = “Produto Teste 03”, Unitario = R$ 40,00 e Grupo=”CAPAS”
- Codigo = “0004”, Descrição  = “Produto Teste 04”, Unitario = R$ 45,00 e Grupo=”CAPAS”
- Codigo = “0005”, Descrição  = “Produto Teste 05”, Unitario = R$ 50,00 e Grupo=”CELULARES”

``` sql
  INSERT INTO tbProdutos (CODIGO, DESCRICAO, FK_CATEGORIA, QTD_ESTOQUE, PRECO)
              VALUES  ('0001', 'Produto Teste 01', 1, 100, 30.00 )
  INSERT INTO tbProdutos (CODIGO, DESCRICAO, FK_CATEGORIA, QTD_ESTOQUE, PRECO)
              VALUES  ('0002', 'Produto Teste 02', 1, 100, 35.00 )
  INSERT INTO tbProdutos (CODIGO, DESCRICAO, FK_CATEGORIA, QTD_ESTOQUE, PRECO)
              VALUES  ('0003', 'Produto Teste 03', 2, 100, 40.00 )
  INSERT INTO tbProdutos (CODIGO, DESCRICAO, FK_CATEGORIA, QTD_ESTOQUE, PRECO)
              VALUES  ('0004', 'Produto Teste 04', 2, 100, 45.00 )
  INSERT INTO tbProdutos (CODIGO, DESCRICAO, FK_CATEGORIA, QTD_ESTOQUE, PRECO)
              VALUES  ('0005', 'Produto Teste 05', 1, 100, 50.00 )
```

2. Inserir um pedido de vendas completo, contendo 10 unidades do Codigo = “0001” e 5 unidades do Codigo = “0002”. Colocar no nome do cliente CPF=”0000001”

``` sql  
  EXEC PROC_ADICIONAR_PEDIDO_COMPLETO
  @PEDIDO = 'PEDIDO TESTE 01',
  @CLIENTE = '00000000001',
  @PRODUTO = 1,
  @QUANTIDADE = 10.00,
  @DESCONTO = 0.00
  
  
  EXEC PROC_ADD_ITEM_ULT_PEDIDO
  @PRODUTO = 2,
  @QUANTIDADE = 5.00,
  @DESCONTO = 0.00
  
```

3. Inserir um pedido de vendas completo, contendo 10 unidades do Codigo=”0002” e 10 unidades do Codigo = “0003”, sendo que neste ultimo item o desconto será de 10% e colocar no nome do cliente CPF = “000002”

``` sql
  EXEC PROC_ADICIONAR_PEDIDO_COMPLETO
  @PEDIDO = 'PEDIDO TESTE 02',
  @CLIENTE = '00000000002',
  @PRODUTO = 2,
  @QUANTIDADE = 10.00,
  @DESCONTO = 0.00

  EXEC PROC_ADD_ITEM_ULT_PEDIDO
  @PRODUTO = 3,
  @QUANTIDADE = 10.00,
  @DESCONTO = 10.00
```

4. Fazer uma pesquisa dos pedidos registrados em um período, contendo os dados: Cliente, Nome, Bairro, Cidade, Pedido, data, Codigo, Descricao, Qtde, Unitario, Desconto e Total Item

``` sql
  SELECT  CLI.ID AS COD_CLIENTE,
        CLI.NOME AS NOME,
        BAI.DESCRICAO AS BAIRRO,
        CID.DESCRICAO AS CIDADE,
        CAST(PED.DATA_HORA AS DATE) AS DATA,
        PIT.FK_PRODUTO_ID AS COD_PRODUTO,
        PRO.DESCRICAO AS DESC_PRODUTO,
        PIT.QUANTIDADE AS QTD_PRODUTO,
        PRO.PRECO AS PRECO_PRODUTO,
        PIT.DESCONTO AS DESC_ITEM,
        PIT.TOTAL_BRUTO AS TOTAL_ITEM
FROM tbPedidos PED
INNER JOIN tbPedidosItens PIT ON PED.ID = PIT.FK_PEDIDO_ID
INNER JOIN tbClientes CLI ON PED.FK_CLIENTE = CLI.CPF
INNER JOIN tbProdutos PRO ON PIT.FK_PRODUTO_ID = PRO.ID
INNER JOIN tbCep CEP ON CLI.FK_CEP = CEP.ID
INNER JOIN tbBairros BAI ON CEP.FK_BAIRRO = BAI.ID
INNER JOIN tbCidades CID ON BAI.FK_CIDADE = CID.ID
WHERE CAST(PED.DATA_HORA AS DATE) BETWEEN '01-01-2020' AND '28-12-2020'

  
```

5. Fazer uma pesquisa totalizando pedidos em determinado período, com as colunas: Data, Pedido, Cliente, Nome, Bairro, Cidade, Qtde Pedidos, Total Bruto, total Descontos e Total Liquido

``` sql
  SELECT  CAST(PED.DATA_HORA AS DATE) AS DATA,
          PED.ID AS PEDIDO,
          CLI.ID AS CLIENTE,
          CLI.NOME AS NOME,
          BAI.DESCRICAO AS BAIRRO,
          CID.DESCRICAO AS CIDADE,
          COUNT(PED.ID) AS QUANTIDADE_PEDIDOS,
          SUM(PIT.TOTAL_BRUTO) AS TOTAL_BRUTO,
          SUM(PIT.DESCONTO) AS TOTAL_DESCONTO,
          SUM(PED.VALOR_VENDA) AS TOTAL_LIQUIDO
  FROM tbPedidos PED
  INNER JOIN tbPedidosItens PIT ON PED.ID = PIT.FK_PEDIDO_ID
  INNER JOIN tbClientes CLI ON PED.FK_CLIENTE = CLI.CPF
  INNER JOIN tbProdutos PRO ON PIT.FK_PRODUTO_ID = PRO.ID
  INNER JOIN tbCep CEP ON CLI.FK_CEP = CEP.ID
  INNER JOIN tbBairros BAI ON CEP.FK_BAIRRO = BAI.ID
  INNER JOIN tbCidades CID ON BAI.FK_CIDADE = CID.ID
  WHERE CAST(PED.DATA_HORA AS DATE) BETWEEN '01-01-2020' AND '28-12-2020'
  GROUP BY PED.DATA_HORA, PED.ID, CLI.ID, CLI.NOME, BAI.DESCRICAO, CID.DESCRICAO
```

6. Fazer uma pesquisa totalizando pedidos de um determinado período, agrupando por DATA, com os campos: Data, Cliente, Cidade, Qtde Pedidos, Total Bruto, total Descontos e Total Liquido

``` sql
  SELECT  CAST(PED.DATA_HORA AS DATE) AS DATA,
          CLI.ID AS CLIENTE,
          CLI.NOME AS NOME,
          CID.DESCRICAO AS CIDADE,
          COUNT(PED.ID) AS QUANTIDADE_PEDIDOS,
          SUM(PIT.TOTAL_BRUTO) AS TOTAL_BRUTO,
          SUM(PIT.DESCONTO) AS TOTAL_DESCONTO,
          SUM(PED.VALOR_VENDA) AS TOTAL_LIQUIDO
  FROM tbPedidos PED
  INNER JOIN tbPedidosItens PIT ON PED.ID = PIT.FK_PEDIDO_ID
  INNER JOIN tbClientes CLI ON PED.FK_CLIENTE = CLI.CPF
  INNER JOIN tbProdutos PRO ON PIT.FK_PRODUTO_ID = PRO.ID
  INNER JOIN tbCep CEP ON CLI.FK_CEP = CEP.ID
  INNER JOIN tbBairros BAI ON CEP.FK_BAIRRO = BAI.ID
  INNER JOIN tbCidades CID ON BAI.FK_CIDADE = CID.ID
  WHERE CAST(PED.DATA_HORA AS DATE) BETWEEN '01-01-2020' AND '28-12-2020'
  GROUP BY PED.DATA_HORA, PED.ID, CLI.ID, CLI.NOME, CID.DESCRICAO
```

7. Fazer uma pesquisa totalizando pedidos de um determinado período, agrupando pelo cliente, contendo os campos: Cliente, Cidade, Qtde Pedidos, Total Bruto, total Descontos e Total Liquido

``` sql
  SELECT  CLI.ID AS CLIENTE,
          CLI.NOME AS NOME,
          CID.DESCRICAO AS CIDADE,
          COUNT(PED.ID) AS QUANTIDADE_PEDIDOS,
          SUM(PIT.TOTAL_BRUTO) AS TOTAL_BRUTO,
          SUM(PIT.DESCONTO) AS TOTAL_DESCONTO,
          SUM(PED.VALOR_VENDA) AS TOTAL_LIQUIDO
  FROM tbPedidos PED
  INNER JOIN tbPedidosItens PIT ON PED.ID = PIT.FK_PEDIDO_ID
  INNER JOIN tbClientes CLI ON PED.FK_CLIENTE = CLI.CPF
  INNER JOIN tbProdutos PRO ON PIT.FK_PRODUTO_ID = PRO.ID
  INNER JOIN tbCep CEP ON CLI.FK_CEP = CEP.ID
  INNER JOIN tbBairros BAI ON CEP.FK_BAIRRO = BAI.ID
  INNER JOIN tbCidades CID ON BAI.FK_CIDADE = CID.ID
  WHERE CAST(PED.DATA_HORA AS DATE) BETWEEN '01-01-2020' AND '28-12-2020'
  GROUP BY PED.ID, CLI.ID, CLI.NOME, CID.DESCRICAO

```

8. Fazer pesquisa totalizando vendas realizadas por cidade, contendo os campos: Cidade, Bairro, Qtde Pedidos, Total Bruto, total Descontos e Total Liquido

``` sql
  SELECT  CID.DESCRICAO AS CIDADE,
          BAI.DESCRICAO AS BAIRRO,
          COUNT(PED.ID) AS QUANTIDADE_PEDIDOS,
          SUM(PIT.TOTAL_BRUTO) AS TOTAL_BRUTO,
          SUM(PIT.DESCONTO) AS TOTAL_DESCONTO,
          SUM(PED.VALOR_VENDA) AS TOTAL_LIQUIDO
  FROM tbPedidos PED
  INNER JOIN tbPedidosItens PIT ON PED.ID = PIT.FK_PEDIDO_ID
  INNER JOIN tbClientes CLI ON PED.FK_CLIENTE = CLI.CPF
  INNER JOIN tbProdutos PRO ON PIT.FK_PRODUTO_ID = PRO.ID
  INNER JOIN tbCep CEP ON CLI.FK_CEP = CEP.ID
  INNER JOIN tbBairros BAI ON CEP.FK_BAIRRO = BAI.ID
  INNER JOIN tbCidades CID ON BAI.FK_CIDADE = CID.ID
  GROUP BY CID.DESCRICAO, BAI.DESCRICAO
```

9. Fazer pesquisa para totalizar os pedidos por Produto, com campos: Produto, Grupo, Qtde Vendido, Total Bruto, Descontos, Total Liquido, Valor Medio vendas

``` sql
  SELECT  PRO.DESCRICAO AS PRODUTO,
          CAT.DESCRICAO AS GRUPO,
          COUNT(PED.ID) AS QUANTIDADE_PEDIDOS,
          SUM(PIT.TOTAL_BRUTO) AS TOTAL_BRUTO,
          SUM(PIT.DESCONTO) AS TOTAL_DESCONTO,
          SUM(PED.VALOR_VENDA) AS TOTAL_LIQUIDO
  FROM tbPedidos PED
  INNER JOIN tbPedidosItens PIT ON PED.ID = PIT.FK_PEDIDO_ID
  INNER JOIN tbClientes CLI ON PED.FK_CLIENTE = CLI.CPF
  INNER JOIN tbProdutos PRO ON PIT.FK_PRODUTO_ID = PRO.ID
  INNER JOIN tbCategoria CAT ON PRO.FK_CATEGORIA = CAT.ID
  INNER JOIN tbCep CEP ON CLI.FK_CEP = CEP.ID
  INNER JOIN tbBairros BAI ON CEP.FK_BAIRRO = BAI.ID
  INNER JOIN tbCidades CID ON BAI.FK_CIDADE = CID.ID
  GROUP BY PRO.DESCRICAO, CAT.DESCRICAO
```

10. Criar comando para excluir o cliente CPF = “0000001”

``` sql
  DELETE FROM tbClientes WHERE CPF= '00000000001'
```

11. Criar comando para excluir todas as vendas dos produtos Codigo = “0001”

``` sql
  DELETE FROM tbPedidosItens WHERE FK_PRODUTO_ID = 1
```