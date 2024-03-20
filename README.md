# OHIF Viewer Using PACS Server (Orthanc) Protected by OpenID

We protect the PACS Server with OpenID, like corporate SSO.

Based on OpenResty (nginx + lua) and Keycloak for OpenID auth.

Use https://github.com/zmartzone/lua-resty-openidc for authentication.

A simple solution that works well, unlike others we've tried.

### CORS

We solve CORS issues by proxying all services through a single host.
Services are proxied from `viewer`, which combines OpenResty and OHIF viewer.

This eliminates CORS problems.

### Auth Session - Nginx Config
Adjust `session.cookie.renew` and `session.cookie.lifetime` for session timing.

Set cookie domain with `authenticate()`'s fourth argument:

```
local session_opts = { cookie = { domain = ".mydomain.com" } }
```

### Start

```
docker-compose up --build
```

Allow time for container startup and Keycloak DB initialization. Initial start may take a minute.
No special sync for start; if it hangs, check logs:

```
docker-compose logs -f
```

Restart if needed:

```
docker-compose restart
```

### Keycloak

Setup in Keycloak:

0) Access Keycloak admin console at http://localhost:3333 (credentials in docker-compose.yml).
1) Create realm `imagingrealm` (top left, "Add realm" in dropdown).
2) Create client `imaging`:
   - `Redirect URL`: `*`
   - `access-type:` confidential
   - `Web Origins`: `+`
3) Add secret key from Keycloak user PACS Credentials to `openid-keycloak-secrets.env`, var `OPENID_CLIENT_SECRET`.
4) Create a user, set password in `Credentials`, to access nginx protected resources with OpenIDC.
5) Restart nginx to apply new secret key.

```
docker-compose stop viewer
docker-compose up -d
```

### Links
- Admin Console: http://localhost/pacs-admin/  
   Upload DICOM files here (`upload` button top right). Don't forget `Start upload`.
- OHIF Viewer: http://localhost/  
   Connected to Orthanc.
- API Example: http://localhost/pacs/series  
   Calls Orthanc API.

### SSL Key
SSL key included for development.
DO NOT USE in production. Secret key is public in this repo.

Regenerate for production:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginxenv/ssl/nginx.key -out nginxenv/ssl/nginx.crt
```
