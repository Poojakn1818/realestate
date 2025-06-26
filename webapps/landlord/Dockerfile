FROM node:20-bookworm-slim AS base

FROM base AS deps
WORKDIR /usr/app

# Copy package files relative to context (which is webapps/landlord/)
COPY package.json .
COPY ../../.yarnrc.yml .
COPY ../../yarn.lock .
COPY ../../.yarn/plugins .yarn/plugins
COPY ../../.yarn/releases .yarn/releases
COPY ../../webapps/commonui/package.json ../commonui/package.json
RUN --mount=type=cache,id=node_modules,target=/root/.yarn YARN_CACHE_FOLDER=/root/.yarn \
    yarn workspaces focus @microrealestate/landlord

FROM base AS build
WORKDIR /usr/app
ENV BASE_PATH=/__MRE_BASE_PATH__
ENV NEXT_TELEMETRY_DISABLED=1

COPY --from=deps /usr/app ./

# Copy commonui folder relative to context (outside, so ../)
COPY ../../webapps/commonui/.eslintrc.json ../commonui/
COPY ../../webapps/commonui/components ../commonui/components
COPY ../../webapps/commonui/locales ../commonui/locales
COPY ../../webapps/commonui/scripts ../commonui/scripts
COPY ../../webapps/commonui/utils ../commonui/utils

# Copy landlord files inside context
COPY .eslintrc.json .
COPY locales locales
COPY public public
COPY src src
COPY i18n.js .
COPY next.config.js .
COPY package.json .
COPY postcss.config.js .
COPY tailwind.config.js .
COPY tsconfig.json .

RUN yarn workspace @microrealestate/landlord run build

# prepare runtime files
RUN touch /usr/app/public/__ENV.js
RUN chown -R 1000 /usr/app/.next && chown -R 1000 /usr/app/public

FROM gcr.io/distroless/nodejs20-debian12
WORKDIR /usr/app
ENV NEXT_TELEMETRY_DISABLED=1

COPY --from=build /usr/app/public ./public
COPY --from=build /usr/app/.next/standalone/node_modules ./node_modules
COPY --from=build /usr/app/.next/standalone/webapps/landlord ./
COPY --from=build /usr/app/.next/static ./.next/static
COPY --from=build /usr/app/../commonui/scripts/generateruntimeenvfile.js ./ 
COPY --from=build /usr/app/../commonui/scripts/replacebasepath.js ./ 
COPY --from=build /usr/app/../commonui/scripts/runner.js ./ 

USER 1000
CMD ["runner.js"]
