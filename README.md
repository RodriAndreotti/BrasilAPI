<h1 align="center"><img src="./public/brasilapi-logo-small.png"> Brasil API</h1>

<div align="center">
  <p>
    <strong>Vamos transformar o Brasil em uma API?</strong>
  </p>
  <p>
    <a href="https://vercel.com/?utm_source=brasilapi" target="_blank" rel="noopener">
      <img src="./public/powered-by-vercel.svg" width="175" alt="Powered by Vercel" />
    </a>
  </p>
</div>

<div align="center">
  <img src="https://github.com/BrasilAPI/BrasilAPI/workflows/Testes%20E2E/badge.svg">
</div>

## Motivo
Acesso programático de informações é algo fundamental na comunicação entre sistemas mas, para nossa surpresa, uma informação tão útil e pública quanto um CEP não consegue ser acessada diretamente por um navegador por conta da API dos Correios não possuir CORS habilitado.

Dado a isso, este projeto experimental tem como objetivo centralizar e disponibilizar endpoints modernos com baixíssima latência utilizando tecnologias como [Vercel Smart CDN](https://vercel.com/smart-cdn/?utm_source=brasilapi) responsável por fazer o cache das informações em atualmente 23 regiões distribuídas ao longo do mundo (incluindo Brasil). Então não importa o quão devagar for a fonte dos dados, nós queremos disponibilizá-la da forma mais rápida e moderna possível.

## Como contribuir
Através do [Next.js](https://nextjs.org/?utm_source=brasilapi), um framework utilizado por empresas como Marvel, Twitch, Nike, Hulu, TypeForm, Nubank, Ferrari, TikTok, Square Enix, entre outras, estamos construindo a página de apresentação do projeto e, por ser um framework híbrido, ele possibilita a construção e deploy de APIs com o mínimo de configuração possível em uma infraestrutura autoescalável da [Vercel](https://vercel.com/?utm_source=brasilapi), a mesma que conta com recursos sensacionais como a [Vercel Smart CDN](https://vercel.co/smart-cdn/?utm_source=brasilapi).

Caso você esteja lendo esta versão de README, você está pegando o projeto num estágio extremamente inicial, porém empolgante, pois há várias coisas a serem definidas. Então caso queira contribuir, utilize as issues para entender quais pontos ainda não foram resolvidos, conversar conosco e contribuir tanto com idéias técnicas, quanto de quais APIs podem ser criadas.

Veja mais detalhes sobre **Como contribuir** no arquivo [CONTRIBUTING.md](CONTRIBUTING.md)


## Endpoints
O primeiro endpoint a ser implementado precisava ser o que estava nos dando a maior dor de cabeça: busca de um endereço através do CEP. É um endpoint extremamente simples de implementar, mas vários detalhes ainda não foram resolvidos, como garantir seu comportamento através de testes E2E utilizando a Preview URL que a Vercel retorna a cada Pull Request. Depois de consolidarmos as melhores práticas para esse endpoint, poderemos replicar para todos os outros que irão vir.

### CEP
Busca por CEP com múltiplos providers de fallback.

**GET** `https://brasilapi.com.br/api/cep/v1/`**[cep]**

#### Consulta com sucesso

```json
// GET https://brasilapi.com.br/api/cep/v1/05010000

{
  "cep": "05010000",
  "state": "SP",
  "city": "São Paulo",
  "neighborhood": "Perdizes",
  "street": "Rua Caiubi"
}
```

#### Consulta com erro

```json
// GET https://brasilapi.com.br/api/cep/v1/00000000

{
  "name": "CepPromiseError",
  "message": "Todos os serviços de CEP retornaram erro.",
  "type": "service_error",
  "errors": [
    {
      "name": "ServiceError",
      "message": "CEP INVÁLIDO",
      "service": "correios"
    },
    {
      "name": "ServiceError",
      "message": "CEP não encontrado na base do ViaCEP.",
      "service": "viacep"
    }
  ]
}
```

### Banks
Busca por dados dos bancos brasileiros direto na base de dados do Bacen.

**GET** `https://brasilapi.com.br/api/banks/v1/`**[code]**

#### Consulta com sucesso

```json
// GET https://brasilapi.com.br/api/banks/v1/260

{
  "ispb": "18236120",
  "name": "NU PAGAMENTOS S.A.",
  "code": 260,
  "fullName": "Nu Pagamentos S.A."
}
```

#### Consulta com código incorreto

```json
// GET https://brasilapi.com.br/api/banks/v1/1111111

{
  "message": "Código bancário não encontrado",
  "type": "BANK_CODE_NOT_FOUND"
}
```

**GET** `https://brasilapi.com.br/api/banks/v1`

#### Consulta com sucesso

```json
// GET https://brasilapi.com.br/api/banks/v1

[
  {
    "ispb": "18236120",
    "name": "NU PAGAMENTOS S.A.",
    "code": 260,
    "fullName": "Nu Pagamentos S.A."
  },
  ...
]
```

### Serviços CPTEC 
Este é um conjunto de rotas que visa abstrair o acesso aos dados do portal [CPTEC/INPE](http://servicos.cptec.inpe.br/XML/), os endpoints disponíveis estão à seguir:

**GET** `https://brasilapi.com.br/api/cptec/v1/cities/[:name]` 

Retorna a listagem de cidades com os respectivos códigos internos da CPTEC, este código é necessário para o funcionamento de alguns dos próximos endpoints.
`[:name]` é opcional, quando informado retorna uma lista de cidades que correspondam ao termo informado. Por exemplo: 


**GET** `https://brasilapi.com.br/api/cptec/v1/cities/Salvador`

**Exemplo de retorno:**

``` json
  [
   {
      "name":"Salvador",
      "state":"BA",
      "code":"242"
   },
   {
      "name":"Salvador das Missões",
      "state":"RS",
      "code":"4505"
   },
   {
      "name":"Salvador do Sul",
      "state":"RS",
      "code":"4506"
   }
]
```


**GET** `https://brasilapi.com.br/api/cptec/v1/weather/airport/:icaoCode` 

Obtém as condições climáticas atuais no aeroporto informado

**Exemplo de retorno:**

```json
{
   "icao_code":"SBSP",
   "last_update":"17/12/2020 17:00:00",
   "pressure":"1014",
   "visibility":">10000",
   "wind":"25",
   "wind_direction":"230",
   "humidity":"58",
   "weather_code":"ps",
   "weather_desc":"Predomínio de Sol",
   "temp":"28"
}
```
**GET** /api/cptec/v1/weather/capital

Obtém uma lista com todas as capitais do Brasil juntamente com suas condições climáticas atuais.

**Exemplo de retorno:**
``` json
[
  {
    "icao_code": "SBAR",
    "last_update": "2021-01-19T20:00:00.188Z",
    "pressure": "1008",
    "visibility": ">10000",
    "wind": "54",
    "wind_direction": "80",
    "humidity": "71",
    "weather_code": "ps",
    "weather_desc": "Predomínio de Sol",
    "temp": "28"
  },
  ...
] 
```
**GET** `/api/cptec/v1/weather/prediction/:cityCode[/:days]`

Obtém a previsão do tempo fornecida pelo CPTEC para a quantidade de dias informada (máximo 14 dias).
Este endpoint encapsula 3 endpoints distintos do CPTEC que traziam as informações separadamente (*cidade/codigo_da_localidade/previsao.xml*, *cidade/7dias/codigo_da_localidade/previsao.xml* e *cidade/codigo_da_localidade/estendida.xml*) de forma a uniformizar a forma de acesso e tratar de forma mais efetiva os resultados.

`[/:days] ` É opicional, quando omitido retorna a previsão para um dia

**Exemplo de retorno:**
``` json
{
  "city_name": "Brejo Alegre",
  "state": "SP",
  "last_update": "2020-12-19",
  "weather": [
    {
      "date": "2020-12-20",
      "condition": "pt",
      "min": "23",
      "max": "33",
      "uv_index": "13.0",
      "condition_desc": "Pancadas de Chuva a Tarde"
    },
    {
      "date": "2020-12-21",
      "condition": "pt",
      "min": "23",
      "max": "35",
      "uv_index": "13.0",
      "condition_desc": "Pancadas de Chuva a Tarde"
    }
  ]
}
```
**GET** `/api/cptec/v1/swell/:cityCode[/:days]`

Esta rota retorna informações referentes as ondas nas cidades costeiras do Brasil, utilizando os XMLs de Ondas do CPTEC.
Assim como o endpoint de previsão meteorológica, este agrega dois xmls do serviço da CPTEC (*cidade/codigo_da_localidade/dia/dia/ondas.xml* e *cidade/codigo_da_localidade/todos/tempos/ondas.xml*)
Novamente foi realizado um tratamento para retorno mais efetivo e organizado das informações.

**Atenção:** Este endpoint **NÃO** é de Tábua das Marés, este serviço é fornecido pela Marinha do Brasil, e não pela CPTEC. São serviços diferentes, com informações diferentes. O serviço da CPTEC é focado em altitude de ondas e direção dos ventos e ondas.

`[/:days] ` É opicional, quando omitido retorna a previsão para um dia

**Exemplo de retorno:**
``` json
{
  "city_name": "Rio de Janeiro",
  "state": "RJ",
  "last_update": "2020-12-19",
  "swell": [
    {
      "date": "19-12-2020",
      "swell_data": [
        {
          "wind": "0.8",
          "wind_direction": "W",
          "wave_height": "1.2",
          "wave_direction": "E",
          "agitation": "Fraco",
          "hour": "00h Z"
        },
        {
          "wind": "2.1",
          "wind_direction": "NNE",
          "wave_height": "1.2",
          "wave_direction": "E",
          "agitation": "Fraco",
          "hour": "03h Z"
        },
        {
          "wind": "1.4",
          "wind_direction": "W",
          "wave_height": "1.1",
          "wave_direction": "E",
          "agitation": "Fraco",
          "hour": "06h Z"
        },
        {
          "wind": "1.1",
          "wind_direction": "SW",
          "wave_height": "1.1",
          "wave_direction": "E",
          "agitation": "Fraco",
          "hour": "09h Z"
        },
        {
          "wind": "1.9",
          "wind_direction": "S",
          "wave_height": "1.1",
          "wave_direction": "E",
          "agitation": "Fraco",
          "hour": "12h Z"
        },
        {
          "wind": "2.7",
          "wind_direction": "SE",
          "wave_height": "1.0",
          "wave_direction": "E",
          "agitation": "Fraco",
          "hour": "15h Z"
        },
        {
          "wind": "3.8",
          "wind_direction": "SE",
          "wave_height": "1.0",
          "wave_direction": "E",
          "agitation": "Fraco",
          "hour": "18h Z"
        },
        {
          "wind": "3.9",
          "wind_direction": "E",
          "wave_height": "1.0",
          "wave_direction": "E",
          "agitation": "Fraco",
          "hour": "21h Z"
        }
      ]
    },
    ...
  ]
}
```
Para maiores detalhes sobre o uso da API consulte a nossa [documentação](https://brasilapi.com.br/docs). 


## Termos de Uso
O BrasilAPI é uma iniciativa feita de brasileiros para brasileiros, por favor, não abuse deste serviço. Estamos em beta e ainda elaborando os Termos de Uso, mas por enquanto por favor não utilize formas automatizadas para fazer "crawling" dos dados da API. Um exemplo prático disto é um dos maiores provedores de telefonia do Brasil estar revalidando, neste exato momento, todos os Ceps (de `00000000` até `99999999`) e estourando em 5 vezes o limite atual da nossa conta no servidor. O volume de consulta dever ter a natureza de uma pessoa real requisitando um determinado dado. E para consultas com um alto volume automatizado, iremos mais para frente fornecer alguma solução, como por exemplo, conseguir fazer o download de toda a base de Ceps em uma única request.

## Contribuidores

| [<img src="https://avatars0.githubusercontent.com/u/22279592?s=400&v=4" width="115"><br><sub>@kevenleone</sub>](https://github.com/kevenleone) | [<img src="https://avatars0.githubusercontent.com/u/29285724?s=400&v=4" width="115"><br><sub>@OtavioCapila</sub>](https://github.com/OtavioCapila) | [<img src="https://avatars2.githubusercontent.com/u/6341210?s=400&v=4" width="115"><br><sub>@rafamancan</sub>](https://github.com/rafamancan) | [<img src="https://avatars2.githubusercontent.com/u/22918282?s=400&v=4" width="115"><br><sub>@lucas-eduardo</sub>](https://github.com/lucas-eduardo) | [<img src="https://avatars1.githubusercontent.com/u/640840?s=400&v=4" width="115"><br><sub>@eliseumds</sub>](https://github.com/eliseumds) | [<img src="https://avatars1.githubusercontent.com/u/11640028?s=400&v=4" width="115"><br><sub>@evertoncastro</sub>](https://github.com/evertoncastro) |
| :---: |  :---: |  :---: |  :---: |  :---: |  :---: |
| [<img src="https://avatars3.githubusercontent.com/u/13923364?s=400&v=4" width="115"><br><sub>@mukaschultze</sub>](https://github.com/mukaschultze) | [<img src="https://avatars2.githubusercontent.com/u/7690649?s=400&v=4" width="115"><br><sub>@paulogdm</sub>](https://github.com/paulogdm) | [<img src="https://avatars2.githubusercontent.com/u/34130446?s=400&u=ce853ec1d505c15b78ffa7d64a4c2a419f9dfdf8&v=4" width="115"><br><sub>@mathleite</sub>](https://github.com/mathleite) |  [<img src="https://avatars0.githubusercontent.com/u/19312651?s=400&u=38b984e80c3c6a59fee61676c504f02313e2212d&v=4" width="115"><br><sub>@WeslleyNasRocha</sub>](https://github.com/WeslleyNasRocha) | [<img src="https://avatars2.githubusercontent.com/u/7424845?s=400&u=346acdf662dbb880ecf659ce27097f5c13bd9dc3&v=4" width="115"><br><sub>@paulo-santana</sub>](https://github.com/paulo-santana) | [<img src="https://avatars2.githubusercontent.com/u/41276009?s=400&u=109f02852994de760c8f0014ef556cafad7429a1&v=4" width="115"><br><sub>@RaphaelOliveiraMoura</sub>](https://github.com/RaphaelOliveiraMoura) |
| [<img src="https://avatars3.githubusercontent.com/u/49703106?s=400&u=364f6affc0b28fee107bc7542a9408fa70da2208&v=4" width="115"><br><sub>@guiaramos</sub>](https://github.com/guiaramos) | [<img src="https://avatars2.githubusercontent.com/u/12580906?s=400&u=42e3f0ae2caa642f687945d4e2091ed8a5d97125&v=4" width="115"><br><sub>@marceloF5</sub>](https://github.com/marceloF5) | [<img src="https://avatars3.githubusercontent.com/u/15824865?s=400&u=dc038f866810c31c8d70f624bd53ca8cb9061d4b&v=4" width="115"><br><sub>@tupizz</sub>](https://github.com/tupizz) | [<img src="https://avatars1.githubusercontent.com/u/4183681?s=400&u=bd588248b3081057433881db40ebaf176cd37211&v=4" width="115"><br><sub>@FlavioAndre</sub>](https://github.com/FlavioAndre) | [<img src="https://avatars2.githubusercontent.com/u/5989971?s=460&u=59378fbe4a020238916b281ebfb72992e921e0c7&v=4" width="115"><br><sub>@matheusvellone</sub>](https://github.com/matheusvellone) | [<img src="https://avatars2.githubusercontent.com/u/11930738?s=400&u=a1270129441990069d997df8e39b6f6224bafd40&v=4" width="115"><br><sub>@danielramosbh74</sub>](https://github.com/danielramosbh74) |
| [<img src="https://avatars3.githubusercontent.com/u/34171773?s=400&u=f05580bc09aaa70b440d7f7f5f8eb3e4f693ec5b&v=4" width="115"><br><sub>@juniorpb</sub>](https://github.com/juniorpb) | [<img src="https://avatars2.githubusercontent.com/u/38855507?s=400&u=20c80252e57c06227186be9761e67a20a82d3717&v=4" width="115"><br><sub>@CarlosZiegler</sub>](https://github.com/CarlosZiegler) | [<img src="https://avatars1.githubusercontent.com/u/370216?s=400&u=7f3ad2ea903d45bbfc4434902a1d081740e9f5dc&v=4" width="115"><br><sub>@otaciliolacerda</sub>](https://github.com/otaciliolacerda) | [<img src="https://avatars0.githubusercontent.com/u/9449048?s=400&u=85262ea3dc049b1f4a8e63e3b574b5fc03327b52&v=4" width="115"><br><sub>@RodriAndreotti</sub>](https://github.com/RodriAndreotti) |

## Autores

| [<img src="https://avatars3.githubusercontent.com/u/4248081?s=460&v=4" width=115><br><sub>@filipedeschamps</sub>](https://github.com/filipedeschamps) | [<img src="https://avatars3.githubusercontent.com/u/8251208?s=400&v=4" width=115><br><sub>@lucianopf</sub>](https://github.com/lucianopf) |
| :---: | :---: |
