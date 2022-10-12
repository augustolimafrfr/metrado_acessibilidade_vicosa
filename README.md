# TUTORIAL PARA ORGANIZAÇÃO DA BASE DE DADOS - ACESSIBILIDADE VIAS DE VIÇOSA
Este é um tutorial com o passo-a-passo de todo o procedimento metodológico utilizado na dissertação de mestrado para a preparação da base de dados do município de Viçosa com o objetivo de calcular a acessibilidade de suas principais vias. Os softwares utilizados durante o processo foram: `PostgreeSQL 14` (com o pacote de extensões para dados espaciais PostGis), `Python 3` (juntamente com a IDE Jupyter Notebook), `QGIS 3.22.7` e `ArcGIS 10.5`.

O objetivo desse tutorial é explicar de forma detalhada os procedimentos realizados para organizar a base de dados das vias de Viçosa e com isso, será possível entender o processo e replicá-lo para demais bases de dados.

Nas pastas X e Y...

## PASSO-A-PASSO:

### **1. ORIGEM DOS DADOS:**

Os dados utilizados nesse trabalho foram fornecidos por Vieira e Castro (2021). Trata-se de um conjunto de arquivos no formato shapefile com grande parte das vias de Viçosa com diverssos atributos.

### **2. SELEÇÃO DOS ATRIBUTOS UTEIS:**

Os atributos de interesse para a dissertação foram `largura da via`, `pavimento`, `sentido da via`. Portanto, todos os demais atributos foram excluidos através da ferramenta `Editar campos` do software `QGIS`. Dois novos campos foram criados: `nome da rua` e `tipo_pm`, sendo que o primeiro será utilizado para incluir o nome das vias e o segundo a classificação de acordo com o plano de mobilidade.

### **3. SELEÇÃO DAS VIAS A SEREM UTILIZADAS:**
As vias mencionadas no Plano de Mobilidade de Viçosa foram selecionadas utilizando o `Open Street Map` e o `Google Maps` como pano de fundo. Durante esse processo os campos os campos `nome da rua` e `tipo_pm` foram preenchidas.

### **4. GARATINDO A INTEGRIDADE TOPOLÓGICA DA REDE:**

Para garantir a integridade topológica da rede foi utilizado o complemento `Verificador de topologia` do software `QGIS`. No botão `Configurações`, seleciona-se o arquivo da malha viária e marca-se a opção `não devem ter dangles`, adiciona-se a regra e confirma. Na janela aberta clica-se em `Validar tudo`. Surgirão pontos vermelhos em alguns locais da malha viária e será possível descobrir quais linhas estão desconectadas (esses pontos vermelhos devem existir apenas onde a via realmente está interrompida, como as extremidades).

### **5. OBTENÇÃO DA REDE EM FORMA DE GRAFOS:**

A malha viária obtida até então não está na forma de grafos. Uma rede viária na forma de grafos deve ser dividida apenas na junção entre os vértices. Para gerar a malha viária de Viçosa dessa forma utilizou-se o software `ArcGIS`. Primeiramente utilizou-se a ferramenta `Dissolve`para agregar todos elementos em apenas uma feição e para concluir utilizou-se a ferramenta `Explode Multipart Feature` da caixa de ferramentas `Advanced Editing`. Os atributos presentes no arquivo anterior devem ser reciados e novos também foram criados. Por fim, os atributos dessa rede foram: `nome da rua`, `tipo_pm`, `comprimento da via`,`largura media`, `pavimentaçao`, `sentido da via` e `declividade media`. Com exceção do atributo `comprimento da via`, todos os demais estão em branco e foram prenchidos posteriormente.

### **6. DIVIDINDO A MALHA VIÁRIA (GERADA NO PASSO 4) EM DIVERSAS MALHAS COM SEUS RESPECTIVOS ATRIBUTOS:**

Antes de preencher os atributos da rede viária em forma de grafos, realizou-se um procedimento com a malha obtida do `passo 4`. A partir dessa malha, gerou-se cinco novos arquivos utilizando a ferramenta Dissolve do software `ArcGIS` para separar cada atributo da rede em um arquivo diferente. Na ferramenta mencionada selecionou-se o shapefile da malha com todos atributos e na aba `Dissolve_Fields (optional)` marcou-se o atributo desejado para a separação. Na aba `Output Feature Class` atribuiu-se o nome do novo arquivo. Esse processo deve ser feito cinco vezes para gerar os arquivos para cada um dos cinco atributos (nome da rua, tipo_pm, sentido da via, declividade media, largura). Esses cinco novos dados foram manipulados pela ferramenta `Explodir linhas` do software `QGIS`, com o objetivo de obter os atributos de cada segmento de reta presente na malha viária.

## 
