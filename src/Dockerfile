FROM alpine/git AS deploy
WORKDIR /app
RUN git clone https://github.com/cristurm/nyan-cat.git

FROM nginxinc/nginx-unprivileged:alpine-slim
COPY --from=deploy /app/nyan-cat/ /usr/share/nginx/html
