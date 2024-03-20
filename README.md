# OHIF Viewer Using PACS Server (Orthanc) Protected by OpenID

We protect the PACS Server with OpenID, for example, with corporate SSO.

It's based on OpenResty (nginx + lua) and Keycloak as the OpenID auth.

We use https://github.com/zmartzone/lua-resty-openidc for authentication.

It's a kind of quick and dirty solution, but the main point is - it just works, 
contrary to all others that I've found so far.

### CORS

The basic idea is we work around the CORS problem by proxying all services as endpoints on
one host.
All services are proxied from `viewer`, which is actually OpenResty + OHIF viewer.

This way, there's just no CORS problem at all.

A separate nginx in the docker-compose is actually not needed; it's just for quick experiments.

### Auth Session - Nginx Config
Play with `session.cookie.renew` and `session.cookie.lifetime` to set up session lifetime.

You can set the cookie's domain using `authenticate()` fourth argument!

``` 
local session_opts = { cookie = { domain = ".mydomain.com" } }
```

### Start

```
docker-compose up --build
```

Give it some time to start all containers and initialize Keycloak DB; the first time, it could take
up to a minute.
There's no special means to sync start, so it could get stuck; you'll see that in the logs.

```
docker-compose logs -f
```

In this case, just restart it.

```
docker-compose restart
```

### Keycloak

To make it work, you need these settings in Keycloak:

0) Log in to Keycloak admin console http://localhost:3333 (see creds in docker-compose.yml).
1) Create realm `imagingrealm` (left upper corner, "Add realm" button in the dropdown).
2) Create clients:
   - `imaging`, Redirect URL `*`, access type: confidential, Web Origins `+`
3) Add a secret key from the Keycloak user PACS Credentials tab to the `openid-keycloak-secrets.env`, 
var `OPENID_CLIENT_SECRET`.
4) Create a user with any name and set a password in the `Credentials` tab; you will use this user to get access to 
resources protected by nginx with OpenIDC.
5) Restart nginx to read the new secret key.

```
docker-compose stop viewer
docker-compose up -d
```

### Links
http://localhost/pacs-admin/  
Orthanc admin console.
You can upload some DICOM files here (`upload` button top right).
`Select files to upload..` and don't forget to press `Start upload` - UI is just genius :(

http://localhost/  
OHIF viewer connected to Orthanc.

http://localhost/pacs/series  
Just an example calling Orthanc API.

### SSL Key
An SSL key is included in the repo.
DO NOT USE IT in production - the secret key is exposed in this repo.

Regenerate it with:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginxenv/ssl/nginx.key -out nginxenv/ssl/nginx.crt
```
