# OHIF viewer using PACS Server (Orthanc) protected by OpenID

We protect PACS Server with OpenID - for example with corporate SSO.

Based on OpenResty (nginx + lua) and Keycloak as OpenID auth.

Use https://github.com/zmartzone/lua-resty-openidc for authentication.

Kind of quick and dirty solution but the main point - it just works 
in contratry to all others that I found so far.

### CORS

Basic idea - we workaround CORS problem proxying all services as endpoints on
one host.
All services are proxed from `viewer` which is actually a OpenResty + OHIV viewer.

This way there is just no CORS problem at all.

Separate nginx in the docker-compose actually is not needed, just for quick experiments.

### Auth Session - nginx config
Play with session.cookie.renew and session.cookie.lifetime to setup session lifetime.

You can set the cookies domain using authenticate() fourth argument!

    local session_opts = { cookie = { domain = ".mydomain.com" } }

### Start

    docker-compose up --build

Give it some time to start all containers and init KeyCloack DB, first time it could take
up to minute.
There is no special means to sync start so it could stuck, you will see that in logs

    docker-compose logs -f

In this case just restart it

    docker-compose restart

### KeyCloack

To make it work you need this settings in KeyCloack:

0) login to KeyCloack adm console http://localhost:3333 (see creds in docker-compose.yml) 
1) Create realm `imagingrealm` (left up corned, "Add realm" button in the drop-down)
2) Create clients:
   - `imaging`, Redirect URL `*`, access-type: confidential, WebOrigins `+`
3) add a secret key from the keycloak user pacs Credentials tab to the `openid-keycloak-secrets.env`, 
var `OPENID_CLIENT_SECRET`
4) create user with any name and set password in `Credentials` tab, you will user this user to get access to 
resources protected by nginx with OpenIDC.
5) restart nginx to read new secret key 

       docker-compose stop viewer
       docker-compose up -d


### Links
http://localhost/pacs-admin/  
Orthang admin console
you can upload some DICOM files here (`upload` button top right)
`Select files to upload..` and do not forget to press `Start upload` - UI just genious :(
http://localhost/
OHIF viewer connected to the Orthanc
http://localhost/pacs/series
just example calling Orthanc API

### SSL Key
In the repo included SSL key.
DO NOT USE IT in production - the secret key is exposed in this repo.

Regenerate it with

   openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginxenv/ssl/nginx.key -out nginxenv/ssl/nginx.crt