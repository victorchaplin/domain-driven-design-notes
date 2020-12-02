# DOMAIN-DRIVEN DESIGN
Anotações baseadas na série "DDD do jeito certo" da EximiaCo
https://www.youtube.com/playlist?list=PLkpjQs-GfEMN8CHp7tIQqg6JFowrIX9ve

## COMPLEXIDADE DESNECESSARIA É CUSTO!
A ideia do DDD é modelar a aplicação prestando muita atenção às regras de negocio para garantir que o codigo reflita de maneira simples as operações e processos do negocio.

## Atacando a complexidade do software
- complexidade do software é uma soma de 3 coisas:
  - complexidade da solução tecnica
  - complexidade do legado
  - complexidade do dominio
- somos pagos para resolver a complexidade de dominio, as outras complexidades sao acidentais
- complexidade do dominio é conhecida pelos domain experts
- DDD agrega pouco valor se o dominio é muito simples

## Linguagem ubiqua (onipresente)
- tecnicos e especialistas de dominio devem falar a mesma lingua, assim se aproximam
- o dominio nao pode ser anemico:
  - garantir que suas classes mereçam ter testes :)
  - se focar nos motivos para mudança das propriedades da classe
  - usar corretamente a orientação a objetos e evitar codigo procedural
- importante conversar com varios domain experts
- quando domain experts usam o mesmo termo para se referir a coisas diferentes, isso normalmente é um sinal de que existe um subdominio

## Subdomains e bounded contexts
- uma das propostas do DDD é criar dois espaços: um do problema e um da solução
  - espaço do problema: aspectos do que precisa ser resolvido
  - espaço da solução: como resolver as coisas descritas no espaço de problema
- normalmente dominios mais complexos acabam tendo separações ou áreas, que são subdominios
- no espaço do problema, temos o dominio e o subdominio, no espaço da solução temos o modelo de dominio e o contexto delimitado, respectivamente
- composição do domain:
  - core domain: atividades do dominio que diferenciam o negocio. é o centro do problema
  - subdominios de suporte: subdominios que suportam a operação principal do negocio
  - subdominios genericos: não tem relação direta com o core domain
- normalmente os especialistas de dominio vão dizer que a sua area é o core domain, porem é importante identificar corretamente o core domain do negocio para coloca-lo no centro do design, colocar em torno dele os subdominios de suporte e saber quais atividades genericas
- geralmente o core domain é a parte do sistema que costuma ter necessidade de customização, e por isso feito em casa
- atividades genericas costumam ser atendidas por softwares de prateleira

## Context Mapping
- mapear os contextos pode ser complicado devido à possiveis semelhanças entre eles
- eventualmente, quando a relação entre dois contextos é muito proxima, pode-se compartilhar parte dos recursos, ou seja, criar um Shared Kernel
- no caso de shared kernels, muitas vezes se usa inner sourcing para reduzir atrito, mas mesmo assim esse shared kernel tem um dono
- sempre prestar atenção na relação fornecedor-cliente (upstream e downstream) quando estiver mapeando contexto
- muitas vezes o fornecedor vai fazer alterações no seu modelo de dominio sem levar em consideração o impacto nos consumidores. nesses casos é importante o time que cuida do bounded context se precaver para mitigar danos em caso de mudança na interface
- quando há uma relação de conformismo com o fornecedor, deve-se evitar ter um acoplamento forte com o fornecedor. pra isso, se cria um modelo intermediario que representa a base do modelo de dominio do bounded context. essa representação minima se Anti-Corruption Layer
- anti-corruption layer é muito util no caso de pluralidade de fornecedores

## Entity e Value Object
- basicamente, sempre que um domain expert usa um substantivo pra descrever um processo do dominio, esse substantivo é uma entidade ou um objeto de valor
- entidades:
  - aqueles substantivos que existem no espaço do problema. normalmente podem ser identificados por um id, e esse id é usado para saber quando estamos falando de uma mesma entidade
  - o conceito de entidade prevê mutabilidade
  - servem como "cola" entre os bounded contexts
- objeto de valor:
  - aparece sempre ligado a uma entidade
  - é usado como descrição ou parte de uma entidade
  - objetos de valor nao possuem id. podem até possuir, mas esse id nao é usado para comparação com outro objeto

## Agregados
- são delimitações transacionais do modelo
- agregados não devem ser representados diretamente no codigo, esses limites simplesmente existem (sao conceituais)
- geralmente os agregados representam pontos de entrada para o modelo/sistema. esse ponto de entrada é conhecido como a raiz do agregado e dá nome ao bloco conceitual do agregado
- agregados devem ser pequenos, pois se forem grandes, as entradas podem ficar grandes demais (ex. web api, banco de dados)

