FROM antora/antora:3.1.4

RUN apk update && apk add make git yq jq

RUN yarn cache clean
RUN yarn global add antora @antora/lunr-extension asciidoctor-kroki asciidoctor-emoji

# These environment variables are required since Antora 2.2
# to customize the "edit this page" URL
# https://docs.antora.org/antora/2.2/whats-new/#customizable-edit-url
ENV FORCE_SHOW_EDIT_PAGE_LINK=1
ENV CI=1