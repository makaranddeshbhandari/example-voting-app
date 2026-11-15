FROM node:22-bookworm-slim

RUN apt-get update -qq && apt-get install -qy \
    ca-certificates \
    bzip2 \
    curl \
    libfontconfig1 \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g phantomjs-prebuilt

COPY . /app

WORKDIR /app

CMD ["/app/tests.sh"]