## Eventos de dominio
- eventos de dominio servem para que um bounded context consiga avisar um outro bounded context que algo importante aconteceu
- no espaço do problema, um evento de dominio é um registro da ocorrencia de algo significativo praquele subdominio
- no espaço da solução, um evento de dominio é a ocorrencia de uma mudança de estado no servidor
- podem ser transpostos no codigo atraves de classes cujos nomes descrevem o evento. quando acontecer uma mudança de estado, é importante que uma instancia dessa classe seja gerada e encaminhada apropriadamente
- de maneira geral, a instancia de evento deve ser convertida em mensagem para algum mecanismo de mensageria
- outro destino comum para os eventos é a persistencia deles em algum banco (event sourcing)
- o bounded context nao deve consumir um evento produzido por ele proprio
- serve pra integração entre contextos ou para ter um historico das entidades

## Specifications
- é um design pattern que ajuda a manter o SOLID quando se tem regras que mudam com frequencia
- a mecanica do specification recomenda mover a logica de negocio que pode mudar, de dentro da classe que representa a entidade ou value object, para uma classe especifica para essa regra de negocio
- normalmente se usam factories como complemento para as specifications

## Repositorios
- de acordo com Eric Evans, um repositorio seria uma coleção em memoria de todos objetos de um determinado tipo
- a ideia é que se tenha uma implementação que lembra uma coleção para abstrair a visao de que essa coleção esta lidando com um banco de dados
- de acordo com Vaughn Vernon, normalmente se tem um repositorio para cada agregado
- o repositorio funciona com um CRUD e pode conter diversos tipos de consulta, porem o retorno desses metodos de consulta devem ser coleções de objetos do proprio tipo
- outra funcionalidade comum dos repositorios é permitir que um cliente do repositorio recupere uma parte do agregado sem precisar recuperar o objeto raiz do agregado
- o repositorio funciona como uma fronteira entre o modelo de persistencia e o modelo de dominio e é responsavel por impedir que o modelo de persistencia contamine o modelo de dominio com propriedades desnecessarias pro dominio
- uma falha muito comum no uso dos repositorios é mapear para tabelas no banco e nao para agregados, quando isso acontece nao se tem repositorios e sim DAOs

## Domain Services
- domain services != application services
- normalmente quando encontramos muitos serviços de dominio, encontramos tambem implementações de entidades e value objects anemicas
- sao usados para implementar operações que ultrapassam os limites naturais dos agregados envolvidos nas operações
- domain services devem ser stateless
- domain services podem ficar numa camada mais proxima da infraestrutura, e nao exatamente no core do dominio
- nao fazer uso da infraestrutura pra preservar o dominio é errado. nao devemos abrir mao da solução tecnica em função de um mal entendimento do que representa o dominio
- serviços de dominio sao aqueles que carregam alguma regra ou logica de negocio que deveria estar na camada de dominio
- serviços de aplicação apenas orquestram interações com o dominio e nao devem possuir regra de negocio

## Factories
- na implementação de um modelo de dominio, instanciar agregados deve ser simples!
- o construtor garante que o "estado" seja valido desde a criação do objeto, porem em alguns cenarios essa maneira nao é a mais indicada para validar
- uma Factory facilita a instanciação de entidades e VOs
- alguns usos recomendados para factories:
  - tornar o codigo mais legivel em caso de construtores com muitos parametros que sejam do mesmo tipo. isso dificulta o uso e a leitura do codigo (aqui se recomenda usar um factory method)
  - quando se precisa instanciar tipos concretos diferentes dependendo dos parametros recebidos na instanciação (aqui se recomenda usar uma abstract factory)
  - quando a logica de validação pra instanciação dos objetos que compoem o agregado for muito complexa, a ponto que ela precise ser apartada (aqui se recomenda usar um builder)
- factories podem ser muito uteis pra representação do modelo de dominio pois podem deixar o codigo mais expressivo e ajudam a reduzir acoplamento

## Organização do código
- usar modulos pra simplificar o entendimento do codigo
- modulos devem funcionar como blocos coesos de codigo, com pouca ou nenhuma dependencia a codigo externo
- modulos não devem possuir referencia cíclica
- modulos devem usar uma estrutura hierarquica que simplifique a navegação
- o que deve conter cada nivel hierarquico:
  - primeiro nivel: nome da empresa que esta desenvolvendo o software
  - segundo nivel: deixar claro os bounded contexts
  - terceiro nivel: a natureza do codigo (domain, infrastructure, api)
  - quarto nivel (dentro de domain): eventualmente segregar services e model
  - quinto nivel (dentro de model): segregar por modulos indicando agregados
