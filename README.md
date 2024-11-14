# P7.DNS_Linux_MarcosAlonso

## Usando a seguinte guía como base: https://ubuntu.com/server/docs/domain-name-service-dns
## Configura un sistema cun servidor DNS e un cliente alpine que cumpla os seguintes requisitos.
Para comezar con esta practica, teno que comentar que vou adxuntar o `.yaml` xunto cos dous directorios que contenñen os volumenes que eu mesmo modifiquei.
## Volumen por separado da configuración
Como podemos ver en el .yaml en el apartado de volumenes ponemos lo siguiente 
```
    volumes:
      - ./conf:/etc/bind/
      - ./zonas:/var/lib/bind/
```
Aqui como podemos observar, significa que la carpeta .conf y .zonas, se van a montar como volumenes en el bind9.  
Por lo que debemos crear estas carpetas con sus respectivas configuraciones.  
En la carpeta de zonas debemos poner las distintas zonas que tendra nuestro servidors DNS y como podemos observar, yo tengo creada una zona llamada `db.pepe.int` y esto quiere decir que si hacemos un dig de por ejemplo `dig ns.pepe.int` nos devolvera la ip `192.28.5.1` ya que es lo que esta establecido en el apartado de zonas.  
  

Y luego en el apartado de .conf, yo lo que hice fue juntar todos los posibles ficheros de configuracion en uno en donde ponemos las propias zonas y configuraciones como los forwarders, etc.




## Red propia interna para tódo-los contenedores
>Ip fixa no servidor  

Para crear una IP fija del servidor es tan sencillo como marcarlo en el .yaml como yo hice. Ejemplo en el apartado de network: 
```networks:
      bind9_subnet:
        ipv4_address: 192.28.5.4
```

> Configurar Forwarders  

Para configurar los forwarders, nos dirigimos a nuestro unico fichero de configuracion en `named.conf`, y en apartado de options, metemos los forwarders que son los servidores a los que acudira bind9 si no es capaz de resolver la consulta, por lo que quedaria tal que así:
```options {
	directory "/var/cache/bind";

    dnssec-validation no; #Soluciona el error de validacion

	forwarders {
	 	8.8.8.8;
		1.1.1.1;
	 };
	 forward only;
}
```

>Crear Zona propia  

Para crear esta zona propia, creamos un fichero de texto en la carpeta zonas con el nomre de la zona que queremos, en mi caso cree la zona "db.pepe.int"

>Rexistros a configurar: NS, A, CNAME, TXT, SOA  

Para crear estes rexistros na zona que acabamos de crear, entramos en el documento de texto que acabamos de crear y ponemos a configurar la segunda parte del documento donde configuramos el NS, A, CNAME, SOA, etc. A continuacion voy a dejar un fragmento del documento donde asigno una ip a cada configuracion.
```
ns		IN A		192.28.5.1
test	IN A		192.28.5.4
www		IN A		192.28.5.7
alias	IN CNAME	test
texto	IN TXT		mensaje
prueba23	IN A		125.44.32.1
``` 


>Cliente con ferramientas de rede  

En mi caso instale un ubuntu ya que el alpine lo habia instalado en la anterior practica y queria probar a poner un cliente de ubuntu ya que el comando de instalacion de bash es distinto que el de sh que tiene alpine.  
Hay dos formas de instalar estas herramientas de red (dig):  
1. Una vez echo el `docker compose up` hacer en otra terminal un `docker exec -it ubuntu /bin/bash` para asi entrar en la terminal del cliente de ubuntu y poner el comando `apt-get update && apt-get install -y dnsutils` ( en alpine seria : "`apk update && apk add --no-cache bind-tools`")  
2. Poniendo en el .yaml la configuracion :
```
    command: /bin/bash -c "apt-get update && apt-get install -y dnsutils && bash" 
```

Para alpine seria algo tal que asi:
```
    command: /bin/sh -c "apk update && apk add --no-cache bind-tools && sh
```
  
    
    Y luego ya solo nos falta conprobar que el dig funciona de acuerdo a las zonas y todo responde correctamente, a continuacion dejo un ejemplo de una resolucion de mi zona :
```
    dig ns.pepe.int

; <<>> DiG 9.18.28-0ubuntu0.24.04.1-Ubuntu <<>> ns.pepe.int
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4194
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: da297157b34f518201000000673621face3780a8c6c3ab1b (good)
;; QUESTION SECTION:
;ns.pepe.int.			IN	A

;; ANSWER SECTION:
ns.pepe.int.		38400	IN	A	192.28.5.1

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Thu Nov 14 16:14:50 UTC 2024
;; MSG SIZE  rcvd: 84

```