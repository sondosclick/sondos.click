---
layout: post
title: "Métricas equivocadas: por qué medir velocidad nos está llevando al lugar equivocado"
date: 2026-03-15
categories: [IA, Tecnología, Cultura]
tags: [IA, Métricas, Productividad, Complejidad]
description: "Por qué medir velocidad con IA nos aleja del valor y aumenta la complejidad."
---

# Métricas equivocadas: por qué medir velocidad nos está llevando al lugar equivocado

Si algo nos gusta en tecnología casi tanto como una herramienta nueva, es medirla.  
Y si algo nos tranquiliza más que medir, es ver una gráfica subir hacia la derecha.

Da igual qué estemos midiendo.  
Da igual si entendemos exactamente qué significa.  
Si sube, vamos bien.

Con la IA, este reflejo se ha acelerado todavía más.

De repente, todo parece mejorar según las métricas:
más líneas de código por día,  
más tareas cerradas,  
más documentos generados,  
menos tiempo “aparente” hasta entrega.

Los dashboards sonríen.  
La caja negra con sonrisa también.

Y alguien dice la frase mágica:  
> “¿Ves? Funciona.”

El problema es que estamos midiendo lo fácil, no lo importante.

Medimos velocidad porque es visible.  
Medimos output porque es contable.  
Medimos actividad porque se puede representar con una gráfica bonita.

Pero casi nunca medimos:
comprensión,  
mantenibilidad,  
deuda futura,  
coste cognitivo,  
o cuántas veces alguien pensó *“esto no debería ser así”* y siguió adelante.

Eso no cabe bien en un KPI.

Este error no es nuevo.  
Lo llevamos cometiendo décadas, solo que ahora lo estamos automatizando.

Durante años hemos medido a los técnicos de soporte por el número de incidencias resueltas. No por su complejidad, ni por su impacto, ni por si el problema volvió a aparecer. Solo por cuántas se cerraban. El resultado fue previsible: tickets rápidos, problemas estructurales intactos y clientes que volvían a llamar.

También hemos medido a desarrolladores por número de líneas subidas al repositorio. Como si escribir más código fuera sinónimo de aportar más valor. El resultado, otra vez, fue el esperado: código inflado, abstracciones innecesarias y sistemas más difíciles de entender y mantener.

Medir velocidad con IA es exactamente el mismo error, solo que con mejores gráficos.

Hay algo que conviene recordar y que solemos olvidar con demasiada facilidad: **más código no es mejor código**.  
De hecho, suele ser lo contrario.

En seguridad lo entendemos perfectamente.  
Cuanto más software instalas, mayor es la superficie de ataque.  
Más binarios, más dependencias, más configuraciones, más puntos de fallo.

Con el código ocurre lo mismo.

Cada línea adicional:
- es una posible fuente de errores,
- aumenta la complejidad del sistema,
- incrementa el esfuerzo de revisión,
- y eleva el coste de mantenimiento futuro.

La IA es excelente produciendo código rápidamente.  
Pero producir código rápido no equivale a producir sistemas mejores.

Aquí aparece una confusión peligrosa: actividad no es progreso.

La IA multiplica la actividad.  
Genera más código, más variantes, más soluciones plausibles.

Pero en sistemas complejos, más no suele ser mejor.  
Y rápido casi nunca significa eficiente.

Uno de los efectos más perversos de medir solo velocidad es que penaliza el pensamiento.

Pensar no produce output inmediato.  
Cuestionar una solución no cierra tickets.  
Pararse a entender algo no mueve la aguja del sprint.

Así que el sistema aprende rápido:
el que duda, retrasa;  
el que revisa, frena;  
el que pregunta “por qué”, estorba.

Mientras tanto, el que acepta la respuesta de la caja y hace commit rápido parece un héroe.  
Al menos durante un tiempo.

Hay métricas que deberían importarnos mucho más, aunque no queden tan bien en una slide.

Por ejemplo:
- número de iteraciones hasta que un técnico confía en el código generado,
- tiempo real desde el primer commit hasta una versión estable en producción,
- número de retrabajos tras la entrega,
- frecuencia de cambios en los mismos módulos,
- ratio de bugs post-despliegue,
- o cuántos componentes nadie quiere tocar.

Estas métricas no son teóricas. Se pueden recoger.

Las iteraciones se pueden medir en los propios flujos de trabajo: commits descartados, PRs reabiertos, ciclos de revisión. El tiempo hasta estabilidad se puede extraer de los pipelines de CI/CD. Los retrabajos aparecen en el histórico de issues. Los módulos “intocables” se identifican solos: son los que siempre acaban en manos de los mismos.

No es que no sepamos medir estas cosas.  
Es que son incómodas.

Otro error habitual es medir reducción de esfuerzo humano sin medir traslado de esfuerzo.

Si antes escribir código llevaba ocho horas y ahora escribir, revisar, depurar, reinterpretar y volver a revisar lleva diez, pero solo medimos las dos primeras, la IA parece un éxito rotundo.

El esfuerzo no ha desaparecido.  
Se ha vuelto invisible.

Y el esfuerzo invisible siempre acaba pasando factura.

La caja negra con sonrisa contribuye a esta ilusión.  
Responde rápido, con seguridad y sin mostrar dudas. Da la sensación de que el trabajo difícil ya está hecho. Que solo queda “pulir”.

Y entonces empezamos a optimizar para la métrica equivocada.

No para sistemas mejores.  
Para gráficos más bonitos.

Una organización madura no se pregunta solo cuánto más rápido va.  
Se pregunta qué está sacrificando para ir más rápido.

Porque en ingeniería casi siempre se sacrifica algo:
comprensión,  
flexibilidad,  
aprendizaje,  
margen de error.

Si no sabes qué estás sacrificando, no es eficiencia.  
Es suerte.

Quizá la IA nos permita ir mucho más rápido.  
Ojalá.

Pero si solo medimos velocidad, nunca sabremos cuándo hemos cruzado el punto en el que el coste oculto supera al beneficio visible.  
Nunca sabremos cuándo la deuda técnica empieza a crecer más rápido que el valor entregado.  
Nunca sabremos cuándo hemos entrenado a la organización para no pensar.

Y eso no lo arregla ningún modelo nuevo.

Tal vez el problema no sea que la IA escriba demasiado rápido.  
Tal vez el problema sea que seguimos midiendo como si la velocidad fuera el objetivo final.

Y en sistemas que tienen que durar años,  
la velocidad rara vez es lo más importante.

Aunque quede muy bien en una gráfica.

{{< smallnote >}}
**Nota**: Este artículo forma parte de una serie que cuestiona cómo estamos adoptando la IA en entornos profesionales. Medir es necesario. Medir bien, imprescindible. Elegir mal las métricas no solo da información incorrecta: cambia el comportamiento de toda la organización.
{{< /smallnote >}}
