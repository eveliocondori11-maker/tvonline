# CORPATELEREDES CRM PRO — Ubuntu VPS

CRM inicial funcional para ISP/IPTV: clientes y subclientes, planes, facturas mensuales, pagos, cuentas por cobrar, tickets, catálogo IPTV, DNS, routers MikroTik RouterOS v7 mediante REST HTTPS, importación de usuarios PPPoE y acciones de suspensión/reactivación.

## Requisitos
- Ubuntu 22.04/24.04, Docker Engine y Docker Compose plugin.
- Para HTTPS público: dominio apuntando al VPS y puertos TCP 80/443 abiertos.
- MikroTik: RouterOS v7 con REST habilitado sobre HTTPS. El soporte depende del modelo y versión RouterOS; esta entrega no incluye el API binario legacy de RouterOS v6.
- Cobro QR automático: contrato/cuenta de comercio y credenciales de un proveedor boliviano que ofrezca API de creación de cobros y webhook firmado. El CRM incluye adaptador HTTP genérico, no credenciales ni integración certificada con un banco específico.

## Instalación rápida
1. Copiar el ZIP al VPS y descomprimirlo: `unzip CORPATELEREDES_CRM_PRO_UBUNTU.zip && cd CORPATELEREDES_CRM_PRO`.
2. Crear configuración: `cp .env.example .env`.
3. Editar `.env` y cambiar OBLIGATORIAMENTE `POSTGRES_PASSWORD`, `DATABASE_URL` (misma contraseña), `JWT_SECRET`, `ADMIN_PASSWORD` y `ROUTER_SECRET_KEY`. Use secretos largos y únicos. No comparta el archivo `.env`.
4. Construir e iniciar: `docker compose up -d --build`.
5. Ver registros: `docker compose logs -f app`.
6. Sin dominio, el panel queda accesible solo desde el propio VPS en `http://127.0.0.1:3000`; use un túnel SSH para administración temporal. No publique HTTP sin HTTPS.

## HTTPS con dominio
1. Ponga `DOMAIN=crm.sudominio.bo` en `.env` y apunte el registro DNS A al VPS.
2. Abra puertos 80 y 443 en firewall y proveedor VPS.
3. Inicie con `docker compose -f docker-compose.yml -f docker-compose.https.yml up -d --build`.
4. Caddy solicitará y renovará el certificado TLS automáticamente.

## MikroTik
- En RouterOS v7 habilite `www-ssl`/REST HTTPS; restrinja el acceso por firewall a la IP del VPS o, preferentemente, a través de VPN.
- Cree un usuario dedicado con permisos mínimos para leer recursos y administrar PPP secrets. No use el usuario `admin` del router.
- Registre host, puerto HTTPS, usuario y contraseña en el módulo MikroTik.
- Pruebe conexión e importe PPPoE. La importación crea clientes sin plan asignado; complete plan, vencimiento y datos de contacto.
- La suspensión/reconexión modifica el atributo `disabled` del PPP secret. Pruebe primero con un usuario de laboratorio y mantenga acceso de emergencia al router.
- Esta versión no automatiza Hotspot, DHCP leases, Queue Tree, VLAN ni perfiles de IPTV en RouterOS.

## QR automático
- Configure `QR_CREATE_URL`, `QR_API_TOKEN`, `QR_WEBHOOK_SECRET` y `BASE_URL` en `.env` según la documentación técnica del proveedor QR contratado.
- El endpoint espera una solicitud HTTP JSON y un webhook con encabezado `x-webhook-signature` calculado como HMAC-SHA256 del cuerpo JSON. Adapte el mapeo de campos/firmas al proveedor real antes de producción.
- No marque pagos como recibidos manualmente hasta comprobar el abono. El QR de muestra del CRM es solo demostrativo y no recibe dinero.
- La conciliación automática depende de la firma, formato y reintentos de webhook del proveedor; se debe probar en sandbox antes de operar.

## Copias de seguridad
`docker compose exec -T db pg_dump -U corpa corpateleredes > backup-corpa.sql`
Restaurar: `cat backup-corpa.sql | docker compose exec -T db psql -U corpa -d corpateleredes`
Guarde backups fuera del VPS y pruebe restauraciones periódicamente.

## Seguridad / estado
- Contraseñas de routers se cifran en base de datos con AES-256-GCM usando `ROUTER_SECRET_KEY`.
- JWT de sesión, hash bcrypt, rate limit en login y registro básico de operaciones.
- Cambie credenciales iniciales, configure HTTPS, firewall, VPN para administración de routers, backups y monitorización.
- No es una integración bancaria certificada ni un sistema fiscal/tributario. Verifique requisitos locales de facturación y protección de datos antes de producción.
- Antes de desplegar en clientes reales, haga pruebas funcionales, auditoría de seguridad y pruebas de recuperación.

## Módulos incluidos
Dashboard, clientes/subclientes, planes, facturas, pagos, QR API adapter, cuentas pendientes, tickets, routers, importación PPPoE, cortes/reconexión PPPoE, canales y paquetes IPTV (catálogo), registros DNS de aplicaciones.
