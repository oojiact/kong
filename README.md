# kong


docker run -d --name kong-database --network=kong-net \ -p 5432:5432
                 \ 
                -e “POSTGRES_USER=kong” \ 
                -e “POSTGRES_DB=kong” \ 
                postgres:9.6


export KONG_LICENSE_DATA='{"license":"version":1}}'





docker run -d --name kong-ee-database \
   -p 5432:5432 \
   -e "POSTGRES_USER=kong" \
   -e "POSTGRES_DB=kong" \
   postgres:9.5



docker run --rm --link kong-ee-database:kong-ee-database \
   -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-ee-database" \
   -e "KONG_CASSANDRA_CONTACT_POINTS=kong-ee-database" \
   -e "KONG_LICENSE_DATA=$KONG_LICENSE_DATA" \
   kong-ee kong migrations up



docker run -d --name kong-ee --link kong-ee-database:kong-ee-database \
  -e "KONG_DATABASE=postgres" \
  -e "KONG_PG_HOST=kong-ee-database" \
  -e "KONG_CASSANDRA_CONTACT_POINTS=kong-ee-database" \
  -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
  -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
  -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
  -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
  -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
  -e "KONG_PORTAL=on" \
  -e "KONG_PORTAL_GUI_PROTOCOL=http" \
  -e "KONG_PORTAL_GUI_HOST=127.0.0.1:8003" \
  -e "KONG_LICENSE_DATA=$KONG_LICENSE_DATA" \
  -p 8000:8000 \
  -p 8443:8443 \
  -p 8001:8001 \
  -p 8444:8444 \
  -p 8002:8002 \
  -p 8445:8445 \
  -p 8003:8003 \
  -p 8004:8004 \
  kong-ee



curl -X POST http://localhost:8001/rbac/users/ -d name=admin -d user_token=12345 

curl -X POST http://localhost:8001/rbac/users/admin/roles -d roles=super-admin 

docker exec -it kong-ee /bin/sh

KONG_ENFORCE_RBAC=on kong reload

curl -X GET http://localhost:8001/status -H "Kong-Admin-Token: 12345" 
