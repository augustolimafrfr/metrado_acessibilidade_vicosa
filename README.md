# TUTORIAL PARA ORGANIZAÇÃO DA BASE DE DADOS - ACESSIBILIDADE E CONECTIVIDADE DAS VIAS DE VIÇOSA
Este é um tutorial com o passo-a-passo de todo o procedimento metodológico utilizado na dissertação de mestrado para a preparação da base de dados do município de Viçosa com o objetivo de calcular a acessibilidade de suas principais vias. Os softwares utilizados durante o processo foram: `PostgreeSQL 14` (com o pacote de extensões para dados espaciais PostGis) através do gerenciador `pgAdmin`, `Python 3` (juntamente com a IDE Jupyter Notebook), `QGIS 3.22.7` e `ArcGIS 10.5`.

O objetivo desse tutorial é explicar de forma detalhada os procedimentos realizados para organizar a base de dados das vias de Viçosa e com isso, será possível entender o processo e replicá-lo para demais bases de dados.

**Todos arquivos utilizados na dissertação estão localizados na pasta `DADOS_DISSERTAÇÃO`, que possui diversas sub-pastas com os arquivos de forma organizada. Essa pasta pode ser baixada através do link: https://drive.google.com/file/d/1mreBchikRm8Gn5EZgYaLn4PTq6l3dMr_/view?usp=sharing**

## PASSO-A-PASSO:

### 1. ORIGEM DOS DADOS:

Os dados utilizados nesse trabalho foram fornecidos por Vieira e Castro (2021) e pelo Serviço Autônomo de Água e Esgoto do município de Viçosa (SAAE). O primeiro arquivo nomeado como `Rede_Viaria_RSU_2021` trata-se do shapefile contando as ruas de Viçosa e conta com os atributos como **Nome da Rua**, **Tipo de Eixo**, **Largura da rua**, **Pavimento**, **Declividade**, **Bairro**, entre outros. Os dados do SAAE, nomeado como `declividade_pontos_cotados_SAAE`, consistem em pontos cotados dos meio-fios das ruas ao longo de Viçosa. A malha viária de Vieira e Castro (2021) será utilizada para formar a rede de análise e os pontos cotados para gerar o Modelo Digital de Elevação (MDE) do município.

**Os arquivos `Rede_Viaria_RSU_2021` e `declividade_pontos_cotados_SAAE` estão localizados na pasta `DADOS INICIAIS`.**


### 2. SELEÇÃO DOS ATRIBUTOS UTEIS:

Diverssos atributos presentes em `Rede_Viaria_RSU_2021` não serão úteis para esse trabalho e outros serão cálculados novamente. Portanto, da tabela de atributos dos dados originais, aqueles escolhidos foram: **largura da via**, **pavimento**, **sentido da via** e todos os demais atributos foram excluidos através da ferramenta `Editar campos` do software `QGIS`. Dois novos campos foram criados: **nome da rua** e **tipo_pm**, sendo que o primeiro será utilizado para incluir o nome das vias e o segundo a classificação de acordo com o plano de mobilidade.

### 3. SELEÇÃO DAS VIAS A SEREM UTILIZADAS:
As vias mencionadas no Plano de Mobilidade de Viçosa foram selecionadas utilizando o `Open Street Map` e o `Google Maps` como pano de fundo. Durante esse processo os campos os campos **nome da rua** e **tipo_pm** foram preenchidos.

### 4. GARATINDO A INTEGRIDADE TOPOLÓGICA DA REDE:

Para garantir a integridade topológica da rede foi utilizado o complemento `Verificador de topologia` do software `QGIS`. No botão `Configurações`, seleciona-se o arquivo da malha viária e marca-se a opção `não devem ter dangles`, adiciona-se a regra e confirma. Na janela aberta clica-se em `Validar tudo`. Surgirão pontos vermelhos em alguns locais da malha viária e será possível descobrir quais linhas estão desconectadas (esses pontos vermelhos devem existir apenas onde a via realmente está interrompida, como as extremidades).

### 5. OBTENÇÃO DA REDE EM FORMA DE GRAFOS:

A malha viária obtida até então não está na forma de grafos. Uma rede viária na forma de grafos deve ser dividida apenas na junção entre os vértices. Para gerar a malha viária de Viçosa dessa forma utilizou-se o software `ArcGIS`. Primeiramente utilizou-se a ferramenta `Dissolve`para agregar todos elementos em apenas uma feição e para concluir utilizou-se a ferramenta `Explode Multipart Feature` da caixa de ferramentas `Advanced Editing`. Os atributos presentes no arquivo anterior devem ser reciados e novos também foram criados. Por fim, os atributos dessa rede foram: **nome da rua**, **tipo_pm**, **comprimento da via**, **largura media**, **pavimentaçao**, **sentido da via** e **declividade media**. Com exceção do atributo **comprimento da via**, todos os demais estão em branco e foram prenchidos posteriormente.

**Esse arquivo foi nomeado como `vias_grafos` e está localizado na pasta `DADOS_INTERMEDIARIOS`.**

### 6. DIVIDINDO A MALHA VIÁRIA (GERADA NO PASSO 4) EM DIVERSAS MALHAS COM SEUS RESPECTIVOS ATRIBUTOS:

Antes de preencher os atributos do arquivo `vias_grafos`, realizou-se um procedimento com a malha obtida do `passo 4`. A partir dessa malha, gerou-se cinco novos arquivos utilizando a ferramenta Dissolve do software `ArcGIS`, com o objetivo de separar cada atributo da rede em um arquivo diferente.

Na ferramenta mencionada selecionou-se a malha viária (que não está na forma de grafos) com todos atributos e na aba `Dissolve_Fields (optional)` marcou-se um dos atributos desejados para a separação. Na aba `Output Feature Class` atribuiu-se o nome do novo arquivo (de forma que remeta ao atributo selecionado) e executou-se a ferramenta. Esse processo deve ser feito cinco vezes para gerar os arquivos para cada um dos cinco atributos (**nome da rua**, **tipo_pm**, **sentido da via**, **pavimento**, **largura**). Esses cinco novos dados foram manipulados pela ferramenta `Explodir linhas` do software `QGIS`, com o objetivo de obter os atributos de cada segmento de reta presente na malha viária. Os cinco arquivos foram nomeados como `vias_nomeruas` (com o nome de rua de cada segmento de reta), `vias_tipopm` (com o tipo de classificação de via de acordo com o Plnao de Mobilidade de Viçosa para cada segmento de reta), `vias_oneway` (com o sentido da via para cada segmento de reta), `vias_pavimentos` (com a pavimentação de cada segmento de reta), `vias_largura` (com a largura de cada segumento de reta).

**Os cinco arquivos se encontram na pasta `DADOS_INTERMEDIARIOS`**

### 7. GERAÇÃO DO MDE

O MDE de Viçosa foi gerado a partir do arquivo `declividade_pontos_cotados_SAAE`. Para isso, utilizou-se a ferramenta `Interpolação IDW` do software `QGIS`. O novo arquivo foi recortado através da ferramenta `Recortar raster pela extensão...` com o objetivo de reduzir a extensão do MDE de forma que englobe apenas a área de estudo. Apesar desse passo não ser obrigatório, é interessante recortar o arquivo raster com o objetivo de reduzir o seu tamanho e facilitar os processamentos. O arquivo que contem o MDE de Viçosa foi nomeado como `mde_vicosa`.

**O MDE do município se encontra na pasta `DADOS_INTERMEDIARIOS`**

### 8. IMPORTANDO TODOS OS DADOS PARA O BANCO DE DADOS

Todos arquivos gerados foram importados para o banco de dados. O procedimento para importar arquivos no formato vetorial (nesse estudo em formato shapefile) e raster são distintos.

#### 8.1. IMPORTANDO OS ARQUIVOS DO FORMATO SHAPEFILE

Os arquivos agora devem ser importados para um banco de dados. O banco usado nessa dissertação possui o nome `dissertacao` e possui todas extensões espaciais provenientes do PostGIS.

Os arquivos `vias_grafos`, `vias_nomeruas`, `vias_tipopm`, `vias_oneway`, `vias_pavimentos` e `vias_largura` foram importados através da ferramenta `Gerenciador BD...` disponível após a instalação do `PostGIS`. O procedimento é feito selecionando de forma individual o arquivo que será importado na aba **Entrada**, em **Esquema** seleciona-se a opção _public_ e em **Tabela** atribuiu-se o nome desejado da tabela (igual ao nome do arquivo em formato shapefile). Isso foi feito para todos os arquivos.

#### 8.2. IMPORTANDO OS ARQUIVOS DO FORMATO RASTER

O MDE foi importado através do **promp de comando**. Com o promp aberto digita-se os seguintes comandos:

**PARA TRANSFORMAR O ARQUIVO MATRICIAL EM UMA TABELA SQL BINÁRIA:**

    raster2pgsql -s 31983 -I -C -M mde_vicosa.tif -F -t 100x100 public.mde_vicosa > mde_vicosa.sql

**PARA IMPORTAR O ARQUIVO NO BANCO DE DADOS:**

    psql -h localhost -U postgres -p 5432 -d dissertacao -f mde_vicosa.sql

### 9. COMPLETANDO O ARQUIVO VIAS_GRAFOS

Os atributos do arquivo `vias_grafos` estão majoritariamente em branco. Para preenchê-los utilizou-se da lingaguem Python através da IDE `Jupyter Notebook`. O procedimento necessita da instalação, através do prompt de comando, do pacote `psycopg2` (necessário para manipular o banco de dados dentro da linaguem Python). Depois disso é possível executar os comandos em lingaguem Python.

**Todos os scripts explicados estão localizados na pasta `SCRIPTS_DISSERTACAO`. Essa pasta se divide em duas sub-pastas: SCRIPTS_SQL (com os codigos executados em linguagem SQL) e SCRIPTS_PYTHON (com o ambiente virtual utilizado nesse trabalho). Para acessar os Scripts que estão separados individualmente é necessário acessar a pasta projetos que está localizada em SCRIPTS_PYTHON >> av_dissertacao >> projetos.**

#### 9.1. INSTALAÇÃO DO PACOTE psycopg2

Para instalar o pacote psycopg2 é necessário digitar o seguinte comando no **prompt de comandos**:

    pip install psycopg2

#### 9.2. SCRIPTS EM PYTHON:

Os comandos descritos a seguir foram todos executados no **Jupyter Notebook**.

#### 9.2.1. IMPORTANDO O PACOTE psycopg2:

    import psycopg2 as pg

#### 9.2.2. CONECTANDO COM O BANCO DE DADOS:

