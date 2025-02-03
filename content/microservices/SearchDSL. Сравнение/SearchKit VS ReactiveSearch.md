![[Pasted image 20250203123145.png]]

![[Pasted image 20250203123259.png]]

![[Pasted image 20250203123333.png]]

![[Pasted image 20250203123353.png]]

![[Pasted image 20250203123414.png]]

![[Pasted image 20250203123430.png]]


| Функция                    | API Searchkit                                                                                                                                                                                                                                                                                      | API ReactiveSearch                                                                                                                                                        |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Интерфейс запроса**      | GraphQL API                                                                                                                                                                                                                                                                                        | REST API (похожие на Elasticsearch DSL)                                                                                                                                   |
| **Кастомизация**           | Логика запросов определяется в конфигурации на сервере                                                                                                                                                                                                                                             | Логика запросов настраивается через REST API и компоненты на фронтенде                                                                                                    |
| **Безопасность**           | Работает как промежуточный слой (GraphQL-эндпоинт защищает Elasticsearch). Вместо отправки запроса напрямую в Elasticsearch API, вы отправляете **GraphQL запрос** (с параметрами поиска в виде переменных) в Searchkit API. Searchkit затем **переводит** этот GraphQL запрос в Elasticsearch DSL | Работает как промежуточный слой (REST-эндпоинт защищает Elasticsearch).  Запрос DSL без дополнительных настроек,  аналогичен тому, что использовался бы для Elasticsearch |
| **Простота использования** | Упрощает использование Elasticsearch с помощью GraphQL (требуется знание в команде)                                                                                                                                                                                                                | Запросы аналогичны стандартным для Elasticsearch API, но позволяет при передаче/получении данных в Elastic встроить дополнительную логику                                 |
| **Сложность настройки**    | Средняя (требуется настройка сервера для GraphQL)                                                                                                                                                                                                                                                  | Средняя (требуется настройка сервера для API)                                                                                                                             |
| **Документация**           | [Документация Searchkit](https://searchkit.co/docs/)                                                                                                                                                                                                                                               | [Документация ReactiveSearch](https://docs.appbase.io/docs/reactivesearch/v3/overview/quickstart/)                                                                        |

# Архитектура Searchkit

![[Pasted image 20250203203718.png]]

# Как делать запросы к API Searchkit и ReactiveSearch

---
## 1. Пример запроса Searchkit (GraphQL)

### Конфигурация backend для Searchkit



```js
import { createSearchkitConfig, SearchkitSchema } from "@searchkit/schema";
import { ApolloServer } from "apollo-server";

const searchkitConfig = createSearchkitConfig({
  host: "http://localhost:9200", // Ваш Elasticsearch-эндпоинт
  index: "ecommerce",            // Имя индекса Elasticsearch
  hits: {
    fields: ["name", "brand.name", "categories.name", "picture"],
  },
  query: {
    fields: ["name", "full_name", "brand.name"],
  },
  facets: [
    { field: "brand.name.keyword", label: "Brand", type: "terms" },
    { field: "categories.name.keyword", label: "Category", type: "terms" },
  ],
});

const server = new ApolloServer({
  schema: SearchkitSchema(searchkitConfig),
});

server.listen().then(({ url }) => {
  console.log(`🚀 Сервер готов на ${url}`);
});


```

### Пример GraphQL запроса


```graphql
query SearchProducts($query: String, $filters: [SearchkitFilterSet]) {
  results(query: $query, filters: $filters) {
    hits {
      items {
        id
        fields {
          name
          brand {
            name
          }
          categories {
            name
          }
          picture
        }
      }
    }
    facets {
      id
      label
      entries {
        label
        count
        id
        isSelected
      }
    }
  }
}

```

### Пример переменных запроса

```json
{
  "query": "Electrolux",
  "filters": [
    {
      "id": "brand.name.keyword",
      "value": "Electrolux"
    }
  ]
}

```

```json
{
  "query": "query SearchProducts($query: String, $filters: [SearchkitFilterSet]) { ... }",
  "variables": {
    "query": "Electrolux",
    "filters": [{ "id": "brand.name.keyword", "value": "Electrolux" }]
  }
}

```

### Пример ответа

```json
{
  "data": {
    "results": {
      "hits": {
        "items": [
          {
            "id": "32c03957-3738-45a0-b5d5-34cba1cedd6c",
            "fields": {
              "name": "Тепловентилятор Electrolux EFH/C-5115",
              "brand": {
                "name": "Electrolux"
              },
              "categories": [
                {
                  "name": "Тепловентиляторы"
                }
              ],
              "picture": "https://rkcdn.ru/products/ba10f5f4-7c4a-11e7-9ead-ac162d7b6f40/thumb_big.jpg"
            }
          }
        ]
      },
      "facets": [
        {
          "id": "brand.name.keyword",
          "label": "Brand",
          "entries": [
            {
              "label": "Electrolux",
              "count": 100,
              "id": "Electrolux",
              "isSelected": true
            }
          ]
        }
      ]
    }
  }
}

```

### Пример фронтенда (React с Apollo Client)

Ниже представлен простой компонент React, который выполняет запрос к GraphQL API Searchkit с использованием Apollo Client:

```jsx
import { ApolloClient, InMemoryCache, gql, useQuery } from "@apollo/client";

const client = new ApolloClient({
  uri: "http://localhost:4000/graphql", // Ваш GraphQL эндпоинт Searchkit
  cache: new InMemoryCache(),
});

const SEARCH_QUERY = gql`
  query SearchProducts($query: String) {
    results(query: $query) {
      hits {
        items {
          id
          fields {
            name
            brand {
              name
            }
            picture
          }
        }
      }
    }
  }
`;

function ProductSearch({ searchQuery }) {
  const { loading, error, data } = useQuery(SEARCH_QUERY, {
    variables: { query: searchQuery },
    client,
  });

  if (loading) return <p>Загрузка...</p>;
  if (error) return <p>Ошибка: {error.message}</p>;

  return (
    <div>
      {data.results.hits.items.map((item) => (
        <div key={item.id}>
          <h2>{item.fields.name}</h2>
          <p>{item.fields.brand.name}</p>
          <img src={item.fields.picture} alt={item.fields.name} />
        </div>
      ))}
    </div>
  );
}

export default ProductSearch;

```

---

## 2. Пример запроса API ReactiveSearch (REST)

### Настройка API ReactiveSearch

настройка для сервера на Node.js с использованием **ReactiveSearch API**:

```js
const express = require("express");
const { ReactiveSearchApi } = require("@appbaseio/reactivesearch-api");

const app = express();
const api = new ReactiveSearchApi({
  host: "http://localhost:9200",    // Ваш Elasticsearch эндпоинт
  credentials: "username:password", // Опционально: учетные данные для Elasticsearch
  enableTelemetry: false,
});

// Монтируем API ReactiveSearch на маршрут, например, `/api`
app.use("/api", api.init());

app.listen(4001, () => {
  console.log("ReactiveSearch API работает на http://localhost:4001/api");
});

```

### Пример REST запроса

Теперь вы можете отправить POST-запросы на `http://localhost:4001/api/ecommerce/_search`. Например:

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "Electrolux",
            "fields": ["name", "full_name", "brand.name"]
          }
        }
      ],
      "filter": [
        {
          "term": { "brand.name.keyword": "Electrolux" }
        }
      ]
    }
  }
}

