---
title: "Crea un servidor casero Parte 1: Como convertir un portátil viejo en un servidor casero"
description: "¡Dale una nueva vida a un portátil antiguo que ya no estas utilizando! Aprende a crear un servidor casero con Linux Debian, realizar configuración básica e instalar servicios básicos como SSH y SMB."
published: 2026/05/05
last-modified: 2026/05/05
slug: "crea-un-servidor-casero-parte-1-como-convertir-un-portatil-viejo-en-un-servidor-casero"
---

BLOG EN CONSTRUCCIÓN

> Si quieres ir directo al tutorial y la parte técnica, sáltate el prefacio.

## Prefacio

Seguramente en tu casa debes tener todo tipo de dispositivos y aparatos a los que ya no le das uso.

[//]: <> (# TODO: Mejorar la descripción del portátil viejo.)

Una tableta a la que se le daño la pantalla táctil, una PDA antigua a la que se le murió la batería, un celular que ya no esta de moda, o un portátil viejo que hace mucho cumplió su deber.

Probablemente no sepas que hacer con todos estos aparatos en desuso. Ya no los usas porque están dañados u obsoletos. No los botas porque por lo menos aun prenden. O quizás simplemente no te quieres deshacer de ellos porque aun guardan valor sentimental para ti (o porque te costaron mucho dinero en su momento).

[//]: <> (# TODO: Especificar la marca del portátil.)

Este ultimo fue mi caso con un viejo portátil ... que le regalaron a mi padre, el cual cumplió valientemente su deber como mi computador personal durante parte de mis años de secundaria y mis primeros años de universidad. Durante su servicio como mi principal medio de trabajo y fuente de entretenimiento, se le daño la carcasa y se recalentaba fácilmente, por lo que mi padre se ingenio un soporte a base de tubos PVC al cual le instalo dos ventiladores de un computador mas viejo que yo, ademas de que tuvo que remendarlo seriamente con un montón de cinta para que evitar que se le desprendiera la pantalla, y un dia simplemente decidiera que ya fue suficiente y se inmolara en la mitad de un trabajo importante.

[//]: <> (# TODO: Agregar una imagen del soporte en cuestión.)

> Imagina la cara de mis compañeros de clase cuando debía de llevar esa monstruosidad al colegio.

Al final fue reemplazado por un computador de escritorio, porque su escasa potencia ya no me era suficiente para mis tareas de programación y la renderización de modelos en mi *hobby* de diseño 3D.

Esta bien. Lo admito. Lo reemplace porque ya no me corría War Thunder.

La buena noticia, es que si tienes un computador de escritorio o un portátil viejo que aun funciona, pero no le has dado ningún uso (y no se lo quieres regalar a un niño pobre que seguramente lo necesita mas que tu), estas guardando un tesoro invaluable de capacidad de procesamiento y almacenamiento de datos, listo para cumplir otras tareas, incluso si ya no tiene valor como herramienta de trabajo.

## ¿Porque necesitas un servidor casero?

Esta es una pregunta retorica. Realmente necesitas un servidor casero.

Un servidor no es mas que un dispositivo conectado a Internet con capacidad de procesamiento y almacenamiento de datos, cuyo propósito es ofrecer servicios a otros dispositivos similares, llamados clientes.
