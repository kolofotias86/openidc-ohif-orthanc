# OHIF Viewer with OpenID-Protected PACS Server (Orthanc)

This project implements a secure medical imaging viewer setup that protects the PACS Server using OpenID authentication (compatible with corporate SSO). The solution uses OpenResty (nginx + lua) and Keycloak for OpenID authentication, leveraging the [lua-resty-openidc](https://github.com/zmartzone/lua-resty-openidc) library.

## Features

- Single-host deployment eliminating CORS issues
- Corporate SSO integration via OpenID
- Streamlined service proxying through OpenResty
- OHIF Viewer integration with Orthanc PACS

## Architecture

### CORS Handling
All services are proxied through a single host (`viewer` container), which combines OpenResty and the OHIF viewer. This approach completely eliminates CORS-related issues by having all services available under one domain.

### Authentication Configuration
Session timing can be configured through the nginx configuration:
- Use `session.cookie.renew` to set renewal timing
- Use `session.cookie.lifetime` to set session duration

To set a custom cookie domain:
```lua
local session_opts = { cookie = { domain = ".mydomain.com" } }
```

## Deployment

### Quick Start
```bash
docker-compose up --build
```

**Note:** Initial startup may take a minute while Keycloak initializes its database. If the startup appears to hang, check the logs:
```bash
docker-compose logs -f
```

If needed, restart the stack:
```bash
docker-compose restart
```

### Keycloak Configuration

1. Access the Keycloak admin console at http://localhost:3333 (credentials in docker-compose.yml)
2. Create a new realm named `imagingrealm`:
   - Use the realm dropdown (top left)
   - Select "Add realm"
3. Create a client named `imaging` with the following settings:
   - Redirect URL: `*`
   - Access Type: `confidential`
   - Web Origins: `+`
4. Copy the client secret from Keycloak to `openid-keycloak-secrets.env`:
   - Set as `OPENID_CLIENT_SECRET`
5. Create a user account and set their password in the "Credentials" tab
6. Restart nginx to apply the new configuration:
   ```bash
   docker-compose stop viewer
   docker-compose up -d
   ```

## Access Points

- OHIF Viewer: http://localhost/
  - Main viewer interface connected to Orthanc
- Admin Console: http://localhost/pacs-admin/
  - Use to upload DICOM files (upload button in top right corner)
- API Example: http://localhost/pacs/series
  - Demonstrates Orthanc API access

## Security Notes

### SSL Configuration
A development SSL key is included in this repository. **DO NOT USE IN PRODUCTION**.

Generate a new SSL key pair for production:
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout nginxenv/ssl/nginx.key \
    -out nginxenv/ssl/nginx.crt
```