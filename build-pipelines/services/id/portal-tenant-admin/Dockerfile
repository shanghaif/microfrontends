# Use this to build a production image of a node single page app hosted in lerna monorepo

# Relative path to local package root - i.e. services/id/portal-tenant-admin
ARG PACKAGE_ROOT=services/id/portal-tenant-admin
# NPM Package command name to make a production build - i.e. build:id-portal-tenant-admin:prod
ARG PACKAGE_BUILD_COMMAND=build:id-portal-tenant-admin:prod

# BASE
FROM node:lts AS base
WORKDIR /checkout

ARG PACKAGE_ROOT
RUN test -n "$PACKAGE_ROOT" || (echo "PACKAGE_ROOT not set" && false)

ARG PACKAGE_BUILD_COMMAND
RUN test -n "$PACKAGE_BUILD_COMMAND" || (echo "PACKAGE_BUILD_COMMAND not set" && false)

# BUILD
FROM base AS build

ARG PACKAGE_ROOT
ARG PACKAGE_BUILD_COMMAND

# Custom env variables to be set during build time - i.e. backend api url
ARG TENANT_ADMINISTRATION_ENDPOINT
RUN test -n "$TENANT_ADMINISTRATION_ENDPOINT" || (echo "TENANT_ADMINISTRATION_ENDPOINT not set" && false)
ENV REACT_APP_TENANT_ADMINISTRATION_ENDPOINT=$TENANT_ADMINISTRATION_ENDPOINT

COPY [".", "."]
RUN npm run $PACKAGE_BUILD_COMMAND

# RELEASE
FROM nginx:stable-alpine

ARG PACKAGE_ROOT
ARG PACKAGE_BUILD_COMMAND

RUN rm /etc/nginx/conf.d/default.conf
COPY --from=build ["/checkout/build-pipelines/$PACKAGE_ROOT/nginx/nginx.conf", "/etc/nginx/"]
COPY --from=build ["/checkout/build-pipelines/$PACKAGE_ROOT/nginx/portal-tenant-admin.conf", "/etc/nginx/conf.d/"]
COPY --from=build ["/checkout/$PACKAGE_ROOT/build", "/usr/share/nginx/html/"]

CMD ["nginx", "-g", "daemon off;"]

EXPOSE 80/tcp
