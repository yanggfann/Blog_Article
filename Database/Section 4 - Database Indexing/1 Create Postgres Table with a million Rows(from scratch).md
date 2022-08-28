docker run -e POSTGRES_PASSWORD=postgres --name pg postgres

docker exec -it pg psql -U postgres create table temp (t int);

insert into temp (t) select random()*100 from generate_series(0,100000);
