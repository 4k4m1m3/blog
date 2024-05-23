---
title: Dockerlabs.es
published: true
---

## Dockerlabs.es nueva plataforma para aprender hacking ético y ciberseguridad

# ¿Qué es Dockerlabs.es?

Su mismo creador la define como nueva plataforma donde se podrá aprender hacking ético y ciberseguridad de una forma muy sencilla y con el menor consumo de recursos posible.

# Como yo veo Dockerlabs.es

Me gustaría destacar que el creador de esta plataforma es Mario Álvarez Fernández (aka. El Pingüino de Mario), he tenido el privilegio de intercambiar conocimientos con Mario y puedo decir que es una persona sumamente amable, respetuosa y dedicada al área del hacking ético. Además, es mi profesor, ya que estoy inscrito en su academia [elrincondelhacker.es] así que puedo mencionar que tanto su plataforma como su academia se destacan por ser interactivas y prácticas, ofreciendo un excelente programa educativo para quienes desean aprender y comenzar en el mundo de la ciberseguridad.

Algunas características a destacar serian:

## **Laboratorio de Hacking:**

- _Diversidad de niveles:_ Los laboratorios están categorizados por niveles de dificultad, desde "Muy fácil" hasta "Difícil", permitiendo a los usuarios avanzar según su ritmo y nivel de experiencia.

- _Experiencia práctica:_ La plataforma ofrece una experiencia educativa práctica en la que los usuarios pueden aprender y aplicar técnicas de hacking en un entorno controlado.

## **Tecnología Docker:**

- _Despliegue rápido:_: Gracias a Docker, los laboratorios se pueden desarrollar en cuestión de segundos utilizando contenedores. Esto asegura que cada laboratorio sea ligero, portátil y encapsule todo lo necesario para su ejecución independiente.

- _Uso óptimo de los recursos:_ A diferencia de las máquinas virtuales tradicionales que usan VirtualBox o VMWare, Dockerlabs.es utiliza contenedores que comparten el sistema operativo del host, lo que reduce significativamente la cantidad de recursos necesarios. Esta eficiencia permite que incluso los usuarios con equipos de menor capacidad puedan acceder y utilizar la plataforma sin problemas.

## **Contribución y Comunidad:**

- _Subida y descarga de máquinas:_ Los usuarios tienen la posibilidad de buscar, descargar y subir máquinas, fomentando una comunidad activa y colaborativa.

- _Writeups y soluciones:_ La plataforma también facilita el acceso a writeups de otros usuarios, permitiendo que los principiantes puedan seguir pasos detallados y aprender de la experiencia de otros, adicional que por los WriteUps enviados participas en el racking comunitario.

## **Automatización y Facilidad de Uso:**

Dockerlabs.es automatiza el despliegue y la eliminación de las máquinas de forma sencilla, lo que simplifica enormemente la gestión de los laboratorios y mejora la experiencia del usuario, así combina la potencia y eficiencia de Docker con un enfoque práctico y educativo en ciberseguridad, proporcionando una plataforma accesible, ideal tanto para principiantes como para usuarios avanzados.

# Usar Dockerlabs.es

Aunque es un proceso realmente fácil y sencillo este proceso, igual lo explico, lo primero seria ingresa a [dockerlabs.es], encontrarás un apartado llamado [Instrucciones de uso](https://dockerlabs.es/assets/instrucciones_de_uso.pdf). Aquí Mario explica en detalles desde los requisitos previos del sistema hasta la instalación en Linux, pasando por la descarga de las máquinas, su ejecución y posterior eliminación.

Lo primero que necesitas hacer es instalar Docker en tu equipo. Para ello, simplemente ejecuta el siguiente comando:

```bash
sudo apt install docker.io
```

Después de instalarlo, el siguiente paso es decidir qué máquina quieres utilizar para aprender conceptos de ciberseguridad. Una vez que hayas elegido, debes descargar el archivo correspondiente, descomprimirlo (puedes instalar y usar "unzip" para esto), y encontrarás dos archivos en la carpeta extraída: un script para el autodespliegue de la máquina, llamado **"auto_deploy.sh"**, y el archivo de la máquina en formato tar, por ejemplo, **"ejemplo.tar".**

Para desplegar la máquina, todo lo que necesitas hacer es ejecutar el siguiente comando en tu terminal:

```bash
sudo bash auto_deploy.sh nombre_maquina.tar
```

Una vez que hayas terminado de trabajar con la máquina, simplemente presiona **Ctrl+C** para eliminarla del sistema. ¡Así de sencillo es el proceso! A diferencia de otras plataformas online que requieren registro y ofrecen contenido premium, Dockerlabs.es es completamente gratuita y no requiere registro.

_Nota:_ Las máquinas en Dockerlabs.es no tienen flags como user.txt o root.txt. Sin embargo, el concepto es el mismo: ingresar a la máquina aprovechando alguna vulnerabilidad y luego escalar privilegios.

---
Próximamente, estaré resolviendo máquinas de la plataforma Dockerlabs.es. ¡Así que mantente atento a mis próximas publicaciones!


```
¡Que la fuerza del hacking ético nos acompañe! :)
```

|   |
|:--|
|   |