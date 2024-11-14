# P7.DNS_Linux_MarcosAlonso

## Usando a seguinte guía como base: https://ubuntu.com/server/docs/domain-name-service-dns
## Configura un sistema cun servidor DNS e un cliente alpine que cumpla os seguintes requisitos.
Para comezar con esta practica, teno que comentar que vou adxuntar o `.yaml` xunto cos dous directorios que contenñen os volumenes que eu mesmo modifiquei.
## Volumen por separado da configuración
Como podemos ver no .yaml no apartado de volumes poñemos o seguinte:  
```
    volumes:
      - ./conf:/etc/bind/
      - ./zonas:/var/lib/bind/
```
Aqui como podemos observar, significa que a carpeta `.conf` e `.zonas`, se van a montar como volumenes no bind9.  
Por lo que debemos crear estas carpetas coas suas respectivas configuraciones.  
Na carpeta de zonas debemos poñer as distintas zonas que terá o noso servidor DNS e como podemos observar, eu teno creada unha zona chamada `db.pepe.int`, isto quere dicir que si facemos un dig de por exemplo `dig ns.pepe.int` nos devolvera a ip `192.28.5.1` xa que é o que esta establecido no apartado de zonas.  
  

Luego no apartado de `.conf`, eu o que fixen fue xuntar todos os posibles ficheros de configuracion nun onde poñemos as propias zonas e configuracions como os forwarders, etc.


## Red propia interna para tódo-los contenedores
>Ip fixa no servidor  

Para crear unha IP fixa do servidor é tan sinxelo como marcalo no .yaml como eu fixen. Exemplo no apartado de `network`: 
```networks:
      bind9_subnet:
        ipv4_address: 192.28.5.4
```

> Configurar Forwarders  

Para configurar os forwarders, nos diriximos a o noso unico ficheiro de configuracion en `named.conf`, e no apartado de options, metemos os forwarders, que son os servidores aos que acudira bind9 se no é capaz de resolver a consulta, polo que quedaria tal que así:
```options {
	directory "/var/cache/bind";

    dnssec-validation no; #Soluciona o error de validacion

	forwarders {
	 	8.8.8.8;
		1.1.1.1;
	 };
	 forward only;
}
```

>Crear Zona propia  

Para crear esta zona propia, creamos un fichero de texto na carpeta `zonas` co nome da zona que queremos, no meu caso creei a zona "db.pepe.int"

>Rexistros a configurar: NS, A, CNAME, TXT, SOA  

Para crear estes rexistros na zona que acabamos de crear, entramos no documento de texto e poñemos a configuracion do NS, A, CNAME, SOA, etc. A continuacion vou a deixar un fragmento do documento donde asigno una ip a cada configuracion.  

>[!NOTE]
>Cabe destacar que o documento db.pepe.int, ten todos estas configuracions explicadas máis a fondo

```
ns		IN A		192.28.5.1
test	IN A		192.28.5.4
www		IN A		192.28.5.7
alias	IN CNAME	test
texto	IN TXT		mensaje
prueba23	IN A		125.44.32.1
``` 


>Cliente con ferramientas de rede  

No meu caso instalei un ubuntu xa que o alpine xa habiao instalado na anterior practica e queria probar a poñer un cliente de ubuntu xa que os comandos de instalacion de bash e dig son distintos que os sh que ten alpine.  
Hay duas formas de instalar estas ferramientas de red (dig):  
1. Unha vez feito o `docker compose up` debemos facer noutra terminal un `docker exec -it ubuntu /bin/bash` para asi entrar na terminal do cliente de ubuntu e poñer o comando `apt-get update && apt-get install -y dnsutils` (en alpine seria o comando : "`apk update && apk add --no-cache bind-tools`")  
2. Poñendo directamente no .yaml la configuracion para que asi xa nos teñamos que facer o update e install a man :
```
    command: /bin/bash -c "apt-get update && apt-get install -y dnsutils && bash" 
```

Para alpine seria algo tal que asi:
```
    command: /bin/sh -c "apk update && apk add --no-cache bind-tools && sh
```
  
    
Logo xa solo nos falta comprobar que o `dig` funciona de acuerdo as zonas e todo responde correctamente, a continuacion deixo un exemplo dunha resolucion da miña zona :
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
  
>[!NOTE]
>E por ultimo cabe destacar que tanto no `yaml` como nas carpetas `conf` e `zonas` temos todas al lineas correctamente explicadas para saber o correcto funcionamento de todo o sistema.