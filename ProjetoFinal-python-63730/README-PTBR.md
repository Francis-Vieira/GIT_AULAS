# Projeto Final: Construção pipeline de dados

Objetivo: Este projeto tem como objetivo a construção de um código que coleta dados de 3 APIs, relacionadas a dados abertos do IBGE, e após atividade de processamento e integração dos dados seja gerado um banco de dados consolidado que viabilize analisar informações de frequência de registro de Nomes de pessoas no país.

## 🚀 Público-alvo

O código deste projeto destina-se principalmente a analista de dados que tenham interesse em estudos relacionado a frequência de registro de Nomes de pessoas no país, mas podendo ser possível sua utilização por desenvolvedores a fim de consultas e/ou estudo.

### 📋 Saída

Ao final do código foi acrescentado instrução para exportação das tabelas do banco de dados em arquivos csv, com estrutura a seguir:

```
TB_Estados.csv, campos: [id,sigla,nome,regiao_id,regiao_sigla,regiao_nome]
```

```
TB_Rank_Nomes.csv, campos: [localidade_id,nome,frequencia,ranking]
```

```
TB_Nome_Decada.csv, campos: [nome,localidade,frequencia 1930[,"frequencia [1930,1940[","frequencia [1940,1950[","frequencia [1950,1960[","frequencia [1960,1970[","frequencia [1970,1980[","frequencia [1980,1990[","frequencia [1990,2000[","frequencia [2000,2010["]
```

### 🔩 Nível de Privacidade

As informações extraídas estão disponibilizadas em APIs de dados abertos diante da subjetividade e imparcialidade dos dados, e, por isso, não foram utilizadas soluções de segurança que visem garantir privacidade neste projeto.

## ⚙️ Pré-requisitos

*Ambientes*
Neste projeto foi utilizado o Visual Studio Code, versão 1.96.0 (user setup) e Python versão 3.13.1 em ambiente com Sistema Operacional Windows_NT x64 versão 10.0.19045.

*Bibliotecas*
As bibliotecas utilizadas e necessárias para execução do código são:
>notification, versão 0.2.1;
>plyer, versão 2.1.0;
>pandas, versão 2.2.3;
>requests, versão 2.32.3;
>missingno, versão 0.5.2; 

*Arquivos*
Para execução deste projeto é necessário o arquivo “requirements.txt” em mesmo diretório de execução do arquivo de código fonte.
E com execução de todo código serão disponibilizados os arquivos de saída mencionado no item 📋 Saída.


### 🔩 APIs utilizadas

i.	API: v1/localidades/estados
a.	URL: https://servicodados.ibge.gov.br/api/v1/localidades/estados
b.	Utilização: chamada da URL completa apenas, não há informação adicional necessária para execução.
c.	Descrição: API compõem acervo do serviço de dados abertos IBGE de localidades, onde conta um conjunto de APIs referente as divisões de mesorregiões, microrregiões e às divisões político-administrativas do Brasil.
```
Campos: "id", "sigla", "nome", "regiao" onde "regiao" conta lista com {"id", "sigla", "nome"} de cada um dos estados
```

ii.	API: api/v2/censos/nomes/ranking?localidade={id}
a.	URL: https://servicodados.ibge.gov.br/api/v2/censos/nomes/ranking?localidade={id}
b.	Utilização: onde está informado a variável {id} na URL de acesso deve ser alterado por um inteiro com indetify da localidade, informação obtida na API anterior “v1/localidades/estados”
c.	Descrição: API compõem acervo do serviço de dados abertos IBGE, utiliza como base o Censo Demográfico 2010 do IBGE, tem como propósito apresentar um ranking de Top 20 de Nomes por frequência da localidade informada pelo {id}.
```
Campos: "localidade", "sexo" e "res" onde "res" conta lista com {"nome","frequencia","ranking":1}, [...], {"nome","frequencia","ranking":20}
```