```

### Пример ответа

```json
{
  "took": 12,
  "timed_out": false,
  "hits": {
    "total": 1,
    "max_score": 1.0,
    "hits": [
      {
        "_index": "ecommerce",
        "_id": "32c03957-3738-45a0-b5d5-34cba1cedd6c",
        "_score": 1.0,
        "_source": {
          "name": "Тепловентилятор Electrolux EFH/C-5115",
          "brand": { "name": "Electrolux" },
          "categories": [{ "name": "Тепловентиляторы" }],
          "picture": "https://rkcdn.ru/products/ba10f5f4-thumb_big.jpg"
        }
      }
    ]
  }
}

```

### Пример фронтенда

Ниже представлен простой компонент React, который выполняет запрос к API ReactiveSearch с использованием Axios:

```jsx
import React, { useState, useEffect } from "react";
import axios from "axios";

function ProductSearch({ searchQuery }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    axios
      .post("http://localhost:4001/api/ecommerce/_search", {
        query: {
          bool: {
            must: [
              {
                multi_match: {
                  query: searchQuery,
                  fields: ["name", "full_name", "brand.name"],
                },
              },
            ],
          },
        },
      })
      .then((response) => {
        setResults(response.data.hits.hits);
      })
      .catch((error) => {
        console.error("Ошибка при получении данных", error);
      });
  }, [searchQuery]);

  return (
    <div>
      {results.map((result) => (
        <div key={result._id}>
          <h2>{result._source.name}</h2>
          <p>{result._source.brand.name}</p>
          <img src={result._source.picture} alt={result._source.name} />
        </div>
      ))}
    </div>
  );
}

export default ProductSearch;

```

---

## 3. Резюме

1. **GraphQL vs. DSL**
    
    - **ReactiveSearch** может принимать **сырой Elasticsearch DSL** и передавать его в Elasticsearch—фактически действуя как простой прокси.
    - **Searchkit**, с другой стороны, предоставляет **GraphQL-эндпоинт**. Вместо отправки Elasticsearch DSL напрямую в ES, вы отправляете **GraphQL запрос** (с вашими параметрами поиска как переменными). Searchkit затем **переводит** этот GraphQL запрос в Elasticsearch DSL.
2. **Серверная конфигурация**
    
    - В **Searchkit** вы определяете, как запросы и фильтры должны маппится на поля Elasticsearch в вашей **конфигурации Searchkit**.
    - Когда приходит запрос GraphQL, Searchkit использует эту конфигурацию для построения реального DSL запроса. Обычно вы **не** передаете сырой DSL с фронтенда; перевод из GraphQL → DSL происходит внутри.
3. **Дополнительные функции**
    
    - **Searchkit** может автоматически обрабатывать фасеты, пагинацию, многоуровневые фильтры и другие функции, удобные для электронной коммерции. Эти функции настраиваются на сервере Searchkit.
    - GraphQL API может возвращать уже структурированные результаты поиска и данные фасетов на ваш фронтенд.
4. **Отсутствие прямого эндпоинта DSL**
    
    - **Searchkit** не предоставляет эндпоинт для запросов DSL. Если вы хотите передавать DSL напрямую, вам нужно будет написать кастомный серверный слой, который обходит или расширяет стандартную обработку GraphQL Searchkit.

---

- **Обе** системы, ReactiveSearch и Searchkit, **находятся между** вашим фронтендом и Elasticsearch.
- **API ReactiveSearch** может работать как **прокси для передачи** Elasticsearch DSL (с дополнительными функциями).
- **Searchkit** представляет собой **GraphQL абстракцию**. Вы отправляете GraphQL запросы, и он автоматически переводит их в DSL — таким образом, вы не видите и не отправляете Elasticsearch DSL с фронтенда.