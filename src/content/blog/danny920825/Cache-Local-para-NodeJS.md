---
title: "Cache Local para NodeJS"
pubDate: "Tue Mar 21 2023"
image: ""
username: "danny920825"
categories: ["tutorials","devops","software"]
description: "Como crear un sistema de cache en nuestra PC que nos permita agilizar la instalación de paquetes de node via npm, pnpm o yarn"
canonicalUrl: ""
---

## Capítulo 1: El problema
 
Si eres Cubano, sufres la conexion de ETECSA y programas en JavaScript, pues vengo a traerles una solución que estoy usando que va a eliminar varios problemas
1. El bloqueo de NPM, PNPM, Yarn, etc a Cuba
2. La vida que ves pasar mientras inicializas un proyecto nuevo
3. Velocidad

Se trata de crear una caché local en tu PC que perdure X dias (los que tú quieras pero no te pases) para que los paquetes esten disponibles. En mi caso voy a usar Docker porque no me gusta el churre de las dependencias globales, pero si lo instalas global tambien funciona.

## Capítulo 2: La solución
 
La herramienta que vamos a utilizar se llama Verdaccio y es un proxy de registry que es super ligero y rapido, facil de utilizar y de configurar, pues solo necesita un archivo YAML para trabajar. Asi que lo primero es descargar la imagen.
Para eso, vamos a hacer uso del siguiente comando de Docker:
``` docker pull verdaccio:verdaccio```
 
Lo segundo será echar a andar el servicio. Aqui te dejo tanto el docker run como el docker-compose, depende de ti la opcion a utilizar aunque recomiendo siempre la segunda.

### Docker Run
```
docker run \
  --name verdaccio \
  -p 4873:4873 \
  -v ./conf:/verdaccio/conf \
  verdaccio/verdaccio
  ``` 
  
### Docker Compose
```
version: '3.1'

services:
  verdaccio:
    image: verdaccio/verdaccio
    container_name: 'verdaccio'
    networks:
      - node-network
    environment:
      - VERDACCIO_PORT=4873
    ports:
      - '4873:4873'
    volumes:
      - './storage:/verdaccio/storage'
      - './config:/verdaccio/conf'
      - './plugins:/verdaccio/plugins'
networks:
  node-network:
    driver: bridge
```

## Capítulo 3: La configuracion
 
Verdaccio usa un fichero YML como ya habiamos dicho y es en ese fichero donde vamos a configurar elementos como la duración de la caché, el servidor a donde haremos las peticiones, etc.
En mi caso, el archivo config.yaml se ve asi: 
 
 ```
 storage: ./storage
auth:
  htpasswd:
    file: ./htpasswd
uplinks:
  bitqba:
    url: https://npm.bitqba.com/
    maxage: 7d
packages:
  '**':
    access: $all
    publish: $authenticated
    proxy: bitqba
  '@*/*':
    access: $all
    publish: $authenticated
    proxy: bitqba
log: { type: stdout, format: pretty, level: http }
```
 
El storage es donde va a guardar los paquetes
El auth es por si quieren publicar sus paquetes en su repo
Los uplinks son los servidores npm que van a usar. En este caso, el de @bitqba
es publico y gratuito y evidentemente no esta bloqueado para cuba.
el maxage es la cantidad de dias que quieren de cache. 
Usualmente viene por 1d pero lo pueden poner a mas si no usan librerias que se actualicen a diario y no les preocupa eso. 
Y el resto son opciones de acceso donde dice que cualquiera puede acceder, los autenticados publicar.
y el proxy que van a usar para conectarse.
Ahora viene la parte final, cambiar el registry a localhost. Para eso acceden a 
http://localhost:4873/ y van a la rueda de opciones y veran  comandos para npm, pnpm y Yarn.
Igual los dejo aquí a continuación:
 

```
npm set registry http://127.0.0.1:4873/
```
```
pnpm set registry http://127.0.0.1:4873/
```
```
yarn config set registry http://127.0.0.1:4873/
```
 
Y una vez hecho todo esto, ya podrán hacer uso de esta caché en sus proyectos. Así que cuentame tu experiencia en los comentarios y dime qué te pareció?

 

