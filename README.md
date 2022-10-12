# TUTORIAL PARA ORGANIZAÇÃO DA BASE DE DADOS - ACESSIBILIDADE VIAS DE VIÇOSA
Este é um tutorial com o passo-a-passo de todo o procedimento metodológico utilizado na dissertação de mestrado para a preparação da base de dados do município de Viçosa com o objetivo de calcular a acessibilidade de suas principais vias. Os softwares utilizados durante o processo foram: `PostgreeSQL 14` (com o pacote de extensões para dados espaciais PostGis), `Python 3` (juntamente com a IDE Jupyter Notebook), `QGIS 3.22.7` e `ArcGIS 10.5`.

O objetivo desse tutorial é explicar de forma detalhada os procedimentos realizados para organizar a base de dados das vias de Viçosa e com isso, será possível entender o processo e replicá-lo para demais bases de dados.

**Todos arquivos utilizados na dissertação estão localizados na pasta `DADOS_DISSERTAÇÃO`, que possui diversas sub-pastas com os arquivos de forma organizada. **

## PASSO-A-PASSO:

### 1. ORIGEM DOS DADOS:

Os dados utilizados nesse trabalho foram fornecidos por Vieira e Castro (2021) e pelo Serviço Autônomo de Água e Esgoto do município de Viçosa (SAAE). O primeiro arquivo nomeado como `Rede_Viaria_RSU_2021` trata-se do shapefile contando as ruas de Viçosa e conta com os atributos como **Nome da Rua**, **Tipo de Eixo**,** Largura da rua**, **Pavimento**, **Declividade**, **Bairro**, entre outros. Os dados do SAAE, nomeado como `declividade_pontos_cotados_SAAE`, consistem em pontos cotados dos meio-fios das ruas ao longo de Viçosa. A malha viária de Vieira e Castro (2021) será utilizada para formar a rede de análise e os pontos cotados para gerar o Modelo Digital de Elevação (MDE) do município.

**Os arquivos `Rede_Viaria_RSU_2021` e `declividade_pontos_cotados_SAAE` estão localizados na pasta `DADOS INICIAIS`.**


### 2. SELEÇÃO DOS ATRIBUTOS UTEIS:

Diverssos atributos presentes em `Rede_Viaria_RSU_2021` não serão úteis para esse trabalho e outros serão cálculados novamente. Portanto, da tabela de atributos dos dados originais, aqueles escolhidos foram: **largura da via**, **pavimento**, **sentido da via** e todos os demais atributos foram excluidos através da ferramenta `Editar campos` do software `QGIS`. Dois novos campos foram criados: **nome da rua** e **tipo_pm**, sendo que o primeiro será utilizado para incluir o nome das vias e o segundo a classificação de acordo com o plano de mobilidade.

### 3. SELEÇÃO DAS VIAS A SEREM UTILIZADAS:
As vias mencionadas no Plano de Mobilidade de Viçosa foram selecionadas utilizando o `Open Street Map` e o `Google Maps` como pano de fundo. Durante esse processo os campos os campos **nome da rua **e **tipo_pm** foram preenchidos.

### 4. GARATINDO A INTEGRIDADE TOPOLÓGICA DA REDE:

Para garantir a integridade topológica da rede foi utilizado o complemento `Verificador de topologia` do software `QGIS`. No botão `Configurações`, seleciona-se o arquivo da malha viária e marca-se a opção `não devem ter dangles`, adiciona-se a regra e confirma. Na janela aberta clica-se em `Validar tudo`. Surgirão pontos vermelhos em alguns locais da malha viária e será possível descobrir quais linhas estão desconectadas (esses pontos vermelhos devem existir apenas onde a via realmente está interrompida, como as extremidades).

### 5. OBTENÇÃO DA REDE EM FORMA DE GRAFOS:

