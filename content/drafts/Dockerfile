FROM hasura/graphql-engine

ENV HASURA_GRAPHQL_ENABLE_CONSOLE="true"

CMD graphql-engine \
    --database-url postgres://postgres:eamcet6759@unix(/cloudsql/hasuraa:asia-south1:hasura-postgres)/postgres \
    serve \
    --server-port $PORT \
