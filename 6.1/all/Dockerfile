FROM debian:bullseye-20230522-slim

LABEL maintainer="wxai@aizen.chat"
ENV BUNDLE_VERSION 6.1.8

# dependencies
RUN apt update -y && apt install -y g++ build-essential git curl python2-minimal wget

# install meteor
RUN curl https://install.meteor.com | sed s/--progress-bar/-sL/g | /bin/sh
RUN export METEOR_NO_RELEASE_CHECK=true \
    && export METEOR_SYMLINK_TARGET=/root/.meteor/meteor \
    && export METEOR_TOOL_DIRECTORY=/root/.meteor \
    && export METEOR_ALLOW_SUPERUSER=true \
    && export PATH=$PATH:/usr/local/bin

# install node npm yarn
WORKDIR /opt/
RUN wget https://nodejs.org/dist/v14.21.2/node-v14.21.2-linux-x64.tar.xz && tar -xvf node-v14.21.2-linux-x64.tar.xz -C /usr/local/
RUN ln -s /opt/node-v14.21.2-linux-x64/bin/node /usr/local/bin/node \
    && ln -s /opt/node-v14.21.2-linux-x64/bin/npm /usr/local/bin/npm
RUN npm install --global yarn && ln -s /opt/node-v14.21.2-linux-x64/bin/yarn /usr/local/bin/yarn

#  build bundle for wxai
RUN mkdir /tmp/dist
WORKDIR /opt/
RUN git clone https://oauth2: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@github.com/taotao2008/wxai.git
WORKDIR /opt/wxai
RUN yarn
RUN yarn turbo run translation-check
RUN yarn build:ci -- --directory /tmp/dist
# --chown requires Docker 17.12 and works only on Linux
RUN cp -rf /tmp/dist/bundle /app/
WORKDIR /app/
RUN tar -zcvf bundle-$BUNDLE_VERSION.tar.gz bundle/

#  upload bundle to github
RUN curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | /bin/sh \
    && apt-get install -y git-lfs \
    && git lfs install
WORKDIR /opt/
RUN git clone https://github.com/taotao2008/wxai-server-package.git
WORKDIR /opt/wxai-server-package
RUN mkdir -p /opt/wxai-server-package/server/$BUNDLE_VERSION

WORKDIR mkdir -p /opt/wxai-server-package/server/$BUNDLE_VERSION
RUN cp -rf /app/bundle-$BUNDLE_VERSION.tar.gz .
RUN echo `md5sum bundle-$BUNDLE_VERSION.tar.gz` > bundle-$BUNDLE_VERSION.tar.gz.md5
WORKDIR /opt/wxai-server-package
RUN git lfs track server/$BUNDLE_VERSION/bundle-$BUNDLE_VERSION.tar.gz \
    && git add .gitattributes \
    && git commit -m `commit .gitattributes` \
    && git push -u origin wali

RUN git add server/$BUNDLE_VERSION/bundle-$BUNDLE_VERSION.tar.gz.md5 \
    && git commit -m `commit bundle-$BUNDLE_VERSION.tar.gz.md5` \
    && git push -u origin wali
RUN git add server/$BUNDLE_VERSION/bundle-$BUNDLE_VERSION.tar.gz \
    && git commit -m `commit bundle-$BUNDLE_VERSION.tar.gz` \
    && git push -u origin wali


# needs a mongoinstance - defaults to container linking with alias 'mongo'
ENV DEPLOY_METHOD=docker \
    NODE_ENV=production \
    MONGO_URL=mongodb://mongo:27017/rocketchat \
    HOME=/tmp \
    PORT=3000 \
    ROOT_URL=https://chat.aizen.chat:3000 \
    Accounts_AvatarStorePath=/app/uploads

EXPOSE 3000

#CMD ["node", "main.js"]
WORKDIR /app
RUN wget --no-check-certificate  https://raw.githubusercontent.com/taotao2008/wxai-server-package/wali/docker-entrypoint.sh -O docker-entrypoint.sh
RUN chmod +x /app/docker-entrypoint.sh
ENTRYPOINT ["/app/docker-entrypoint.sh"]