A malha viária obtida até então não está na forma de grafos. Uma rede viária na forma de grafos deve ser dividida apenas na junção entre os vértices. Para gerar a malha viária de Viçosa dessa forma utilizou-se o software `ArcGIS`. Primeiramente utilizou-se a ferramenta `Dissolve`para agregar todos elementos em apenas uma feição e para concluir utilizou-se a ferramenta `Explode Multipart Feature` da caixa de ferramentas `Advanced Editing`. Os atributos presentes no arquivo anterior devem ser reciados e novos também foram criados. Por fim, os atributos dessa rede foram: **nome da rua**, **tipo_pm**, **comprimento da via**, **largura media**, **pavimentaçao**, **sentido da via** e **declividade media**. Com exceção do atributo **comprimento da via**, todos os demais estão em branco e foram prenchidos posteriormente.

**Esse arquivo foi nomeado como `vias_grafos` e está localizado na pasta `DADOS_INTERMEDIARIOS`.**

### 6. DIVIDINDO A MALHA VIÁRIA (GERADA NO PASSO 4) EM DIVERSAS MALHAS COM SEUS RESPECTIVOS ATRIBUTOS:

Antes de preencher os atributos do arquivo `vias_grafos`, realizou-se um procedimento com a malha obtida do `passo 4`. A partir dessa malha, gerou-se cinco novos arquivos utilizando a ferramenta Dissolve do software `ArcGIS`, com o objetivo de separar cada atributo da rede em um arquivo diferente.

Na ferramenta mencionada selecionou-se a malha viária (que não está na forma de grafos) com todos atributos e na aba `Dissolve_Fields (optional)` marcou-se um dos atributos desejados para a separação. Na aba `Output Feature Class` atribuiu-se o nome do novo arquivo (de forma que remeta ao atributo selecionado) e executou-se a ferramenta. Esse processo deve ser feito cinco vezes para gerar os arquivos para cada um dos cinco atributos (**nome da rua**, **tipo_pm**, **sentido da via**, **pavimento**, **largura**). Esses cinco novos dados foram manipulados pela ferramenta `Explodir linhas` do software `QGIS`, com o objetivo de obter os atributos de cada segmento de reta presente na malha viária. Os cinco arquivos foram nomeados como `vias_nomeruas` (com o nome de rua de cada segmento de reta), `vias_tipopm` (com o tipo de classificação de via de acordo com o Plnao de Mobilidade de Viçosa para cada segmento de reta), `vias_oneway` (com o sentido da via para cada segmento de reta), `vias_pavimentos` (com a pavimentação de cada segmento de reta), `vias_largura` (com a largura de cada segumento de reta).

**Os cinco arquivos se encontram na pasta `DADOS_INTERMEDIARIOS`**

## 7. GERAÇÃO DO MDE

O MDE de Viçosa foi gerado a partir do arquivo `declividade_pontos_cotados_SAAE`. Para isso, utilizou-se a ferramenta `Interpolação IDW` do software `QGIS`. O novo arquivo foi recortado através da ferramenta `Recortar raster pela extensão...` com o objetivo de reduzir a extensão do MDE de forma que englobe apenas a área de estudo. Apesar desse passo não ser obrigatório, é interessante recortar o arquivo raster com o objetivo de reduzir o seu tamanho e facilitar os processamentos. O arquivo que contem o MDE de Viçosa foi nomeado como `mde_vicosa`.

### 8. IMPORTANDO TODOS OS DADOS PARA O BANCO DE DADOS

Todos arquivos gerados foram importados para o banco de dados. O procedimento para importar arquivos no formato vetorial (nesse estudo em formato shapefile) e raster são distintos.

#### 8.1. IMPORTANDO OS ARQUIVOS DO FORMATO SHAPEFILE

 Os arquivos `vias_grafos`, `vias_nomeruas`, `vias_tipopm`, `vias_oneway`, `vias_pavimentos` e `vias_largura` foram importados através da ferramenta `Gerenciador BD...` disponível após a instalação do `PostGIS`. O procedimento é feito selecionando de forma individual o arquivo que será importado na aba **Entrada**, em **Esquema** seleciona-se a opção _public_ e em **Tabela** atribuiu-se o nome desejado da tabela (igual ao nome do arquivo em formato shapefile). Isso foi feito para todos os arquivos.

#### 8.2. IMPORTANDO OS ARQUIVOS DO FORMATO RASTER

O MDE foi importado através do promp de comando. Com o promp aberto digita-se os seguintes comandos:

``
