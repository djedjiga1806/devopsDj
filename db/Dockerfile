FROM postgres:14.1-alpine

ENV POSTGRES_DB=mydb \
   POSTGRES_USER=postgres \
   POSTGRES_PASSWORD=postgres

COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/

EXPOSE 5432

CMD ["postgres"]