Inseriu-se as configurações do banco de dados. Ele está sendo utilizado no servidor local (host = 'localhost'), o nome do banco de dados é dissertacao (database = 'dissertacao'), o nome de usuário é postgres (user='postgres') e a senha do banco de dados é admin (password = 'admin).

    con = pg.connect(host='localhost', 
                database='dissertacao',
                user='postgres', 
                password='admin')

    cur = con.cursor() #CRIANDO UMA INSTÂNCIA PARA EXECUTAR COMANDOS EM SQL

    # OBS: O servidor hospedado na máquina local será conectado no banco de dados nomeado DISSERTAÇÃO, que possui usuário postgres e senha admin.

#### 9.2.3. VENDO QUANTOS GRAFOS A MALHA POSSUI

É necessário descobrir quantas linhas a rede possui. Para isso executa-se os seguintes comandos:

    tabela_grafos = 'vias_grafos'#TABELA COM OS GRAFOS

    sql = f'select max(id) from {tabela_grafos}' #COMANDO EM SQL A SER EXECUTADO

    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO

    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

    id_max = dados_consultados[0][0] #ID MÁXIMO DA MALHA


#### 9.2.4. ATRIBUINDO NOME DAS VIAS PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará quais nomes de ruas presentes na tabela `vias_nomeruas` estão contidas na linha com id em análise. Esses nomes (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'nome_rua' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_nomeruas' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vnr' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'nome_rua' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO O NOME DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_dados_str = lista_dados_str + dado + '; '
            else:
                lista_dados_str = lista_dados_str + dado

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 9.2.5. ATRIBUINDO O TIPO DA VIA DE ACORDO COM O PLANO DE MOBILIDADE PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará quais classificações de acordo com o Plano de Mobilidade de Viçosa presentes na tabela `vias_tipopm` estão contidas na linha com id em análise. Essas classificações (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'largura_me' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_largura' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vl' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'largura' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO A LARGURA DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados}, st_length({abr_dados}.geom) from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        numerador = 0 #CRIANDO UMA LISTA VAZIA, NO QUAL SERÁ SOMADO TODOS OS ATRIBUTOS PONDERADOS PELO TAMANHO DE SUA LINHA
        denominador = 0 #SOMA DOS PESOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NAS LISTAS CRIADAS
            numerador = numerador + dado[0]*dado[1]
            denominador = denominador + dado [1]

        largura_me_i = float(f'{numerador/denominador:.2f}') #CALCULANDO A LARGURA MEDIA DO GRAFO i

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

                sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{largura_me_i}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
                cur.execute(sql) #EXECUTANDO O COMANDO
                con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

                print(f'ID: {id_i} {largura_me_i} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 9.2.6. ATRIBUINDO O TIPO DE PAVIMENTO PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará quais tipos de pavimento presentes na tabela `vias_pavimentos` estão contidas na linha com id em análise. Esses pavimentos (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'pavimento' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_pavimentos' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vp' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'pavimento' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO O TIPO DE PAVIMENTO DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_dados_str = lista_dados_str + dado + '; '
            else:
                lista_dados_str = lista_dados_str + dado

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 9.2.7. ATRIBUINDO O SENTIDO DA VIA PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará qual sentido da via presente na tabela `vias_oneway` estão contidas na linha com id em análise. Esse sentido de via (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

OBS.: Caso a linha possua mais de um sentido de via significa que existe um erro na geração da rede viária e isso deve ser consertado manualmente.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'oneway' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_oneway' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vow' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'oneway' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO SENTIDO DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_dados_str = lista_dados_str + dado + '; '
            else:
                lista_dados_str = lista_dados_str + dado

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.


#### 9.2.8. ATRIBUINDO A LARGURA DA VIA PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará qual a largura média e comprimento do segmento de rede presente na tabela `vias_largura` estão contidas na linha com id em análise. Será calculado uma média ponderada dessas larguras (sendo o comprimento do segmento de reta o ponderador). Essa largura média de via será atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'largura_me' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_largura' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vl' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'largura' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO A LARGURA DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados}, st_length({abr_dados}.geom) from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        numerador = 0 #CRIANDO UMA LISTA VAZIA, NO QUAL SERÁ SOMADO TODOS OS ATRIBUTOS PONDERADOS PELO TAMANHO DE SUA LINHA
        denominador = 0 #SOMA DOS PESOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NAS LISTAS CRIADAS
            numerador = numerador + dado[0]*dado[1]
            denominador = denominador + dado [1]

        largura_me_i = float(f'{numerador/denominador:.2f}') #CALCULANDO A LARGURA MEDIA DO GRAFO i

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{largura_me_i}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {largura_me_i} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.


#### 9.2.9. ATRIBUINDO A INCLINAÇÃO MÉDIA PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição gerará pontos igualmente espaçados ao longo da linha com id em análise e será calculado a declividade média dos segmentos de reta formado por esses pontos. A quantidade de segmentos de reta gerados dependerá do comprimento da linha (linhas com comprimentos menores que 300 metros serão divididas em 4 partes, linhas com comprimento entre 300 metros e 600 metros serão divididos em 10 partes, linhas com comprimento acima de 600 serão divididas em 20 partes). A declividade média. A declividade média da linha será atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'decliv_me' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'mde_vicosa' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'mde' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA


    #ADICIONANDO A DECLIVIDADE DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        #COMPRIMENTO DO GRAFO

        sql = f'SELECT ST_Length(geom) FROM {tabela_grafos} WHERE id = {id_i}' #COMANDO EM SQL PARA CONSULTAR O COMPRIMENTO DO GRAFO

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        comp_grafo = dados_consultados[0][0] #COMPRIMENTO DO GRAFO

        #SELECIONANDO AS ALTITUDES AO LONGO DO GRAFO DE ACORDO COM O COMPRIMENTO DO GRAFO:

        if comp_grafo <= 300: #CASO O COMPRIMENTO DO GRAFO SEJA <250m, O GRAFO SERÁ DIVIDIDO EM 4 PARTES:

            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

            for per in range(0, 101, 25):            

                sql = f"SELECT ST_Value({abr_dados}.rast, pts.geom) as elevacao FROM (SELECT ST_LineInterpolatePoint(ST_LineMerge(geom), {per/100}) as geom FROM {tabela_grafos} WHERE id = {id_i}) as pts, {tabela_dados} {abr_dados} WHERE ST_Intersects({abr_dados}.rast, pts.geom);" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS COTAS DOS PONTOS AO LONGO DO GRAFO PARA O ID DO GRAFO EM ANÁLISE.

                cur.execute(sql) #EXECUTANDO O COMANDO

                dados_consultados = cur.fetchall() #RETORNANDO OS DADOS            

                lista_dados.append(dados_consultados[0][0])    

            #CALCULANDO A DECLIVIDADE MEDIA DO GRAFO
            decliv = 0
            for i in range(0, len(lista_dados)-1):
                decliv = decliv + (lista_dados[i+1] - lista_dados[i])/(comp_grafo/4)

            decliv = round(decliv/4*100, 2)

        elif comp_grafo <= 600: #SE O COMPRIMENTO DO GRAFO FOR <600m ENTÃO O GRAFO SERÁ DIVIDIDO EM 10 PARTES:

            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

            for per in range(0, 101, 10):            

                sql = f"SELECT ST_Value({abr_dados}.rast, pts.geom) as elevacao FROM (SELECT ST_LineInterpolatePoint(ST_LineMerge(geom), {per/100}) as geom FROM {tabela_grafos} WHERE id = {id_i}) as pts, {tabela_dados} {abr_dados} WHERE ST_Intersects({abr_dados}.rast, pts.geom);" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS COTAS DOS PONTOS AO LONGO DO GRAFO PARA O ID DO GRAFO EM ANÁLISE.

                cur.execute(sql) #EXECUTANDO O COMANDO

                dados_consultados = cur.fetchall() #RETORNANDO OS DADOS            

                lista_dados.append(dados_consultados[0][0])    

            #CALCULANDO A DECLIVIDADE MEDIA DO GRAFO
            decliv = 0
            for i in range(0, len(lista_dados)-1):
                decliv = decliv + (lista_dados[i+1] - lista_dados[i])/(comp_grafo/10)

            decliv = round(decliv/10*100, 2)

        else:     #SE O COMPRIMENTO DO GRAFO FOR >600m ENTÃO O GRAFO SERÁ DIVIDIDO EM 20 PARTES:

            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

            for per in range(0, 101, 5):            

                sql = f"SELECT ST_Value({abr_dados}.rast, pts.geom) as elevacao FROM (SELECT ST_LineInterpolatePoint(ST_LineMerge(geom), {per/100}) as geom FROM {tabela_grafos} WHERE id = {id_i}) as pts, {tabela_dados} {abr_dados} WHERE ST_Intersects({abr_dados}.rast, pts.geom);" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS COTAS DOS PONTOS AO LONGO DO GRAFO PARA O ID DO GRAFO EM ANÁLISE.

                cur.execute(sql) #EXECUTANDO O COMANDO

                dados_consultados = cur.fetchall() #RETORNANDO OS DADOS            

                lista_dados.append(dados_consultados[0][0])    


            #CALCULANDO A DECLIVIDADE MEDIA DO GRAFO
            decliv = 0
            for i in range(0, len(lista_dados)-1):
                decliv = decliv + (lista_dados[i+1] - lista_dados[i])/(comp_grafo/20)

            decliv = round(decliv/20*100, 2)

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{decliv}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {decliv} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 9.2.10. ENCERRANDO A CONEXÃO COM O BANCO DE DADOS

Após a conclusão dos processos acima, a tabela `via_grafos` foi preenchida e a conexão com o banco de dados pode ser encerrada com os seguintes comandos:

    cur.close() #ENCERRANDO A INSTÂNCIA CRIADA PARA A EXECUÇÃO DO COMANDO
    con.close() #ENCERRANDO A CONEXÃO COM O BANCO DE DADOS

**OBS.: Esses códigos estão disponíveis em SCRIPTS_DISSERTACAO > SCRIPTS_PYTHON > av_dissertacao > projetos > 1_ORGANIZAR_DADOS > VIÇOSA > ORGANIZAR_DADOS_DAS_VIAS_SEM_O_ANEL.ipyb**

**O arquivo `vias_grafos` preenchido se encontra na pasta `DADOS_FINAIS`.**

### 10. PREPARAÇÃO DA REDE COM ANÉL VIÁRIO

Para criar a rede com o anél viário proposto por Silva (2012) utilizou-se do mapa com seu traçado disponibilizado em sua tese. O mapa foi georreferenciado através da ferramenta `Georreferenciador` do software `QGIS` e serviu como pano de fundo para selecionar da base disponibilizada por Vieira e Castro (2021) aquelas vias que compunham o anel. A união do anel viário com a malha atual de Viçosa gerou uma malha viária fictícia com a simulação do Anel Viário. Os `passos 4 e 5 ` desse tutorial foram realizados para essa nova malha com o objetivo de garantir a integridade topológica da rede e transformá-la na forma de grafos. Com isso, gerou-se uma rede nomeada como `vias_grafos_anel` com os mesmo atributos da rede `via_grafos` mais um adicional: id_grafo_semanel, que possui o objetivo indicar qual o id correspondente da tabela `via_grafos` na tabela `via_grafos_anel`. Portanto, os atributos dessa nova rede são: **nome da rua**, **tipo_pm**, **comprimento da via**, **largura media**, **pavimentaçao**, **sentido da via**, **declividade media** e **id_grafo_semanel**.

Esse dado foi importado para o banco de dados através da ferramenta `Gerenciador DB...`.

**O arquivo `vias_grafos_anel` se encontra na pasta `DADOS_INTERMEDIARIOS`.**

### 11. COMPLETANDO O ARQUIVO VIA_GRAFOS_ANEL

Para completar a rede `via_grafos_anel` realizou-se os mesmos processos da rede anterior. Algumas informações referentes ao anél viário foram preenchidas de forma manual através do software `QGIS` após a execução dos códigos, uma vez que o esse elemento da rede tem características específicas, tais como largura, classificação e pavimentação.

Os códigos utilizados são semelhantes.

#### 11.1. SCRIPTS EM PYTHON:

#### 11.1.1. IMPORTANDO O PACOTE psycopg2 QUE CONECTA O PYTHON COM O POSTGRE-SQL

    import psycopg2 as pg
    
#### 11.1.2. CONECTANDO AO BANCO DE DADOS

    con = pg.connect(host='localhost', 
                    database='dissertacao',
                    user='postgres', 
                    password='admin')

    cur = con.cursor() #CRIANDO UMA INSTÂNCIA PARA EXECUTAR COMANDOS EM SQL

    # OBS: O servidor hospedado na máquina local será conectado no banco de dados nomeado DISSERTAÇÃO, que possui usuário postgres e senha admin.

#### 11.1.3. VENDO QUANTOS GRAFOS A MALHA POSSUI

    tabela_grafos = 'vias_grafos_anel' #TABELA COM OS GRAFOS
    sql = f'select max(id) from {tabela_grafos}' #COMANDO EM SQL A SER EXECUTADO
    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    id_max = dados_consultados[0][0] #ID MÁXIMO DA MALHA

#### 11.1.4. PREENCHENDO A COLUNA ID_GRAFO_SEMANEL DA MALHA VIAS_GRAFOS_ANEL

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos_anel = 'vias_grafos_anel' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA COM O ANEL
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos_a = 'vga' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA COM O ANEL
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA

    #CONSULTANDO OS DADOS DE CADA GRAFO DO MALHA COM ANEL VIARIO:

    for id_i in range(1, id_max + 1):

        sql = f"SELECT {abr_grafos}.id FROM {tabela_grafos} {abr_grafos}, {tabela_grafos_anel} {abr_grafos_a} WHERE ST_Contains({abr_grafos}.geom, {abr_grafos_a}.geom) and {abr_grafos_a}.id = {id_i}" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ID DA TABELA DOS DOS GRAFOS (QUE JÁ ESTÁ COMPLETA).

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        #INSERINDO AS INFORMAÇÕES CONSULTADAS:

        if dados_consultados == []: #CONFERINDO SE A CONSULTA É VAZIA. CASO SEJA, PULA A ITERAÇÃO.
            print(f'ID grafo anel: {id_i} -> Passa!\n')
            continue        

        else:
            sql = f"UPDATE {tabela_grafos_anel} SET id_grafo_semanel = '{dados_consultados[0][0]}' WHERE id = {id_i}" #COMANDO EM SQL A SER EXECUTADO. SERÁ ADICIONADO OS ATRIBUTOS CONSULTADOS NA MALHA COM ANEL VIARIO.

            cur.execute(sql) #EXECUTANDO O COMANDO

            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

            print(f'ID grafo anel: {id_i} -> ID grafo sem anel: {dados_consultados[0][0]}\n')


#### 11.1.5. ATRIBUINDO NOME DAS VIAS PARA O BANCO DE DADOS DOS GRAFOS

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos_anel' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'nome_rua' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_nomeruas' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vnr' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'nome_rua' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO O NOME DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_dados_str = lista_dados_str + dado + '; '
            else:
                lista_dados_str = lista_dados_str + dado

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 11.1.6. ATRIBUINDO O TIPO DA VIA DE ACORDO COM O PLANO DE MOBILIDADE PARA O BANCO DE DADOS DOS GRAFOS

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos_anel' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'tipo_pm' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_tipopm' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vpm' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'tipo_pm' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO O TIPO DA VIA DE ACORDO COM O PLANO DE MOBILIDADE DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_dados_str = lista_dados_str + dado + '; '
            else:
                lista_dados_str = lista_dados_str + dado

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 11.1.7. ATRIBUINDO O SENTIDO DA VIA PARA O BANCO DE DADOS DOS GRAFOS

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos_anel' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'oneway' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_oneway' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vow' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'oneway' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO SENTIDO DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_dados_str = lista_dados_str + dado + '; '
            else:
                lista_dados_str = lista_dados_str + dado

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.


#### 11.1.8. ATRIBUINDO A LARGURA DA VIA PARA O BANCO DE DADOS DOS GRAFOS

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos_anel' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'largura_me' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_largura' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vl' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'largura' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO A LARGURA DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados}, st_length({abr_dados}.geom) from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        if dados_consultados == []: #CONFERINDO SE INFORMAÇÕES CONSULTADAS. SE NÃO EXISTIR, PULA A ITERAÇÃO.
            print(f'{id_i}: Pula!')
            continue

        else:
            numerador = 0 #CRIANDO UMA LISTA VAZIA, NO QUAL SERÁ SOMADO TODOS OS ATRIBUTOS PONDERADOS PELO TAMANHO DE SUA LINHA
            denominador = 0 #SOMA DOS PESOS

            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NAS LISTAS CRIADAS
                numerador = numerador + dado[0]*dado[1]
                denominador = denominador + dado [1]

            largura_me_i = float(f'{numerador/denominador:.2f}') #CALCULANDO A LARGURA MEDIA DO GRAFO i

            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

            sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{largura_me_i}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

            print(f'ID: {id_i} {largura_me_i} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.


#### 11.1.9. ATRIBUINDO A INCLINAÇÃO MÉDIA PARA O BANCO DE DADOS DOS GRAFOS

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos_anel' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'decliv_me' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'mde_vicosa' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'mde' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO A DECLIVIDADE DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA
    # OBS: O GRAFO DE ID = 1 SERÁ PULADO, UMA VEZ QUE O MDE
    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        if id_i == 7: #O GRAFO COM ID 7 NÃO POSSUI INFORMAÇÕES COMPLETAS DE MDE, PORTANTO SERÁ PULADO. PARA OS DEMAIS GRAFOS, SEGUE A ROTINA NORMALMENTE.

            print('Pulando o grafo de id 7.')
            continue

        else:
            #COMPRIMENTO DO GRAFO

            sql = f'SELECT ST_Length(geom) FROM {tabela_grafos} WHERE id = {id_i}' #COMANDO EM SQL PARA CONSULTAR O COMPRIMENTO DO GRAFO

            cur.execute(sql) #EXECUTANDO O COMANDO

            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

            comp_grafo = dados_consultados[0][0] #COMPRIMENTO DO GRAFO

            #SELECIONANDO AS ALTITUDES AO LONGO DO GRAFO DE ACORDO COM O COMPRIMENTO DO GRAFO:

            if comp_grafo <= 300: #CASO O COMPRIMENTO DO GRAFO SEJA <300m, O GRAFO SERÁ DIVIDIDO EM 4 PARTES:

                lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

                for per in range(0, 101, 25):            

                    sql = f"SELECT ST_Value({abr_dados}.rast, pts.geom) as elevacao FROM (SELECT ST_LineInterpolatePoint(ST_LineMerge(geom), {per/100}) as geom FROM {tabela_grafos} WHERE id = {id_i}) as pts, {tabela_dados} {abr_dados} WHERE ST_Intersects({abr_dados}.rast, pts.geom);" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS COTAS DOS PONTOS AO LONGO DO GRAFO PARA O ID DO GRAFO EM ANÁLISE.

                    cur.execute(sql) #EXECUTANDO O COMANDO

                    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS            

                    lista_dados.append(dados_consultados[0][0])    

                #CALCULANDO A DECLIVIDADE MEDIA DO GRAFO
                decliv = 0
                for i in range(0, len(lista_dados)-1):
                    decliv = decliv + (lista_dados[i+1] - lista_dados[i])/(comp_grafo/4)

                decliv = round(decliv/4*100, 2)

            elif comp_grafo <= 600: #SE O COMPRIMENTO DO GRAFO FOR <600m ENTÃO O GRAFO SERÁ DIVIDIDO EM 10 PARTES:

                lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

                for per in range(0, 101, 10):            

                    sql = f"SELECT ST_Value({abr_dados}.rast, pts.geom) as elevacao FROM (SELECT ST_LineInterpolatePoint(ST_LineMerge(geom), {per/100}) as geom FROM {tabela_grafos} WHERE id = {id_i}) as pts, {tabela_dados} {abr_dados} WHERE ST_Intersects({abr_dados}.rast, pts.geom);" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS COTAS DOS PONTOS AO LONGO DO GRAFO PARA O ID DO GRAFO EM ANÁLISE.

                    cur.execute(sql) #EXECUTANDO O COMANDO

                    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS            

                    lista_dados.append(dados_consultados[0][0])    

                #CALCULANDO A DECLIVIDADE MEDIA DO GRAFO
                decliv = 0
                for i in range(0, len(lista_dados)-1):
                    decliv = decliv + (lista_dados[i+1] - lista_dados[i])/(comp_grafo/10)

                decliv = round(decliv/10*100, 2)

            else:        

                lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

                for per in range(0, 101, 5):            

                    sql = f"SELECT ST_Value({abr_dados}.rast, pts.geom) as elevacao FROM (SELECT ST_LineInterpolatePoint(ST_LineMerge(geom), {per/100}) as geom FROM {tabela_grafos} WHERE id = {id_i}) as pts, {tabela_dados} {abr_dados} WHERE ST_Intersects({abr_dados}.rast, pts.geom);" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS COTAS DOS PONTOS AO LONGO DO GRAFO PARA O ID DO GRAFO EM ANÁLISE.

                    cur.execute(sql) #EXECUTANDO O COMANDO

                    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS            

                    lista_dados.append(dados_consultados[0][0])    

                #CALCULANDO A DECLIVIDADE MEDIA DO GRAFO
                decliv = 0
                for i in range(0, len(lista_dados)-1):
                    decliv = decliv + (lista_dados[i+1] - lista_dados[i])/(comp_grafo/20)

                decliv = round(decliv/20*100, 2)

            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

            sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{decliv}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

            print(f'ID: {id_i} {decliv} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.
        
#### 11.1.10. ENCERRANDO A CONEXÃO COM O BANCO DE DADOS

    cur.close() #ENCERRANDO A INSTÂNCIA CRIADA PARA A EXECUÇÃO DO COMANDO
    con.close() #ENCERRANDO A CONEXÃO COM O BANCO DE DADOS
    

**OBS. 1.: Esses códigos estão disponíveis em SCRIPTS_DISSERTACAO > SCRIPTS_PYTHON > av_dissertacao > projetos > 1_ORGANIZAR_DADOS > VIÇOSA > ORGANIZAR_DADOS_DAS_VIAS_COM_O_ANEL.ipyb**

**OBS. 2.: A largura do anél viário foi adicionada manualmente (largura = 16m, conforme propõe Silva (2012)). A pavimentação também foi colocada de forma manual (pavimentação = asfalto). A atributo `tipo_pm` foi preenchido manualmente (tipo_pm = anel_viario). A declividade da parte mais ao sul do anél foi calculada de forma manual, pois nesse trecho não há informações sufientes de MDE, portanto a declividade foi calculada com as informações disponíveis.**

**O arquivo `vias_grafos_anel` preenchido se encontra na pasta `DADOS_FINAIS.`**

### 12. CÁLCULO DA CONECTIVIDADE E ACESSIBILIDADE (REDE SEM ANEL VIÁRIO)

Essa parte foi dividida em duas etapas: pré-processamento e cálculos. Foi utilizados comandos em linguaguem `SQL` e linguaguem `Python` nessas etapas. Nos título dos tópicos terá informações de os comandos foram executados no pgAdmin (comando SQL) ou no Jupyter Notebook (comando Python).

#### 12.1. PRÉ-PROCESSAMENTO

#### 12.1.1. CRIAÇÃO DA REDE A SER ANALISADA `(SQL)`

Uma cópia da tabela `via_grafos` foi criada e nomeada como `rede_vicosa` com o seguinte comando:

    -- CRIANDO UMA NOVA TABELA SIMILAR AO 'vias_grafos' CHAMADA 'rede_vicosa':
    SELECT * INTO rede_vicosa FROM vias_grafos ORDER BY id;
    
#### 12.1.2. CRIAÇÃO DE NOVAS COLUNAS PARA A TABELA `(SQL)`

Para resolver problemas de rede com a extensão `pgRouting` é necessário a criação das colunas de nó inicial, no final, custo e custo reverso. Utiliza-se o seguinte comando:

    -- CRIANDO OS CAMPOS PARA OS VÉRTICES DE FIM E INICIO DO GRAFO:
    ALTER TABLE rede_vicosa
    ADD source INT4,
    ADD target INT4,
    ADD cost REAL,
    ADD reverse_cost REAL;
    
#### 12.1.3. RENOMEANDO O CAMPO 'geom' PARA 'the_geom' `(SQL)`

    ALTER TABLE rede_vicosa
    RENAME COLUMN geom TO the_geom;
    
#### 12.1.4. REDEFININDO OS DADOS DE DIREÇÃO DA VIA PARA 'YES' OU 'NO' `(SQL)`

    -- VIAS COM MÃO ÚNICA:
    UPDATE rede_vicosa SET oneway = 'YES' WHERE oneway = 'TF';

    -- VIAS COM MÃO DUPLA:
    UPDATE rede_vicosa SET oneway = 'NO' WHERE oneway = 'B';
    
#### 12.1.5. DEFININDO OS CUSTOS DOS ARCOS `(SQL)`

    -- DEFININDO CUSTOS 1 PARA TODOS OS VÉRTICES:
    UPDATE rede_vicosa SET cost = 1, reverse_cost = 1;

    -- DEFININDO O CUSTO REVERSOS = -1 PARA OS GRAFOS COM DIREÇÃO ÚNICA:
    UPDATE rede_vicosa SET reverse_cost = '-1' WHERE oneway = 'YES';

#### 12.1.6. CRIANDO A TOPOLOGIA DA REDE `(SQL)`

    SELECT pgr_createTopology('rede_vicosa', 1);
    
As colunas source e target serão preenchidas com id's de vérticies iniciais e finais. Uma nova tabela surgirá com a geometria dos vértices gerados.

#### 12.1.7. CRIANDO VÉRTICES NO MEIO DAS LINHAS `(SQL)`

Para o cálculo da acessibilidade é necessário computar o custo entre arcos, porém, a extensão só permite o cálculo entre nós. Para contornar esse problema, criou-se nós no meio de cada linha. O custo para atravessar esses nós centrais (com o custo de cada metade de arco igual a 0,5) é igual o custo entre arcos.

A tabela com pontos intermediários chama-se `mid_points` e para criá-la usou o seguinte comando:

    -- CRIANDO OS VÉRTICES NO MEIO DE CADA GRAFO:
    CREATE TABLE mid_points AS
    SELECT id, ST_LineInterpolatePoint(ST_LineMerge(the_geom), 0.5) as geom
    FROM rede_vicosa;

    ALTER TABLE mid_points
    ADD CONSTRAINT mid_points_pk PRIMARY KEY (id);

    CREATE INDEX sidx_mid_points
     ON mid_points
     USING GIST (geom);
     
#### 12.1.8. VERIFICAR SE AS COLUNAS SOURCE E TARGET ESTÃO CORRETAS `(SQL)`

Foi necessário verificar no `QGIS` se as colunas source e target estão preenchidas corretamente com os valores de id's dos vértices para as linhas com sentido de via unidirecional. Foi possível constatar que as linhas com id 1, 62, 71 e 96 estavam com o sentido invertido e isso foi consertado com os seguintes comandos:

    -- id do grafo: 1
    UPDATE rede_vicosa
    SET source = '47'
    WHERE id = 1;

    UPDATE rede_vicosa
    SET target = '46'
    WHERE id = 1;

    -- id do grafo: 62
    UPDATE rede_vicosa
    SET source = '80'
    WHERE id = 62;

    UPDATE rede_vicosa
    SET target = '23'
    WHERE id = 62;

    -- id do grafo: 71
    UPDATE rede_vicosa
    SET source = '25'
    WHERE id = 71;

    UPDATE rede_vicosa
    SET target = '24'
    WHERE id = 71;

    -- id do grafo: 96
    UPDATE rede_vicosa
    SET source = '43'
    WHERE id = 96;

    UPDATE rede_vicosa
    SET target = '42'
    WHERE id = 96;
    
 #### 12.1.9. CRIANDO A REDE COM PONTOS INTERMEDIÁRIOS ATRAVÉS DO QGIS
 
 Para criar a rede com pontos intermediários é necessário usar o software `QGIS` através da ferramenta `Connect nodes to lines` presente no complemento `Networks`. Criou-se uma cópia da camada `rede_vicosa` renomeando-a para `rede_vicosa_mp_prov` e a ferramenta foi utilizada nessa nova camada e o resultado foi importado para dentro do banco de dados utilizando o Gerenciador BD do `QGIS`. Durante a importação o id dessa rede foi nomeado como id0.
 
 #### 12.1.10. REMOVENTO ELEMENTOS INUTEIS DA `rede_vicosa_mp_prov` `(SQL)`
 
 A criação dessa nova rede não resultou em um arquivo pronto para ser trabalhado e novos processamento foram realizados para "limpar" a tabela. A tabela limpa foi nomeada como `rede_vicosa_mp` e os comandos foram os seguintes:
 
     CREATE TABLE rede_vicosa_mp AS
    SELECT rvmpp.id, rvmpp.comp_via, rvmpp.nome_rua, rvmpp.tipo_pm, rvmpp.largura_me, rvmpp.pavimento, rvmpp.oneway, rvmpp.decliv_me, rvmpp.source, rvmpp.target, rvmpp.cost, rvmpp.reverse_co, ST_Difference(rvmpp.geom, ptos.geom) as geom 
    FROM (SELECT rvmpp.*
    FROM rede_vicosa_mp_prov rvmpp, mid_points mp
    WHERE ST_Contains(mp.geom, rvmpp.geom)) as ptos, rede_vicosa_mp_prov rvmpp
    WHERE ST_Intersects(rvmpp.geom, ptos.geom);

    DELETE FROM rede_vicosa_mp
    WHERE ST_LENGTH(geom) = 0;

    ALTER TABLE rede_vicosa_mp
    ADD COLUMN id0 SERIAL PRIMARY KEY;
    
     --DELETANDO A TABELA rede_vicosa_mp_prov, JÁ QUE A DEFINITIVA FOI CRIADA
    DROP TABLE rede_vicosa_mp_prov;

    -- ALTERANDO A TABELA REDE_VICOSA_MP:
    ALTER TABLE rede_vicosa_mp
    RENAME COLUMN id TO id_grafo;
    
#### 12.1.11. RENOMEANDO O id DA TABELA `mid_points` `(SQL)`

A tabela mid_points apresenta o id de cada ponto igual ao id de sua respectiva linha da tabela `rede_vicosa`. Para diferenciar os pontos intermediários dos pontos finais e iniciais de cada linha original da `rede_vicosa`, escolheu-se somar 1000 ao valor do id do ponto intermediário. Dessa forma fica fácil de saber que o ponto de id 1050 da tabela `rede_vicosa_mp` corresponde ao ponto intermediário da linha 50, enquanto o ponto com id 50 é algum vértice inicial e/ou final de alguma linha. O comando utilizado foi:

    --ALTERANDO A TABELA 'mid_points' PARA INCLUIR UMA COLUNA CHAMADA ID_GRAFO E ALTERAR O ID PARA 1000 + id DO GRAFO.
    ALTER TABLE mid_points
    ADD id_grafo INT4;

    UPDATE mid_points
    SET id_grafo = id;

    UPDATE mid_points
    SET id = 1000 + id;

    SELECT * FROM mid_points;
    
#### 12.1.12. CRIANDO A TABELA `rede_vicosa_mp_vertices` COM TODOS OS VÉRTICES DA `rede_vicosa_mp` (VÉRTICES DE INÍCIO E FIM DA `rede_vicosa` E VÉRTICES DA TABELA `mid_points` `(SQL)`
   
       --CRIANDO A TABELA rede_vicosa_mp_vertices QUE POSSUIRÁ TODOS OS VERTICES DA REDE PARA CALCULO DE ACESSIBILIDADE:
    SELECT *
    INTO rede_vicosa_mp_vertices
    FROM mid_points;

    ALTER TABLE rede_vicosa_mp_vertices
    ADD CONSTRAINT rede_vicosa_mp_vertices_pk PRIMARY KEY (id);

    INSERT INTO rede_vicosa_mp_vertices (id, geom)
    (SELECT id, the_geom as geom FROM rede_vicosa_vertices_pgr);

    SELECT * FROM rede_vicosa_mp_vertices;

#### 12.1.13. COMPLETANDO A TABELA `rede_vicosa_mp` COM OS VÉRTICES INICIAIS E FINAIS DE CADA LINHA `(PYTHON)`

Para completar as colunas source e target da tabela `rede_vicosa_mp` utilizou-se um código em linguagem Python. O código realiza uma estrutura de repetição que percorre da linha 1 até a linha de id máximo da tabela e analisa qual elemento da tabela `rede_vicosa_mp_vertices` está contido no inicio e fim da linha em análise. Em seguida atualiza as colunas source e target com os valores encontrados. Segue o código:

#### 12.1.13.1. IMPORTANDO O PACOTE psycopg2 QUE CONECTA O PYTHON COM O POSTGRE-SQL E O PACOTE pandas PARA ORGANIZAR AS TABELAS DE ACESSIBILIDADE `(PYTHON)`

    import psycopg2 as pg
    
#### 12.1.13.2. CONECTANDO AO BANCO DE DADOS `(PYTHON)`

    con = pg.connect(host='localhost', 
                    database='dissertacao',
                    user='postgres', 
                    password='admin')

    cur = con.cursor() #CRIANDO UMA INSTÂNCIA PARA EXECUTAR COMANDOS EM SQL

    # OBS: O servidor hospedado na máquina local será conectado no banco de dados nomeado rede_exemplo, que possui usuário postgres e senha admin.

#### 12.1.13.3. VENDO QUANTOS GRAFOS A NOVA MALHA POSSUI `(PYTHON)`

    tabela_grafos = 'rede_vicosa_mp' #TABELA COM A REDE
    tabela_vertices = 'rede_vicosa_mp_vertices'
    sql = f'select max(id0) from {tabela_grafos}' #COMANDO EM SQL A SER EXECUTADO
    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    id_max = dados_consultados[0][0] #ID MÁXIMO DA MALHA

#### 12.1.13.4. DEFININDO OS VÉRTICES DE INICIO E FIM `(PYTHON)`

    for id_i in range (1, id_max + 1):
        #CONSULTANDO O VÉRTICE INICIAL DO GRAFO i:

        sql = f'SELECT rvmp.id, rvmp.geom FROM (SELECT ST_LineInterpolatePoints(ST_LineMerge(geom), 0) as geom FROM {tabela_grafos} WHERE id0 = {id_i}) as pto, {tabela_vertices} rvmp WHERE ST_Intersects(pto.geom, rvmp.geom)' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O VÉRTICE INICIAL DO GRAFO i

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        id_ini = dados_consultados[0][0]

        #CONSULTANDO O VÉRTICE FINAL DO GRAFO i

        sql = f'SELECT rvmp.id FROM (SELECT ST_LineInterpolatePoints(ST_LineMerge(geom), 1) as geom FROM {tabela_grafos} WHERE id0 = {id_i}) as pto, {tabela_vertices} rvmp WHERE ST_Intersects(pto.geom, rvmp.geom)' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O VÉRTICE FINAL DO GRAFO i

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        id_fim = dados_consultados[0][0]

        #ATUALIZANDO A TABELA DOS GRAFOS DA REDE:

        sql = f"update {tabela_grafos} set source ='{id_ini}', target = '{id_fim}' where id0 = {id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DOS GRAFOS A SOURCE E TARGET ENCONTRADA.

        cur.execute(sql) #EXECUTANDO O COMANDO

        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} -> v_ini = {id_ini} e v_fim = {id_fim}')

    cur.close() #ENCERRANDO A INSTÂNCIA CRIADA PARA A EXECUÇÃO DO COMANDO

    con.close() #ENCERRANDO A CONEXÃO COM O BANCO DE DADOS
    
**Esse SCRIPT Python se encontra na pasta SCRIPTS_PYTHON > av_dissertacao > projetos > 1_ORGANIZAR_DADOS\VIÇOSA > VERTICES_REDE_VICOSA_SEM_ANEL.ipynb**

#### 12.1.14. VERIFICANDO VÉRTICES INICIAIS E FINAIS DAS LINHAS COM SENTIDO UNIDIRECIONAL `(SQL)`

Novamente é necessário verificar no `QGIS` se os campos source e target das linhas com sentido unidirecional estão corretos. Mas mesmas linhas com problemas descritas no `tópico 12.8` apresentarão problemas. São elas: id 1,2, 37,38, 51, 52, 63 e 64. Para corrigir utilizou-se o seguinte comando:

    -- id do grafo: id_grafo = 1 e id0 = 63 e 64
    UPDATE rede_vicosa_mp
    SET source = '1001'
    WHERE id0 = 63;

    UPDATE rede_vicosa_mp
    SET target = '46'
    WHERE id0 = 63;
    ------
    UPDATE rede_vicosa_mp
    SET source = '47'
    WHERE id0 = 64;

    UPDATE rede_vicosa_mp
    SET target = '1001'
    WHERE id0 = 64;

    -- id do grafo: id_grafo = 62 e id0 = 1 e 2
    UPDATE rede_vicosa_mp
    SET source = '1062'
    WHERE id0 = 1;

    UPDATE rede_vicosa_mp
    SET target = '23'
    WHERE id0 = 1;
    ------
    UPDATE rede_vicosa_mp
    SET source = '80'
    WHERE id0 = 2;

    UPDATE rede_vicosa_mp
    SET target = '1062'
    WHERE id0 = 2;

    -- id do grafo: id_grafo = 71 e id0 = 37 e 38
    UPDATE rede_vicosa_mp
    SET source = '1071'
    WHERE id0 = 37;

    UPDATE rede_vicosa_mp
    SET target = '24'
    WHERE id0 = 37;
    -------
    UPDATE rede_vicosa_mp
    SET source = '25'
    WHERE id0 = 38;

    UPDATE rede_vicosa_mp
    SET target = '1071'
    WHERE id0 = 38;

    -- id do grafo: id_grafo = 96 e id0 = 51 e 52
    UPDATE rede_vicosa_mp
    SET source = '43'
    WHERE id0 = 51;

    UPDATE rede_vicosa_mp
    SET target = '1096'
    WHERE id0 = 51;
    -----
    UPDATE rede_vicosa_mp
    SET source = '1096'
    WHERE id0 = 52;

    UPDATE rede_vicosa_mp
    SET target = '42'
    WHERE id0 = 52;
    
 #### 12.1.15. RENOEMANDO ALGUMAS COLUNAS DA REDE E RECRIANDO A TOPOLOGIA DA REDE:

     ALTER TABLE rede_vicosa_mp
    RENAME COLUMN geom TO the_geom;

    ALTER TABLE rede_vicosa_mp
    RENAME COLUMN id0 TO id;

    ALTER TABLE rede_vicosa_mp
    RENAME COLUMN reverse_co to reverse_cost;

    SELECT pgr_createTopology('rede_vicosa_mp', 1);
    
  Após esse processo a rede está pronta para o cálculo da acessibilidade.
  
  **Os arquivos com os códigos executados estão localizados em:**
  
  **Códigos SQL: SCRIPTS_DISSERTACAO > SCRIPTS_SQL > pgRouting**
  
  **Os arquivos `rede_vicosa`, `rede_vicosa_vertices_pgr`, `rede_vicosa_mp` e `rede_vicosa_mp_vertices` estão localizados na pasta `DADOS_FINAIS`.**
  
#### 12.2. CÁLCULO DA CONECTIVIDADE E ACESSIBILIDADE DA REDE VIÁRIA SEM O ANEL `(PYTHON)`

Com o arquivo `rede_vicosa_mp` é possível calcular a conectividade da malha viária e sua acessibilidade através do custo entre vértices com função `pgr_dijkstra`. Será calculado a matriz de curso para percorrer todos os nós centrais entre si e esses valores correspondem ao custo de percorrer todos arcos entre si e as acessibilidades são resultados dessa matriz.

Primeiramente é necessario instalar o pacote `pandas` para manipular dataframes dentro do Python. O comando para instalar tal pacote deve ser executado no `promp de comando`:

    pip install pandas
    
 Os códigos no Jupyter Notebook são os seguintes:
 
 #### 12.2.1. IMPORTANDO O PACOTE psycopg2 QUE CONECTA O PYTHON COM O POSTGRE-SQL E O PACOTE pandas PARA ORGANIZAR AS TABELAS DE ACESSIBILIDADE
 
     import psycopg2 as pg
    import pandas as pd
    
#### 12.2.2. CONECTANDO AO BANCO DE DADOS

    con = pg.connect(host='localhost', 
                    database='dissertacao',
                    user='postgres', 
                    password='admin')

    cur = con.cursor() #CRIANDO UMA INSTÂNCIA PARA EXECUTAR COMANDOS EM SQL

    # OBS: O servidor hospedado na máquina local será conectado no banco de dados nomeado rede_exemplo, que possui usuário postgres e senha admin.

#### 12.2.3. VENDO QUANTOS GRAFOS A MALHA POSSUI

    tabela_grafos = 'rede_vicosa' #TABELA COM A REDE
    sql = f'select max(id) from {tabela_grafos}' #COMANDO EM SQL A SER EXECUTADO
    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    id_max = e = dados_consultados[0][0] #ID MÁXIMO DA MALHA
    
#### 12.2.4. VENDO QUANTOS VÉRTICES A MALHA POSSUI

    tabela_vertices = 'rede_vicosa_vertices_pgr' #TABELA COM A REDE
    sql = f'select max(id) from {tabela_vertices}' #COMANDO EM SQL A SER EXECUTADO
    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    v = dados_consultados[0][0] #ID MÁXIMO DA MALHA
    
    
#### 12.2.5. CONECTIVIDADE DA MALHA

    #INDICE ALFA: NÚMERO DE CLICOS DA REDE 'u'
    #FORMULA:    u = e-v+1
    #            u_max = 2v-5
    #            alfa = u/u_max ou (e-v+1) / (2v-5)
    #            sendo e: numero de linhas; v: numero de vértices

    #INDICE BETA: SIMPLES RELAÇÃO ENTRE NUMERO DE LINHAS E VÉRTICES
    #FORMULA:    beta = e / v

    #INDICE GAMA: RELAÇÃO NUMERO DE LINHAS OBSERVADAS E O NUMERO MÁXIMO DE LINHAS
    #FORMULA:    gama = e / (3(v-2))

    alfa = (e-v+1)/(2*v-5)
    beta = e/v
    gama = e/(3*(v-2))

    print(f'No cálculo de conectividade da rede, obtemos os seguintes resultados:\nÍndice Alfa: {alfa:.3f}\nÍndice Beta: {beta:.3f}\nÍndice Gama: {gama:.3f}')

#### 12.2.6. NOME DAS TABELAS

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos_acess = 'rede_vicosa_mp' #NOME DE TABELA DA REDE
    abr_grafos = 'rvmp' #ABREVIAÇÃO PARA A TABELA DA REDE
    lista_custos = []
    
#### 12.2.7. LIGAÇÕES ENTRE OS GRAFOS

    for id_i in range(1001, id_max + 1001): #ITERAÇÃO QUE IRÁ PERCORRER TODOS OS GRAFOS COMO VÉRTICE INICIAL

        lista_custos_i = [] #CRIANDO UMA LISTA VAZIA PARA RECEBER OS CUSTOS DO GRAFO i

        for id_j in range(1001, id_max + 1001): #ITERAÇÃO QUE IRÁ PERCORRER TODOS OS GRAFOS COMO VÉRTICE FINAL

            if id_i == id_j: #SE A ITERAÇÃO CALCULAR A DISTANCIA DE UM GRAFO PARA ELE MESMO, PULA A ITERAÇÃO E ADICIONA CUSTO ZERO NA LISTA
                lista_custos_i.append(0)
                continue

            else:

                sql = f"SELECT sum(djk.cost)/2 as tot_cost FROM pgr_dijkstra('SELECT id, source, target, cost, reverse_cost from {tabela_grafos_acess}', {id_i}, {id_j}, true) as djk JOIN {tabela_grafos_acess} {abr_grafos} ON djk.edge = {abr_grafos}.id;" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O VÉRTICE INICIAL E FINAL DO GRAFO DO OBJETIVO FINAL.

                cur.execute(sql) #EXECUTANDO O COMANDO

                dados_consultados = cur.fetchall() #RETORNANDO OS DADOS            

                cost = int(dados_consultados[0][0]) #CUSTO ENTRE i E j

                #ADICIONANDO OS DADOS EM UMA LISTA:
                lista_custos_i.append(cost) #ADICIONANDO O CUSTO DE ATRAVESSAR DO GRAFO i ATÉ O GRAFO j EM UMA LISTA PROVISÓRIA

        lista_custos.append(lista_custos_i) #ADICIONANDO A TABELA PROVISIÓRIA ACIMA EM UMA LISTA QUE TERÁ TODAS INFORMAÇÕES

#### 12.2.8. TRANSFORMANDO A LISTA EM TABELA COM O PANDAS

    #CRIANDO UMA MATRIZ A PARTIR DA LISTA ACIMA:
    matrizCustos = pd.DataFrame(lista_custos)

    #SUBSTITUINDO O CABEÇALHO E LINHAS QUE ESTÁ INDO DE 0 ATÉ 7 PARA OS NOMES DOS GRAFOS QUE VAI DE A ATÉ H:
    matrizCustos_ren = matrizCustos #CRIANDO UMA COPIA DA MATRIZ ANTIGA, PARA PRESERVAR A ESTRUTURA ORIGINAL

    for i in range(0, len(matrizCustos)+1): #ITERAÇÃO PARA SUBSTITUIR O NOME DO CABEÇALHO E DAS LINHAS
        matrizCustos_ren = matrizCustos_ren.rename(columns={i: f'{i+1}'}, index = {i: f'{i+1}'})
        
    matrizCustos
    
#### 12.2.9. Matriz dos custos de grafo a grafo

    matrizCustos_ren
    
#### 12.2.10. Acessibilidade pelo Método 1:

    matrizAcess_conectiv = matrizCustos_ren.filter(items=list(map(lambda x: f'{x}', range(1, id_max + 1))))\
                                            .where(matrizCustos_ren.values == 1) #SELECIONANDO APENAS OS GRAFOS QUE POSSUEM LIGAÇÕES DIRETAS

    matrizAcess_conectiv = matrizAcess_conectiv.apply(lambda x: x.replace(float('NaN'), 0)) #ATRIBUINDO VALOR ZERO PARA OS GRAFOS SEM LIGAÇÕES DIRETAS
    
    matrizAcess_conectiv #MATRIZ DE CONECTIVADE
    
    Acess_1 = matrizAcess_conectiv.sum(axis=1) #SOMA DAS LINHAS DA MATRIZ DE CONECTIVIDADE
    Acess_1 = pd.DataFrame(Acess_1).rename(columns = {0: 'SOMATÓRIO'}) #CRIANDO UM DATA FRAME PARA RECEBER OS DADOS
    
    Acess_1
    
#### 12.2.11.  Acessibilidade pelo Método 2:

    Acess_2 = matrizCustos_ren.max() #VALORES MÁXIMOS PARA CADA GRAFO ATÉ O GRAFO MAIS DISTANTE NA REDE (NÚMERO ASSOCIADO)
    Acess_2 = pd.DataFrame(Acess_2).rename(columns = {0: 'SOMATÓRIO'}) #NÚMERO ASSOCIADO
    
    Acess_2
    
#### 12.2.12. Acessibilidade pelo Método 3:

    # A ORDEM DE LIGAÇÃO MÁXIMA PODE SER OBTIDADE DA TABELA DO CÁLCULO DO NÚMERO ASSOCIADO, SENDO O VALOR MÁXIMO OBTIDO NO CALCULO DE ACESSIBILIDADE DO MÉTODO 2
    lig_max = int(Acess_2.max())

    lista_acess_3 = [] #CRIANDO UMA LISTA QUE RECEBERÁ O SOMATÓRIO DA ACESSIBILIDADE PARA CADA ORDEM

    for i in range(1, lig_max + 1): #ITERAÇÃO QUE IRÁ PERCORRER DA ORDEM 1 ATÉ A ORDEM MÁXIMA

        matrizAcess_ordem_i = matrizCustos_ren.filter(items=list(map(lambda x: f'{x}', range(1, id_max + 1))))\
                                                .where(matrizCustos_ren.values == i) #SELECIONANDO APENAS OS GRAFOS QUE POSSUEM LIGAÇÕES DE ORDEM i

        matrizAcess_ordem_i = matrizAcess_ordem_i.apply(lambda x: x.replace(float('NaN'), 0)) #DEFININDO OS VALORES 'NaN' IGUAL A ZERO

        sumMatrizAcess_ordem_i = matrizAcess_ordem_i.sum(axis=1) #SOMANDO AS LINHAS

        lista_acess_3.append(list(sumMatrizAcess_ordem_i)) #ADICIONANDO O SOMATÓRIO A LISTA DE ACESSIBILIDADE 3

    Acess_3 = pd.DataFrame(pd.DataFrame(lista_acess_3).sum()) #DEFININDO ACESSIBILIDADE PELO MÉTODO TRÊS COMO O SOMATÓRIO DE TODAS ACESSIBILIDADES DE ORDEM N

    #RENOMEANDO AS LINHAS E COLUNAS DESSA MATRIZ:


    Acess_3 = Acess_3.rename(columns = {0: 'SOMATÓRIO'}) #DEFININDO O NOME DA COLUNA

    for i in range(0, len(Acess_3)): #ITERAÇÃO PARA SUBSTITUIR DAS LINHAS
        Acess_3 = Acess_3.rename(index = {i: f'{i+1}'})
    
#### 12.2.13. RESULTADOS:

#### 12.2.13.1. Acessibilidade Método 1:

    Acess_1

#### 12.2.13.2. Acessibilidade Método 2:

    Acess_2

#### 12.2.13.3. Acessibilidade Método 3:

    Acess_3

#### 12.2.14. Adicionando as Acessibilidades na tabela rede_vicosa:

    for id_i in range (1, id_max + 1):

        sql = f"update {tabela_grafos} set acess_1 = '{Acess_1.values[id_i-1][0]}', acess_2 = '{Acess_2.values[id_i-1][0]}', acess_3 = '{Acess_3.values[id_i-1][0]}' where id = {id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA 'REDE_VICOSA' AS ACESSIBILIDADES DAS VIAS.

        cur.execute(sql) #EXECUTANDO O COMANDO

        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'Grafo id {id_i}: Acessibilidade 1: {Acess_1.values[id_i-1][0]}; Acessibilidade 2: {Acess_2.values[id_i-1][0]}; Acessibilidade 3: {Acess_3.values[id_i-1][0]};')


#### 12.2.15. ENCERRANDO A CONEXAO COM O BANCO DE DADOS:

    cur.close() #ENCERRANDO A INSTÂNCIA CRIADA PARA A EXECUÇÃO DO COMANDO

    con.close() #ENCERRANDO A CONEXÃO COM BANCO DE DADOS

**Os scripts acima se encontram na pasta SCRIPTS_DISSERTACAO > SCRIPTS_PYTHON > av_dissertacao > projetos > 2_ACESSIBILIDADE > VIÇOSA  > pgROUTING_RedeVicosa_SEM_ANEL.ipynb**

### 13. CÁLCULO DA CONECTIVIDADE E ACESSIBILIDADE (REDE COM ANEL VIÁRIO)

Todos os processos feitos para a malha viária com o anel proposto por Silva (2012) foram similares ao anterior. Segue os códigos a seguir:

#### 13.1. PRÉ-PROCESSAMENTO
#### 13.1.1. CRIAÇÃO DA REDE A SER ANALISADA `(SQL)`

    -- CRIANDO UMA NOVA TABELA SIMILAR AO 'vias_grafos' CHAMADA 'rede_vicosa':
    SELECT * INTO rede_vicosa_anel FROM vias_grafos_anel ORDER BY id;

#### 13.1.2. CRIAÇÃO DE NOVAS COLUNAS PARA A TABELA `(SQL)`

    -- CRIANDO OS CAMPOS PARA OS VÉRTICES DE FIM E INICIO DO GRAFO:
    ALTER TABLE rede_vicosa_anel
    ADD source INT4,
    ADD target INT4,
    ADD cost REAL,
    ADD reverse_cost REAL;

#### 13.1.3. RENOMEANDO O CAMPO 'geom' PARA 'the_geom' `(SQL)`

    -- RENOMEANDO O CAMPO 'geom' PARA 'the_geom':
    ALTER TABLE rede_vicosa_anel
    RENAME COLUMN geom TO the_geom;

#### 13.1.4. REDEFININDO OS DADOS DE DIREÇÃO DA VIA PARA 'YES' OU 'NO' `(SQL)`

    -- VIAS COM MÃO ÚNICA:
    UPDATE rede_vicosa_anel SET oneway = 'YES' WHERE oneway = 'TF';

    -- VIAS COM MÃO DUPLA:
    UPDATE rede_vicosa_anel SET oneway = 'NO' WHERE oneway = 'B';

#### 13.1.5. DEFININDO OS CUSTOS DOS ARCOS `(SQL)`

    -- DEFININDO CUSTOS 1 PARA TODOS OS VÉRTICES:
    UPDATE rede_vicosa_anel SET cost = 1, reverse_cost = 1;

    -- DEFININDO O CUSTO REVERSOS = -1 PARA OS GRAFOS COM DIREÇÃO ÚNICA:
    UPDATE rede_vicosa_anel SET reverse_cost = '-1' WHERE oneway = 'YES';

#### 13.1.6. CRIANDO A TOPOLOGIA DA REDE `(SQL)`

    -- CRIANDO A TOPOLOGIA DA REDE:
    SELECT pgr_createTopology('rede_vicosa_anel', 1);

#### 13.1.7. CRIANDO VÉRTICES NO MEIO DAS LINHAS `(SQL)`

    -- CRIANDO OS VÉRTICES NO MEIO DE CADA GRAFO:
    CREATE TABLE mid_points_anel AS
    SELECT id, ST_LineInterpolatePoint(ST_LineMerge(the_geom), 0.5) as geom
    FROM rede_vicosa_anel;

    ALTER TABLE mid_points_anel
    ADD CONSTRAINT mid_points_anel_pk PRIMARY KEY (id);

    CREATE INDEX sidx_mid_points_anel
     ON mid_points_anel
     USING GIST (geom);

#### 13.1.8. VERIFICAR SE AS COLUNAS SOURCE E TARGET ESTÃO CORRETAS `(SQL)`

    -- id do grafo: 98 (id grafo sem anel: 1)
    UPDATE rede_vicosa_anel
    SET source = '8', target = '24'
    WHERE id = 98;

    -- id do grafo: 74 (id grafo sem anel: 62)
    UPDATE rede_vicosa_anel
    SET source = '89', target = '34'
    WHERE id = 74;

    -- id do grafo: 49 (id grafo sem anel: 71)
    UPDATE rede_vicosa_anel
    SET source = '87', target = '86'
    WHERE id = 49;

    -- id do grafo: 64 (id grafo sem anel: 96)
    UPDATE rede_vicosa_anel
    SET source = '73', target ='69'
    WHERE id = 64;

#### 13.1.9. CRIANDO A REDE COM PONTOS INTERMEDIÁRIOS ATRAVÉS DO QGIS

Etapa realizada de forma semelhante a malha sem anel viário.

#### 13.1.10. REMOVENTO ELEMENTOS INUTEIS DA rede_vicosa_anel_mp_prov `(SQL)`

    -- REMOVENDO DA CAMADA rede_vicosa_anel_mp AQUELAS GEOMETRIAS NÃO UTEIS PARA A ATUAL ANÁLISE

    CREATE TABLE rede_vicosa_anel_mp AS
    SELECT rvmpp.id, rvmpp.comp_via, rvmpp.nome_rua, rvmpp.tipo_pm, rvmpp.largura_me, rvmpp.pavimento, rvmpp.oneway, rvmpp.decliv_me, rvmpp.id_grafo_s, rvmpp.source, rvmpp.target, rvmpp.cost, rvmpp.reverse_co, ST_Difference(rvmpp.geom, ptos.geom) as geom 
    FROM (SELECT rvmpp.*
    FROM rede_vicosa_anel_mp_prov rvmpp, mid_points_anel mp
    WHERE ST_Contains(mp.geom, rvmpp.geom)) as ptos, rede_vicosa_anel_mp_prov rvmpp
    WHERE ST_Intersects(rvmpp.geom, ptos.geom);

    DELETE FROM rede_vicosa_anel_mp
    WHERE ST_LENGTH(geom) = 0;

    ALTER TABLE rede_vicosa_anel_mp
    ADD COLUMN id0 SERIAL PRIMARY KEY;

    --DELETANDO A TABELA rede_vicosa_anel_mp_prov, JÁ QUE A DEFINITIVA FOI CRIADA
    DROP TABLE rede_vicosa_anel_mp_prov;

    -- ALTERANDO A TABELA REDE_VICOSA_anel_MP:
    ALTER TABLE rede_vicosa_anel_mp
    RENAME COLUMN id TO id_grafo;

#### 13.1.11. RENOMEANDO O id DA TABELA mid_points_anel `(SQL)`

    --ALTERANDO A TABELA 'mid_points_anel' PARA INCLUIR UMA COLUNA CHAMADA ID_GRAFO E ALTERAR O ID PARA 1000 + id DO GRAFO.
    ALTER TABLE mid_points_anel
    ADD id_grafo INT4;

    UPDATE mid_points_anel
    SET id_grafo = id;

    UPDATE mid_points_anel
    SET id = 1000 + id;


#### 13.1.12. CRIANDO A TABELA rede_vicosa_anel_mp_vertices COM TODOS OS VÉRTICES DA rede_vicosa_anel_mp (VÉRTICES DE INÍCIO E FIM DA rede_vicosa_anel E VÉRTICES DA TABELA mid_points_anel `(SQL)`

    -CRIANDO A TABELA rede_vicosa_anel_mp_vertices QUE POSSUIRÁ TODOS OS VERTICES DA REDE PARA CALCULO DE ACESSIBILIDADE:
    SELECT *
    INTO rede_vicosa_anel_mp_vertices
    FROM mid_points_anel;

    ALTER TABLE rede_vicosa_anel_mp_vertices
    ADD CONSTRAINT rede_vicosa_anel_mp_vertices_pk PRIMARY KEY (id);

    INSERT INTO rede_vicosa_anel_mp_vertices (id, geom)
    (SELECT id, the_geom as geom FROM rede_vicosa_anel_vertices_pgr);

    SELECT * FROM rede_vicosa_anel_mp_vertices;

#### 13.1.13. COMPLETANDO A TABELA rede_vicosa_anel_mp COM OS VÉRTICES INICIAIS E FINAIS DE CADA LINHA `(PYTHON)`

Os processo a seguir são feitos em ligaguem python

#### 13.1.13.1. IMPORTANDO O PACOTE psycopg2 QUE CONECTA O PYTHON COM O POSTGRE-SQL E O PACOTE pandas PARA ORGANIZAR AS TABELAS DE ACESSIBILIDADE `(PYTHON)`

    import psycopg2 as pg

#### 13.1.13.2. CONECTANDO AO BANCO DE DADOS `(PYTHON)`

    con = pg.connect(host='localhost', 
                    database='dissertacao',
                    user='postgres', 
                    password='admin')

    cur = con.cursor() #CRIANDO UMA INSTÂNCIA PARA EXECUTAR COMANDOS EM SQL

    # OBS: O servidor hospedado na máquina local será conectado no banco de dados nomeado rede_exemplo, que possui usuário postgres e senha admin.

#### 13.1.13.3. VENDO QUANTOS GRAFOS A NOVA MALHA POSSUI `(PYTHON)`

    tabela_grafos = 'rede_vicosa_anel_mp' #TABELA COM A REDE
    tabela_vertices = 'rede_vicosa_anel_mp_vertices'
    sql = f'select max(id0) from {tabela_grafos}' #COMANDO EM SQL A SER EXECUTADO
    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    id_max = dados_consultados[0][0] #ID MÁXIMO DA MALHA

#### 13.1.13.4. DEFININDO OS VÉRTICES DE INICIO E FIM `(PYTHON)`

        for id_i in range (1, id_max + 1):
        #CONSULTANDO O VÉRTICE INICIAL DO GRAFO i:

        sql = f'SELECT rvmp.id, rvmp.geom FROM (SELECT ST_LineInterpolatePoints(ST_LineMerge(geom), 0) as geom FROM {tabela_grafos} WHERE id0 = {id_i}) as pto, {tabela_vertices} rvmp WHERE ST_Intersects(pto.geom, rvmp.geom)' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O VÉRTICE INICIAL DO GRAFO i

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        id_ini = dados_consultados[0][0]

        #CONSULTANDO O VÉRTICE FINAL DO GRAFO i

        sql = f'SELECT rvmp.id FROM (SELECT ST_LineInterpolatePoints(ST_LineMerge(geom), 1) as geom FROM {tabela_grafos} WHERE id0 = {id_i}) as pto, {tabela_vertices} rvmp WHERE ST_Intersects(pto.geom, rvmp.geom)' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O VÉRTICE FINAL DO GRAFO i

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        id_fim = dados_consultados[0][0]

        #ATUALIZANDO A TABELA DOS GRAFOS DA REDE:

        sql = f"update {tabela_grafos} set source ='{id_ini}', target = '{id_fim}' where id0 = {id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DOS GRAFOS A SOURCE E TARGET ENCONTRADA.

        cur.execute(sql) #EXECUTANDO O COMANDO

        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} -> v_ini = {id_ini} e v_fim = {id_fim}')

    cur.close() #ENCERRANDO A INSTÂNCIA CRIADA PARA A EXECUÇÃO DO COMANDO

    con.close() #ENCERRANDO A CONEXÃO COM O BANCO DE DADOS

**Esse Script se encontra na pasta SCRIPTS_PYTHON > av_dissertacao > projetos > 1_ORGANIZAR_DADOS > VIÇOSA > VERTICES_REDE_VICOSA_COM_ANEL.ipynb**

#### 13.1.14. VERIFICANDO VÉRTICES INICIAIS E FINAIS DAS LINHAS COM SENTIDO UNIDIRECIONAL `(SQL)`

       -- id do grafo: 224, 223 (id grafo sem anel: 1)
    UPDATE rede_vicosa_anel_mp
    SET source = '8', target ='1098'
    WHERE id0 = 224;

    UPDATE rede_vicosa_anel_mp
    SET source = '1098', target = '24'
    WHERE id0 = 223;

    -- id do grafo: 221, 222 (id grafo sem anel: 62)
    UPDATE rede_vicosa_anel_mp
    SET source = '1074', target ='34'
    WHERE id0 = 221;

    UPDATE rede_vicosa_anel_mp
    SET source = '89', target = '1074'
    WHERE id0 = 222;

    -- id do grafo: 207, 208 (id grafo sem anel: 71)
    UPDATE rede_vicosa_anel_mp
    SET source = '1049', target = '86'
    WHERE id0 = 207;

    UPDATE rede_vicosa_anel_mp
    SET source = '87', target = '1049'
    WHERE id0 = 208;

    -- id do grafo: 213, 214 (id grafo sem anel: 96)
    UPDATE rede_vicosa_anel_mp
    SET source = '1064', target = '69'
    WHERE id0 = 213;

    UPDATE rede_vicosa_anel_mp
    SET source = '73', target = '1064'
    WHERE id0 = 214;

#### 13.1.15. RENOEMANDO ALGUMAS COLUNAS DA REDE E RECRIANDO A TOPOLOGIA DA REDE:

    -- RENOEMANDO ALGUMAS COLUNAS DA REDE E RECRIANDO A TOPOLOGIA DA REDE:
    ALTER TABLE rede_vicosa_anel_mp
    RENAME COLUMN geom TO the_geom;

    ALTER TABLE rede_vicosa_anel_mp
    RENAME COLUMN id0 TO id;

    ALTER TABLE rede_vicosa_anel_mp
    RENAME COLUMN reverse_co to reverse_cost;

    ALTER TABLE rede_vicosa_anel_mp
    RENAME COLUMN id_grafo_s to id_grafo_semanel;

    SELECT pgr_createTopology('rede_vicosa_anel_mp', 1);
    
#### 13.2. CÁLCULO DA CONECTIVIDADE E ACESSIBILIDADE DA REDE VIÁRIA SEM O ANEL `(PYTHON)`

#### 13.2.1. IMPORTANDO O PACOTE psycopg2 QUE CONECTA O PYTHON COM O POSTGRE-SQL E O PACOTE pandas PARA ORGANIZAR AS TABELAS DE ACESSIBILIDADE

    import psycopg2 as pg
    import pandas as pd

#### 13.2.2. CONECTANDO AO BANCO DE DADOS

    con = pg.connect(host='localhost', 
                    database='dissertacao',
                    user='postgres', 
                    password='admin')

    cur = con.cursor() #CRIANDO UMA INSTÂNCIA PARA EXECUTAR COMANDOS EM SQL

    # OBS: O servidor hospedado na máquina local será conectado no banco de dados nomeado rede_exemplo, que possui usuário postgres e senha admin.

#### 13.2.3. VENDO QUANTOS GRAFOS A MALHA POSSUI

    tabela_grafos = 'rede_vicosa_anel' #TABELA COM A REDE
    sql = f'select max(id) from {tabela_grafos}' #COMANDO EM SQL A SER EXECUTADO
    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    id_max = e = dados_consultados[0][0] #ID MÁXIMO DA MALHA

#### 13.2.4. VENDO QUANTOS VÉRTICES A MALHA POSSUI

    tabela_vertices = 'rede_vicosa_anel_vertices_pgr' #TABELA COM A REDE
    sql = f'select max(id) from {tabela_vertices}' #COMANDO EM SQL A SER EXECUTADO
    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    v = dados_consultados[0][0] #ID MÁXIMO DA MALHA

#### 13.2.5. CONECTIVIDADE DA MALHA
 
     #INDICE ALFA: NÚMERO DE CLICOS DA REDE 'u'
    #FORMULA:    u = e-v+1
    #            u_max = 2v-5
    #            alfa = u/u_max ou (e-v+1) / (2v-5)
    #            sendo e: numero de linhas; v: numero de vértices

    #INDICE BETA: SIMPLES RELAÇÃO ENTRE NUMERO DE LINHAS E VÉRTICES
    #FORMULA:    beta = e / v

    #INDICE GAMA: RELAÇÃO NUMERO DE LINHAS OBSERVADAS E O NUMERO MÁXIMO DE LINHAS
    #FORMULA:    gama = e / (3(v-2))

    alfa = (e-v+1)/(2*v-5)
    beta = e/v
    gama = e/(3*(v-2))

    print(f'No cálculo de conectividade da rede, obtemos os seguintes resultados:\nÍndice Alfa: {alfa:.3f}\nÍndice Beta: {beta:.3f}\nÍndice Gama: {gama:.3f}')
 
#### 13.2.6. NOME DAS TABELAS

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos_acess = 'rede_vicosa_anel_mp' #NOME DE TABELA DA REDE
    abr_grafos = 'rvmp' #ABREVIAÇÃO PARA A TABELA DA REDE
    lista_custos = []

#### 13.2.7. LIGAÇÕES ENTRE OS GRAFOS
 
     for id_i in range(1001, id_max + 1001): #ITERAÇÃO QUE IRÁ PERCORRER TODOS OS GRAFOS COMO VÉRTICE INICIAL

        lista_custos_i = [] #CRIANDO UMA LISTA VAZIA PARA RECEBER OS CUSTOS DO GRAFO i

        for id_j in range(1001, id_max + 1001): #ITERAÇÃO QUE IRÁ PERCORRER TODOS OS GRAFOS COMO VÉRTICE FINAL

            if id_i == id_j: #SE A ITERAÇÃO CALCULAR A DISTANCIA DE UM GRAFO PARA ELE MESMO, PULA A ITERAÇÃO E ADICIONA CUSTO ZERO NA LISTA
                lista_custos_i.append(0)
                continue

            else:

                sql = f"SELECT sum(djk.cost)/2 as tot_cost FROM pgr_dijkstra('SELECT id, source, target, cost, reverse_cost from {tabela_grafos_acess}', {id_i}, {id_j}, true) as djk JOIN {tabela_grafos_acess} {abr_grafos} ON djk.edge = {abr_grafos}.id;" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O VÉRTICE INICIAL E FINAL DO GRAFO DO OBJETIVO FINAL.

                cur.execute(sql) #EXECUTANDO O COMANDO

                dados_consultados = cur.fetchall() #RETORNANDO OS DADOS            

                cost = int(dados_consultados[0][0]) #CUSTO ENTRE i E j           

                #ADICIONANDO OS DADOS EM UMA LISTA:
                lista_custos_i.append(cost) #ADICIONANDO O CUSTO DE ATRAVESSAR DO GRAFO i ATÉ O GRAFO j EM UMA LISTA PROVISÓRIA

        lista_custos.append(lista_custos_i) #ADICIONANDO A TABELA PROVISIÓRIA ACIMA EM UMA LISTA QUE TERÁ TODAS INFORMAÇÕES

 
#### 13.2.8. TRANSFORMANDO A LISTA EM TABELA COM O PANDAS

    #CRIANDO UMA MATRIZ A PARTIR DA LISTA ACIMA:
    matrizCustos = pd.DataFrame(lista_custos)

    #SUBSTITUINDO O CABEÇALHO E LINHAS QUE ESTÁ INDO DE 0 ATÉ 7 PARA OS NOMES DOS GRAFOS QUE VAI DE A ATÉ H:
    matrizCustos_ren = matrizCustos #CRIANDO UMA COPIA DA MATRIZ ANTIGA, PARA PRESERVAR A ESTRUTURA ORIGINAL

    for i in range(0, len(matrizCustos)+1): #ITERAÇÃO PARA SUBSTITUIR O NOME DO CABEÇALHO E DAS LINHAS
        matrizCustos_ren = matrizCustos_ren.rename(columns={i: f'{i+1}'}, index = {i: f'{i+1}'})
    
    matrizCustos

#### 13.2.9. Matriz dos custos de grafo a grafo
 
    matrizCustos_ren
 
#### 13.2.10. Acessibilidade pelo Método 1:

    matrizAcess_conectiv = matrizCustos_ren.filter(items=list(map(lambda x: f'{x}', range(1, id_max + 1))))\
                                            .where(matrizCustos_ren.values == 1) #SELECIONANDO APENAS OS GRAFOS QUE POSSUEM LIGAÇÕES DIRETAS

    matrizAcess_conectiv = matrizAcess_conectiv.apply(lambda x: x.replace(float('NaN'), 0)) #ATRIBUINDO VALOR ZERO PARA OS GRAFOS SEM LIGAÇÕES DIRETAS
    
    matrizAcess_conectiv #MATRIZ DE CONECTIVADE
    
#### 13.2.11. Acessibilidade pelo Método 2:

    Acess_1 = matrizAcess_conectiv.sum(axis=1) #SOMA DAS LINHAS DA MATRIZ DE CONECTIVIDADE
    Acess_1 = pd.DataFrame(Acess_1).rename(columns = {0: 'SOMATÓRIO'}) #CRIANDO UM DATA FRAME PARA RECEBER OS DADOS

    Acess_1
    
#### 13.2.12. Acessibilidade pelo Método 3:

    Acess_2 = matrizCustos_ren.max() #VALORES MÁXIMOS PARA CADA GRAFO ATÉ O GRAFO MAIS DISTANTE NA REDE (NÚMERO ASSOCIADO)
    Acess_2 = pd.DataFrame(Acess_2).rename(columns = {0: 'SOMATÓRIO'}) #NÚMERO ASSOCIADO

    Acess_2

#### 13.2.13. RESULTADOS:

    # A ORDEM DE LIGAÇÃO MÁXIMA PODE SER OBTIDADE DA TABELA DO CÁLCULO DO NÚMERO ASSOCIADO, SENDO O VALOR MÁXIMO OBTIDO NO CALCULO DE ACESSIBILIDADE DO MÉTODO 2
    lig_max = int(Acess_2.max())

    lista_acess_3 = [] #CRIANDO UMA LISTA QUE RECEBERÁ O SOMATÓRIO DA ACESSIBILIDADE PARA CADA ORDEM

    for i in range(1, lig_max + 1): #ITERAÇÃO QUE IRÁ PERCORRER DA ORDEM 1 ATÉ A ORDEM MÁXIMA

        matrizAcess_ordem_i = matrizCustos_ren.filter(items=list(map(lambda x: f'{x}', range(1, id_max + 1))))\
                                                .where(matrizCustos_ren.values == i) #SELECIONANDO APENAS OS GRAFOS QUE POSSUEM LIGAÇÕES DE ORDEM i

        matrizAcess_ordem_i = matrizAcess_ordem_i.apply(lambda x: x.replace(float('NaN'), 0)) #DEFININDO OS VALORES 'NaN' IGUAL A ZERO

        sumMatrizAcess_ordem_i = matrizAcess_ordem_i.sum(axis=1) #SOMANDO AS LINHAS

        lista_acess_3.append(list(sumMatrizAcess_ordem_i)) #ADICIONANDO O SOMATÓRIO A LISTA DE ACESSIBILIDADE 3

    Acess_3 = pd.DataFrame(pd.DataFrame(lista_acess_3).sum()) #DEFININDO ACESSIBILIDADE PELO MÉTODO TRÊS COMO O SOMATÓRIO DE TODAS ACESSIBILIDADES DE ORDEM N

    #RENOMEANDO AS LINHAS E COLUNAS DESSA MATRIZ:


    Acess_3 = Acess_3.rename(columns = {0: 'SOMATÓRIO'}) #DEFININDO O NOME DA COLUNA

    for i in range(0, len(Acess_3)): #ITERAÇÃO PARA SUBSTITUIR DAS LINHAS
        Acess_3 = Acess_3.rename(index = {i: f'{i+1}'})


#### 13.2.13.1. Acessibilidade Método 1:

    Acess_1

#### 13.2.13.2. Acessibilidade Método 2:

    Acess_2

#### 13.2.13.3. Acessibilidade Método 3:

    Acess_3

#### 13.2.14. Adicionando as Acessibilidades na tabela rede_vicosa:

    for id_i in range (1, id_max + 1):

        sql = f"update {tabela_grafos} set acess_1 = '{Acess_1.values[id_i-1][0]}', acess_2 = '{Acess_2.values[id_i-1][0]}', acess_3 = '{Acess_3.values[id_i-1][0]}' where id = {id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA 'REDE_VICOSA' AS ACESSIBILIDADES DAS VIAS.

        cur.execute(sql) #EXECUTANDO O COMANDO

        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'Grafo id {id_i}: Acessibilidade 1: {Acess_1.values[id_i-1][0]}; Acessibilidade 2: {Acess_2.values[id_i-1][0]}; Acessibilidade 3: {Acess_3.values[id_i-1][0]};')

    cur.close() #ENCERRANDO A INSTÂNCIA CRIADA PARA A EXECUÇÃO DO COMANDO

    con.close() #ENCERRANDO A CONEXÃO COM BANCO DE DADOS
    
    
**OBS. 1: Os scripts em SQL estão na pasta: SCRIPTS_DISSERTACAO > SCRIPTS_SQL > pgRouting_anel**

**OBS. 2: Os scripts em Python estão localizados na pasta: SCRIPTS_DISSERTACAO > SCRIPTS_PYTHON > av_dissertacao > projetos > 2_ACESSIBILIDADE > VIÇOSA > pgROUTING_RedeVicosa_COM_ANEL.ipynb**

**Os arquivos `rede_vicosa_anel`, `rede_vicosa_anel_vertices_pgr`, `rede_vicosa_anel_mp` e `rede_vicosa_anel_mp_vertices` estão localizados na pasta `DADOS_FINAIS`.**

## APÓS ESSES PROCESSOS OS ARQUIVOS `rede_vicosa` E `rede_vicosa_anel` POSSUEM A ACESSIBILIDADE DE CADA LINHA EM SUA TABELA DE ATRIBUTOS, PRONTAS PARA SEREM ABERTAS NO QGIS.

*CONTATO:*

Augusto Franco de Lima

e-mail pessoal: augustolimafrfr@gmail.com

e-mail institucional: augusto.f.lima@ufv.br
