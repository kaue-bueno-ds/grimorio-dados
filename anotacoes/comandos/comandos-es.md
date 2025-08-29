## Criação de index no ES
```
PUT /nome-do-indice
{
  "mappings": {
    "properties": {
      "nome":      { "type": "text" },
      "idade":     { "type": "integer" },
      "ativo":     { "type": "boolean" },
      "nascimento":{ "type": "date" }
    }
  }
}
```

## Formatos de tipos no Elasticsearch

Os principais tipos são:

🔡 Strings:

```"text"```: Para busca full-text (usa analisadores).

```"keyword"```: Para dados exatos (códigos, categorias, tags).

🔢 Numéricos:

```"integer"```: até ±2 bilhões.

```"long"```: até ±9 quintilhões.

```"short"``` / ```"byte"```: economizam espaço.

```"float"``` / ```"double"```: para valores decimais.

📅 Datas:

```"date"```: aceita vários formatos (```"strict_date_optional_time"```, ```"epoch_millis"```, etc.).

{ ```"type"```: ```"date"```, ```"format"```: ```"yyyy-MM-dd HH:mm:ss||epoch_millis"```}

🧠 Booleanos:

```"boolean"```

🗺️ Geoespacial:

```"geo_point"```: latitude/longitude.

```"geo_shape"```: formas complexas.

🧱 Objetos:

```"object"```: JSON aninhado.

```"nested"```: para arrays de objetos com buscas relacionais.