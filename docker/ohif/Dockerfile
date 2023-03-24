FROM ohif/viewer:v3.8.16.10895 as builder

FROM openresty/openresty:alpine-fat

RUN luarocks install lua-resty-openidc 1.7.6
RUN apk update && apk add git && luarocks install lua-resty-cookie

# Copy build output to image
COPY --from=builder /usr/share/nginx/html /var/www/html

ENTRYPOINT ["/usr/local/openresty/nginx/sbin/nginx", "-g", "daemon off;"]
