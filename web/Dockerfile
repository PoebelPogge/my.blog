# Basis-Image mit Hugo
FROM hugomods/hugo:latest AS build

# Arbeitsverzeichnis setzen
WORKDIR /src

# Projektdateien in den Container kopieren
COPY . .

RUN git clone https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke

# Hugo Module Abhängigkeiten auflösen
RUN hugo mod tidy

# Seite bauen
RUN hugo --minify

# Produktions-Image mit Nginx
FROM nginx:alpine

# Inhalt aus dem Build in das Nginx-Verzeichnis kopieren
COPY --from=build /src/public /usr/share/nginx/html

# Optional: Port setzen (nicht zwingend notwendig)
EXPOSE 80

# Startbefehl
CMD ["nginx", "-g", "daemon off;"]