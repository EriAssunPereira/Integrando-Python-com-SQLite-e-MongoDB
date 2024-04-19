# Integrando-Python-com-SQLite-e-MongoDB

Vamos começar pela *Parte 1*, onde precisamos implementar um banco de dados relacional com SQLAlchemy. Aqui está um exemplo de como podemos definir o esquema usando classes SQLAlchemy:

No SQLAlchemy, as relações entre tabelas são definidas usando o conceito de **chaves estrangeiras** e **relacionamentos**. Vamos usar o exemplo das tabelas `Cliente` e `Conta` para entender melhor.

### Chaves Estrangeiras
Uma **chave estrangeira** é uma coluna ou combinação de colunas usadas para estabelecer e impor um link entre os dados em duas tabelas. No exemplo que forneci anteriormente, a tabela `Conta` tem uma coluna chamada `cliente_id`, que é uma chave estrangeira para a coluna `id` na tabela `Cliente`:

```python
cliente_id = Column(Integer, ForeignKey('clientes.id'))
```

Isso significa que cada `Conta` está associada a um `Cliente` através do `cliente_id`.

### Relacionamentos
O **relacionamento** é definido para expressar a conexão entre as duas tabelas. No SQLAlchemy, isso é feito usando a função `relationship()`. No exemplo, o relacionamento é definido em ambas as classes `Cliente` e `Conta`:

```python
class Cliente(Base):
    # ...
    contas = relationship('Conta', back_populates='cliente')

class Conta(Base):
    # ...
    cliente = relationship('Cliente', back_populates='contas')
```

Aqui, `Cliente.contas` é uma lista de todas as `Contas` associadas a esse `Cliente`, e `Conta.cliente` é uma referência ao `Cliente` associado a essa `Conta`. O parâmetro `back_populates` é usado para configurar a propriedade na outra classe que deve ser atualizada quando a relação é modificada.

### Benefícios dos Relacionamentos
Os relacionamentos permitem que você carregue e acesse dados relacionados de forma eficiente. Por exemplo, se você tem um objeto `Cliente`, pode facilmente acessar todas as suas `Contas` relacionadas sem precisar fazer uma nova consulta ao banco de dados:

```python
cliente = session.query(Cliente).filter_by(nome='João Silva').first()
print(cliente.contas)  # Isso irá imprimir todas as contas associadas ao cliente 'João Silva'
```

Da mesma forma, se você tem um objeto `Conta`, pode encontrar o `Cliente` associado:

```python
conta = session.query(Conta).filter_by(id=1).first()
print(conta.cliente)  # Isso irá imprimir o cliente associado à conta com id 1
```

Essas relações são fundamentais para trabalhar com bancos de dados relacionais, pois permitem que você modele a estrutura de dados de maneira que reflita as conexões do mundo real entre entidades.

Desafio parte 1:

from sqlalchemy import create_engine, Column, Integer, String, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship

Base = declarative_base()

class Cliente(Base):
    __tablename__ = 'clientes'
    id = Column(Integer, primary_key=True)
    nome = Column(String)
    contas = relationship('Conta', back_populates='cliente')

class Conta(Base):
    __tablename__ = 'contas'
    id = Column(Integer, primary_key=True)
    cliente_id = Column(Integer, ForeignKey('clientes.id'))
    saldo = Column(Integer)
    cliente = relationship('Cliente', back_populates='contas')

# Criar um engine que armazena os dados no diretório local
engine = create_engine('sqlite:///banco_de_dados.db')

# Criar todas as tabelas no banco de dados
Base.metadata.create_all(engine)

# Criar uma sessão para interagir com o banco de dados
Session = sessionmaker(bind=engine)
session = Session()

# Inserir dados de exemplo
cliente1 = Cliente(nome='João Silva')
conta1 = Conta(saldo=5000, cliente=cliente1)
session.add(cliente1)
session.add(conta1)
session.commit()

# Métodos de recuperação de dados
clientes = session.query(Cliente).all()
for cliente in clientes:
    print(cliente.nome, cliente.contas)

    Vamos falar sobre a **Parte 2** desse desafio, que envolve trabalhar com MongoDB e Pymongo.

### MongoDB
O MongoDB é um banco de dados NoSQL orientado a documentos. Isso significa que, em vez de armazenar dados em tabelas como em bancos de dados relacionais, ele armazena dados em estruturas de documentos semelhantes a JSON, chamadas BSON. Essa abordagem oferece flexibilidade para armazenar e combinar dados de formas complexas e hierárquicas.

### Pymongo
Pymongo é uma biblioteca Python que permite interagir com o MongoDB. Com Pymongo, podemos realizar operações como inserir, atualizar, deletar e consultar documentos no banco de dados MongoDB.

### Conexão com MongoDB Atlas
O MongoDB Atlas é um serviço de banco de dados em nuvem que hospeda bancos de dados MongoDB. Para conectar-se ao MongoDB Atlas usando Pymongo, precisaremos do URI de conexão fornecido pelo Atlas. Aqui está um exemplo de como podemos se conectar:

```python
from pymongo import MongoClient

# Substitua 'seu_uri_do_mongo_atlas' pelo URI fornecido pelo MongoDB Atlas
client = MongoClient('seu_uri_do_mongo_atlas')
```

### Criando Banco de Dados e Coleções
No MongoDB, um banco de dados contém coleções, que são equivalentes a tabelas em bancos de dados relacionais. Cada coleção contém documentos, que são equivalentes a registros ou linhas. Para criar um banco de dados e uma coleção:

```python
# Substitua 'nome_do_banco_de_dados' pelo nome que você deseja dar ao seu banco de dados
db = client['nome_do_banco_de_dados']

# Crie uma coleção chamada 'bank'
colecao = db['bank']
```

### Inserindo Documentos
Documentos são inseridos na coleção usando o método `insert_one()` ou `insert_many()` para inserir vários documentos. Um documento é um dicionário Python que representa os dados que você deseja armazenar.

```python
documento_cliente = {
    'nome': 'João Silva',
    'contas': [
        {'id': 1, 'saldo': 5000}
    ]
}

colecao.insert_one(documento_cliente)
```

### Recuperando Informações
Para recuperar informações, podemos usar métodos como `find()` ou `find_one()`. Esses métodos permitem que você especifique critérios de pesquisa usando pares de chave e valor.

```python
# Para encontrar todos os documentos onde o nome é 'João Silva'
clientes = colecao.find({'nome': 'João Silva'})

for cliente in clientes:
    print(cliente)
```

### Considerações Finais
Trabalhar com MongoDB e Pymongo é bastante direto e oferece muita flexibilidade. É importante planejar bem a estrutura dos seus documentos e entender como as operações de agregação funcionam para tirar o máximo proveito do MongoDB.

Desafio parte 2:

from pymongo import MongoClient

# Conectar ao MongoDB Atlas
client = MongoClient('seu_uri_do_mongo_atlas')

# Criar um banco de dados
db = client['nome_do_banco_de_dados']

# Definir uma coleção
colecao = db['bank']

# Inserir documentos
documento_cliente = {
    'nome': 'João Silva',
    'contas': [
        {'id': 1, 'saldo': 5000}
    ]
}
colecao.insert_one(documento_cliente)

# Recuperar informações
clientes = colecao.find({'nome': 'João Silva'})
for cliente in clientes:
    print(cliente)

Lembre-se de substituir 'seu_uri_do_mongo_atlas' pelo URI de conexão fornecido pelo MongoDB Atlas e 'nome_do_banco_de_dados' pelo nome que você deseja dar ao seu banco de dados.
