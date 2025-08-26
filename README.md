# Sistema de Perguntas e Respostas com SLMs Gratuitos

### *Equipe:*
* Álvaro Luís Silva Peixoto
* Charlie Rodrigues Fonseca
* Ryan dos Santos Carvalho
---

### *Vídeo Elaborado*
[Acesso ao vídeo](https://drive.google.com/file/d/1P1qFPS0OpZmyfzf0QjeJkQHKSgCktAzV/view?usp=sharing)
---

### *Resumo*

Este relatório detalha o desenvolvimento e a avaliação de um sistema de Perguntas e Respostas (QA) projetado para comparar a eficácia de três *Modelos de Linguagem Pequenos (SLMs)* gratuitos em extrair informações de dois formatos de documentos distintos: um PDF não estruturado sobre doenças respiratórias e um DOCX contendo um dicionário de dados com tabelas complexas. A arquitetura do sistema foi baseada em Geração Aumentada por Recuperação (RAG), utilizando um método de busca híbrida. O principal desafio do projeto foi a interpretação das tabelas no arquivo DOCX, que resultou em um desempenho inicial baixo. Para mitigar esse problema, foram implementadas estratégias de processamento de texto específicas para tabelas e, posteriormente, foi integrado um modelo especializado (TAPAS). A avaliação, realizada com métricas de F1-Score, Exact Match e Similaridade Semântica, demonstrou que, embora os resultados quantitativos tenham sido limitados, a abordagem forneceu insights valiosos sobre as limitações de SLMs genéricos para dados estruturados e a importância de modelos especializados para tarefas específicas.

### *1. Introdução*

A ascensão dos Modelos de Linguagem abriu novas fronteiras para aplicações de Processamento de Linguagem Natural (PLN). Enquanto os Modelos de Linguagem Grandes (LLMs) dominam as manchetes, os *Modelos de Linguagem Pequenos (SLMs)*, como os abordados neste trabalho, representam uma alternativa eficiente e acessível, ideal para aplicações com recursos computacionais limitados. A técnica de Geração Aumentada por Recuperação (RAG) permite que esses modelos baseiem suas respostas em um corpus de conhecimento externo, aumentando sua relevância e confiabilidade.

O objetivo deste projeto foi construir um sistema de QA para avaliar e comparar, de forma objetiva, o desempenho de três SLMs gratuitos disponíveis no Hugging Face: *BERT-pt, **XLM-RoBERTa-multi* e *Gemma-2b-it*. A comparação foi realizada em dois cenários distintos:

1.  Um documento *PDF não estruturado* (doencas_respiratorias_cronicas.pdf), contendo texto corrido.
2.  Um documento *DOCX com tabelas complexas* (dicionario_de_dados.docx), representando um desafio para a extração de dados estruturados.

Este relatório detalha a metodologia empregada, a avaliação quantitativa dos modelos e a análise dos resultados, com foco nos desafios e aprendizados obtidos.

### *2. Metodologia*

A arquitetura do sistema foi desenvolvida em um notebook do Google Colab, seguindo as etapas de um pipeline RAG clássico, com adaptações para os diferentes tipos de arquivo.

#### *2.1. Arquitetura do Sistema*

O sistema foi estruturado da seguinte forma:

1.  *Carregamento e Processamento de Documentos:* Funções específicas para extrair conteúdo de PDFs e DOCX.
2.  *Divisão de Texto (Chunking):* O texto extraído foi dividido em blocos menores (chunks).
3.  *Indexação (Retriever):* Um modelo de embeddings (intfloat/multilingual-e5-base) converteu os chunks em vetores, indexados com FAISS (busca semântica) e BM25 (busca por palavra-chave).
4.  *Busca Híbrida:* Uma combinação dos resultados de FAISS e BM25 para recuperar os chunks mais relevantes.
5.  *Geração de Resposta (Reader):* Os chunks recuperados foram passados como contexto para os SLMs ou para o modelo especializado para gerar a resposta.

#### *2.2. Processamento de Documentos*

O maior desafio residiu no tratamento do documento DOCX. As estratégias foram:

* *PDF (Texto não estruturado):* O texto foi extraído e dividido em chunks de 768 caracteres, uma abordagem padrão para texto corrido.
* *DOCX (Tabelas Complexas):* Foi implementada uma função avançada que cria *chunks granulares* em formato de sentença para cada linha da tabela (ex: "Na tabela LFCES038, o campo COD_MUN tem a descrição...") e um *"super-chunk"* resumindo cada tabela. Essa dupla abordagem visou fornecer contexto detalhado e geral.

#### *2.3. Modelos Utilizados*

* *Modelo de Embeddings (Retriever):* Após testes iniciais, optamos pelo **intfloat/multilingual-e5-base** por sua performance robusta em domínios diversos.
* *Modelos de Leitura (Readers) - SLMs:* Foram selecionados três SLMs por sua eficiência e disponibilidade:
    1.  **pierreguillou/bert-base-cased-squad-v1.1-portuguese (BERT-pt):** Um modelo extrativo robusto, fine-tuned para português.
    2.  **deepset/xlm-roberta-base-squad2 (XLM-R):** Um modelo extrativo multilíngue.
    3.  **google/gemma-2b-it (Gemma):** Um modelo generativo moderno e de alta performance.
* *Modelo Especializado para Tabelas:*
    * **google/tapas-base-finetuned-sqa (TAPAS):** Devido à dificuldade dos SLMs com o DOCX, introduzimos o TAPAS, um modelo projetado para responder perguntas diretamente sobre dados tabulares.

#### *2.4. Perguntas e Avaliação de Desempenho*

Foram elaboradas três perguntas para cada documento, visando testar diferentes capacidades de extração de informação.

*Perguntas (dicionario_de_dados.docx):*

1.  Qual é a descrição da tabela NFCES058?
2.  Quais são todos os campos que compõem a chave primária da tabela de Profissionais das Equipes, identificada como LFCES038?
3.  Qual é a descrição da tabela de domínio NFCES028 e qual o seu nome correspondente no banco de produção federal?

*Perguntas (doencas_respiratorias_cronicas.pdf):*

1.  Quais são os critérios (relação VEF1/CVF e VEF1 do previsto) para um paciente ser classificado no Estádio 3 (DPOC grave)?
2.  Qual a pontuação necessária no Teste de Fagerstrom para que o grau de dependência seja classificado como "muito elevado"?
3.  No manejo da crise de asma em uma unidade de saúde, qual é o tratamento inicial recomendado para um paciente que apresenta uma crise grave?

Para a avaliação, foram criadas respostas "padrão-ouro" (ground truth) e calculadas as seguintes métricas:

* *Exact Match (EM):* Correspondência exata da resposta.
* *F1-Score:* Média harmônica entre precisão e recall de tokens.
* *Similaridade Semântica:* Similaridade de cosseno entre os embeddings da resposta gerada e da esperada.

### *3. Resultados e Análise*

A tabela a seguir resume o desempenho médio dos modelos para cada documento:

| *Documento* | *Modelo* | *Exact Match* | *F1 Score* | *Semantic Similarity* |
| :----------------------- | :------------------ | :-------------- | :----------- | :-------------------- |
| Dicionário de Dados      | BERT-pt             | 0.0             | 0.000000     | 0.831797              |
| Dicionário de Dados      | Gemma-2b-it         | 0.0             | 0.000000     | 0.802216              |
| Dicionário de Dados      | TAPAS               | 0.0             | 0.066667     | 0.829888              |
| Dicionário de Dados      | XLM-RoBERTa-multi   | 0.0             | 0.000000     | 0.841953              |
| Doenças Respiratórias    | BERT-pt             | 0.0             | 0.148148     | 0.843889              |
| Doenças Respiratórias    | Gemma-2b-it         | 0.0             | 0.188811     | 0.866925              |
| Doenças Respiratórias    | XLM-RoBERTa-multi   | 0.0             | 0.095238     | 0.869877              |

* *Desempenho no PDF:* Os resultados foram melhores para o documento não estruturado. O SLM *Gemma* obteve o maior F1-Score (0.189), indicando a melhor capacidade de gerar termos relevantes, enquanto o *XLM-RoBERTa-multi* alcançou a maior similaridade semântica (0.870), sugerindo uma compreensão superior do contexto geral.
* *Desempenho no DOCX:* Este foi o maior desafio. Os SLMs baseados em RAG obtiveram *F1-Score de 0, reforçando a ineficácia da abordagem de chunking para dados tabulares. A introdução do modelo **TAPAS* foi crucial: ele foi o único a obter um *F1-Score maior que zero (0.067)*, demonstrando que um modelo especializado, mesmo com desempenho modesto, é superior para essa tarefa.

#### *3.1. Análise Qualitativa de Exemplos*

*Exemplo 1: Documento PDF (Texto Não Estruturado)*

* *Pergunta:* Quais são os critérios \[...\] para um paciente ser classificado no Estádio 3 (DPOC grave)?
* *Ground Truth:* VEF1/CVF<0,70 e 30%≤ VEF1<50% do previsto.
* *Respostas dos Modelos:*
    * *BERT-pt (Extrativo):* < 0,70 e 30%
    * *Gemma (Generativo):* Os critérios [...] são VEF1/CVF < 0,70 e 30% ≤ VEF1<50% do previsto.
* *Análise:* O *BERT-pt* extraiu um trecho incompleto. O *Gemma*, por sua capacidade generativa, sintetizou uma resposta completa e bem formulada, mostrando a vantagem de SLMs generativos para tarefas que exigem mais do que a simples extração.

*Exemplo 2: Documento DOCX (Tabelas Complexas)*

* *Pergunta:* Qual é a descrição da tabela NFCES058?
* *Ground Truth:* DETALHAMENTO DO VÍNCULO.
* *Respostas dos Modelos:*
    * *BERT-pt (RAG):* Código da Área na qual o Profissional completa a Carga Horária
    * *TAPAS (Especializado):* Indica a vinculação, o tipo e o sub tipo de vínculo do Profissional com o Estabelecimento de Saúde
* *Análise:* O *BERT-pt* "alucinou" uma resposta a partir de um chunk incorreto, um risco comum em SLMs quando o contexto recuperado não é preciso. O *TAPAS*, por outro lado, operou diretamente na tabela e forneceu uma resposta semanticamente próxima e útil. Isso comprova que, para dados tabulares, um modelo especializado é indispensável.

### *4. Desafios e Insights*

* *Principal Desafio:* A extração de informações de tabelas em um pipeline RAG com SLMs é extremamente desafiadora. A divisão do texto (chunking) quebra a estrutura relacional dos dados, dificultando a interpretação.
* *Principal Insight:* Para documentos com dados estruturados, uma abordagem que utiliza modelos especializados como o TAPAS é fundamental. Os SLMs, embora eficientes, mostram limitações em tarefas que exigem raciocínio estruturado, reforçando a necessidade de arquiteturas específicas para cada tipo de conteúdo.
* *Valor das Métricas:* A Similaridade Semântica se mostrou uma métrica mais útil que o F1-Score para avaliar SLMs generativos como o Gemma.

### *5. Uso de LLMs no Desenvolvimento*

Em conformidade com as boas práticas acadêmicas, registramos o uso de *Modelos de Linguagem Grandes (LLMs)* durante o desenvolvimento deste projeto. Utilizamos o *Google Gemini* (integrado ao ambiente Colab) e o *ChatGPT* para as seguintes finalidades, que foram distintas do escopo de avaliação dos SLMs:

* *Aprimoramento de Código:* Sugestões para otimizar funções.
* *Organização do Notebook:* Auxílio na estruturação das seções.
* *Depuração (Debugging):* Ajuda na identificação e correção de erros.
* *Redação do Relatório:* O modelo Gemini também foi utilizado como assistente na elaboração e revisão deste documento, ajudando a estruturar as seções e a refinar a linguagem técnica.

### *6. Conclusão*

O projeto demonstrou com sucesso a viabilidade de construir um sistema de QA para comparar SLMs gratuitos, ao mesmo tempo que expôs as limitações inerentes das abordagens RAG atuais para lidar com dados estruturados. A evolução da metodologia, desde um pipeline genérico até a introdução de um modelo especializado, reflete uma jornada de aprendizado crucial. Embora os scores quantitativos finais não tenham sido altos, os insights obtidos são de grande valor: a necessidade de arquiteturas específicas por tipo de conteúdo e a importância de selecionar as métricas de avaliação corretas para cada tipo de modelo são as principais lições aprendidas.