iii.	API: v2/censos/nomes/{nome}
a.	URL: https://servicodados.ibge.gov.br/api/v2/censos/nomes/{nome}
b.	Utilização: onde está informado a variável {nome} na URL de acesso deve ser alterado por uma string de nome, exemplo: MARIA
c.	Descrição: API compõem acervo do serviço de dados abertos IBGE, utiliza como base o Censo Demográfico 2010 do IBGE, tem como propósito apresentar a frequência no Nome no país ao longo das décadas, de 1930 a 2010.
```
Campos: "nome", "sexo", "localidade", "res" onde "res" consta lista com "periodo":"1930[", "periodo":"[1930,1940[", "periodo":"[1940,1950[", {"periodo":"[1950,1960[", {"periodo":"[1960,1970[", {"periodo":"[1970,1980[", {"periodo":"[1980,1990[", {"periodo":"[1990,2000[" e {"periodo":"[2000,2010["
```

### ⌨️ Funções criadas

i.	Nome: Alerta_default 
a.	Parâmetros: string com código retorno
b.	Retorno: apresentada mensagem de erro com notificação em tela da chamada da API devido status_code recebido ser diferente de 200.
c.	Descrição: função tem objetivo de gerar alerta para etapa de extração das informações de retorno da API.

ii.	Nome: buscar_nomes
a.	Parâmetros: string com informação de nome
b.	Retorno: faz chamadas para API api/v2/censos/nomes/ utilizando função drop_duplicates que desconsidera informações duplicadas obtidas.
c.	Descrição: função tem objetivo de obter os dados da API v2/censos/nomes/{nome} para cada ocorrência única de nome, garantido a remoção de duplicados.


## 🛠️ Tratamentos aplicados

*Erros*
Erro chamada API: Foram considerados possíveis erros no retorno de chamada das APIs e, para tanto, foi criada função de notificação de Alerta: Alerta_default.
*Limpeza de dados*
Utilizada biblioteca missingno para verificar se há ocorrência de campos com valores nulos e selecionado para essa finalidade, devido a praticidade de visualização. E em caso afirmativo, em que toda coluna está nula, realizada ação de exclusão.


## 📦 Banco de dados

Utilizando biblioteca sqlite3 foram criadas e schema de tabelas para gravação das informações em banco de dados.
Após ação, foram realizadas consultas em tabela do banco de dados para validar a efetividade da criação e analisar informações via consultas. 

```
Exemplo: 

#Consultando TB_Estado com "condição” para exibir os dados filtrados pela região "Norte"
# Conectando ao banco de dados
conn = sqlite3.connect('BD_IBGE_Nomes.db')
# consulta tabela 'TB_Estado'
consulta_TB_Estado = '''SELECT  id ID_Estado,
                                nome Estado,
                                sigla UF,
                                regiao_nome 'Região'
                        FROM TB_Estado
                        WHERE regiao_nome = 'Norte' '''
# Gravando informação em variável temporária
temp_Estado = pd.read_sql_query(consulta_TB_Estado, conn)
# Fechando a conexão com o banco de dados
conn.close()
#apresentando o DataFrame
temp_Estado

```

## 📌 Versionamento

Possível ser consultado com base no arquivo  [requirements.txt](https://github.com/Francis-Vieira/GIT_AULAS/blob/main/ProjetoFinal-python-63730/requirements.txt) lista de bibliotecas e suas versões utilizados neste projeto.

## ✒️ Referências


* **Documentação APIs** - - [api/docs/](https://servicodados.ibge.gov.br/api/docs/)
* **IBGE, Censo Demográfico 2010** - [censo2010.ibge](https://censo2010.ibge.gov.br/nomes/#/search)
* **Doc API localidades** - [api/docs/localidades](https://servicodados.ibge.gov.br/api/docs/localidades)
* **Doc API Nomes** - [api/docs/nomes](https://servicodados.ibge.gov.br/api/docs/nomes?versao=2)

---
⌨️ por [Francis Vieira](https://github.com/Francis-Vieira) 