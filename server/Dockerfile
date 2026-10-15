FROM denoland/deno:debian-2.4.2 AS build
WORKDIR /src
COPY . .
RUN deno task build:server && mkdir /data

FROM gcr.io/distroless/cc-debian12:nonroot

COPY --from=build /src/server/build/server /app/server
COPY --from=build --chown=nonroot:nonroot /data /data

ADD --chmod=444 \
    --checksum=sha256:c3282f7b247167eb1f2e63bdbd99cdd7def5095932453329e68dae252312f52d \
    https://github.com/denodrivers/sqlite3/releases/download/0.12.0/libsqlite3.so \
    /app/libsqlite3.so

ENV SERVER_PORT=8080 \
    DB_PATH=/data/db.sqlite3 \
    DENO_SQLITE_PATH=/app/libsqlite3.so

VOLUME ["/data"]
EXPOSE 8080
ENTRYPOINT ["/app/server"]
