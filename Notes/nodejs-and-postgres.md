# NodeJS with Postgres SQL

## Config docker

```yaml
postgresql:
    image: 'postgres:9.5.5'
    environment:
      - POSTGRES_USER=atoll
      - POSTGRES_PASSWORD=atoll
      - POSTGRES_DB=atoll
    ports:
      - '12400:5432'
    logging:
      driver: "none"

  postgresql_test:
    image: 'postgres:9.5.5'
    environment:
      - POSTGRES_USER=atoll
      - POSTGRES_PASSWORD=atoll
      - POSTGRES_DB=atoll_test
```

## Dependencies

```yaml
"pg-promise": "5.9.7"
"loopback-connector-postgresql": "3.0.1"
"db-migrate": "0.10.0-beta.21"
"db-migrate-pg": "0.1.11"

```

## References

[https://www.postgresql.org/docs/9.5/static/index.html](https://www.postgresql.org/docs/9.5/static/index.html)

[https://blog.codeship.com/unleash-the-power-of-storing-json-in-postgres/](https://blog.codeship.com/unleash-the-power-of-storing-json-in-postgres)

[http://vitaly-t.github.io/pg-promise/](http://vitaly-t.github.io/pg-promise)

[https://node-postgres.com/](https://node-postgres.com/)A

## Add builder

```ts
function aCover(coverShape: Partial<Cover>): Cover {
  const defaultCover = {
    id: uuid.v4(),
    type: REPATRIATION_COVER_TYPE,
    locations: [WHOLE_WORLD],
    limits: [],
    deductible: [],
    // other required properties
  };
  return {
    ...defaultCover,
    ...coverShape,
  };
};

const medicalExpensesCover = aCover({
  type: MEDICAL_EXPENSES_COVER_TYPE,
  limits: [{
    value: 11000,
    currency: 'EUR',
  }],
});
```
