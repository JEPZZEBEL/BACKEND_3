Banco XYZ - Backend for Frontend (BFF)
Objetivo

Implementar el patrón Backend for Frontend (BFF) para el Banco XYZ, creando backends independientes y optimizados para cada canal (Web, Móvil y Cajero Automático) que consumen datos simulados de un sistema legacy (data-mock).
Esto permite:

Reducir consumo de recursos en móviles.

Entregar vistas completas en web.

Asegurar operaciones críticas en cajeros.

📂 Estructura del Proyecto
bff-banco-xyz-parent/
│── common-lib/     → DTOs y clases compartidas
│── data-mock/      → Mock de datos legacy (puerto 8090)
│── bff-web/        → BFF Web (puerto 8081)
│── bff-mobile/     → BFF Mobile (puerto 8082)
│── bff-atm/        → BFF ATM (puerto 8083)
│── reverse-proxy/  → Nginx como proxy inverso (puerto 80/443)
│── docker-compose.yml

🔐 Seguridad

Cada BFF implementa autenticación HTTP Basic con usuario = nombre del módulo y contraseña = bff123.

Canal	Usuario	Password
Web	web	bff123
Mobile	mobile	bff123
ATM	atm	bff123

El reverse proxy Nginx soporta HTTPS.
Ejemplo de configuración:

server {
  listen 443 ssl http2;
  server_name banco-xyz.local;

  ssl_certificate     /etc/nginx/ssl/banco.crt;
  ssl_certificate_key /etc/nginx/ssl/banco.key;

  location /web/ {
    proxy_pass http://bff-web:8081/;
  }
  location /mobile/ {
    proxy_pass http://bff-mobile:8082/;
  }
  location /atm/ {
    proxy_pass http://bff-atm:8083/;
  }
}


▶️ Ejecución
Requisitos

JDK 17

Maven 3.9+

Docker y Docker Compose

Construcción
# Compilar sin tests
mvn -DskipTests clean package

Levantar stack completo
docker compose up -d --build

Verificar estado
docker compose ps


Debe aparecer:

reverse-proxy → Up

bff-web, bff-mobile, bff-atm, data-mock → Up

🌐 Endpoints
BFF Web (http://localhost/api/web/
*)

/api/web/customers → listado completo de clientes

/api/web/accounts → detalle completo de cuentas

Ejemplo:

curl -u web:bff123 http://localhost/api/web/customers

BFF Mobile (http://localhost/api/mobile/
*)

/api/mobile/customers → clientes resumidos (nombre + id)

/api/mobile/accounts → cuentas resumidas (saldo, tipo)

Ejemplo:

curl -u mobile:bff123 http://localhost/api/mobile/accounts

BFF ATM (http://localhost/api/atm/
*)

/api/atm/balance/{id} → consulta de saldo segura

/api/atm/withdraw → simula retiro

Ejemplo:

curl -u atm:bff123 http://localhost/api/atm/balance/1

Data Mock (http://localhost:8090/mock/
*)

/mock/customers

/mock/accounts

🧾 Evidencias de Ejecución

(Adjuntar capturas en la carpeta /evidencias)

✔️ docker compose ps con todos los contenedores en estado Up.

✔️ Respuesta JSON de cada BFF desde Postman o cURL.

✔️ Captura del navegador accediendo a https://localhost/web/api/... con certificado cargado.

📈 Arquitectura

Descripción breve:

Los frontends consumen APIs de su BFF.

Cada BFF aplica autenticación y adapta la respuesta.

Los BFFs consultan a data-mock.

Todo el tráfico pasa por Nginx reverse proxy con HTTPS.

⚠️ Troubleshooting

Si algún servicio aparece en Restarting:

docker compose logs <servicio>


Verificar que los puertos 8081–8083 y 8090 no estén ocupados.

Revisar certificados si el navegador bloquea la conexión HTTPS.
