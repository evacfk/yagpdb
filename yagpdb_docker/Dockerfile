FROM golang:stretch as builder

WORKDIR /appbuild


# Uncomment during development
COPY . /appbuild/yagpdb

RUN git clone https://github.com/evacfk/yageconomy

WORKDIR /appbuild/yagpdb/cmd/yagpdb

RUN CGO_ENABLED=0 GOOS=linux go build -v

#RUN CGO_ENABLED=0 GOOS=linux go get -v \
#  github.com/jonas747/yagpdb/cmd/yagpdb

FROM alpine:latest

WORKDIR /app
# /app/cert looks like where it's expecting certificates. maybe where /letsencrypt should point to
VOLUME /app/soundboard \
  /app/cert
EXPOSE 80 443
#EXPOSE 5000 5001

# We need the X.509 certificates for client TLS to work.
RUN apk --no-cache add ca-certificates

# Add ffmpeg for soundboard support
RUN apk --no-cache add ffmpeg

# Handle templates for plugins automatically
COPY --from=builder /appbuild/yagpdb/*/assets/*.html templates/plugins/

COPY --from=builder /appbuild/yagpdb/cmd/yagpdb/templates templates/
COPY --from=builder /appbuild/yagpdb/cmd/yagpdb/posts posts/
COPY --from=builder /appbuild/yagpdb/cmd/yagpdb/static static/

COPY --from=builder /appbuild/yageconomy/assets/*.html templates/plugins/

COPY --from=builder /appbuild/yagpdb/cmd/yagpdb .

# add extra flags here when running YAGPDB
# Set "-exthttps=true" if using a TLS-enabled proxy such as
# jrcs/letsencrypt-nginx-proxy-companion
# Set "-https=false" do disable https
ENV extra_flags "-feeds=reddit,twitter"

# `exec` allows us to receive shutdown signals.
CMD exec /app/yagpdb -all -pa $extra_flags -backgroundworkers
