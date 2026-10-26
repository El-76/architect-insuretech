### Соответствие запросов REST и GraphQL

#### Получение дааных клиента

```
GET /clients/123
```

```
query {
  client(id: "123") {
    id
    name
    age
  }
}
```

#### Получение документов клиента

```
GET /clients/123/documents
```

```
query {
  client(id: "123") {
    documents {
      id
      type
      number
      issueDate
      expiryDate
    }
  }
}
```

#### Получение родственников клиента

```
GET /clients/123/relatives
```

```
query {
  client(id: "123") {
    relatives {
      id
      relationType
      name
      age
    }
  }
}
```