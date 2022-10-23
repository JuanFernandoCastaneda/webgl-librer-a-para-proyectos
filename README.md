# Librería para proyectos

## Orden en el que pasan las cosas en WebGL

Dentro del código hay comentarios para explicar cada cosa que está pasando. Esto está limitado por mi entendimiento actual en el área de conocimiento.

### Shaders

Dentro de WebGL hay dos tipos de shaders: vertex shaders y fragment shaders, y su función se basa en la forma en que se dibujan las cosas dentro de WebGL (y me imagino que otros motores gráficos).

**(Insertar imagen de un cuadradito con solo vértices)**

Las imágenes se pintan a través de vértices conectados por triángulos que WebGL rellena con pixeles. Estos pixeles terminan recibiendo una información promedio de cada uno de los vértices de los que son parte.

El vertex shader es el que determina las propiedades de los vértices. Recibe información que el programador determinó acerca de diferentes propiedades de los vértices (como la posición, el color, etc) para hacer dos cosas: primero, determinar la posición del vértice en cada momento de tiempo, y segundo, pasarle información a los pixeles cercanos, de los triángulos de los que ese vértice hace parte. 

**(Insertar imagen del alcance de cada vértice y su influencia en sus pixeles)**

A cambio, el fragment shader determina las propiedades de los pixeles. Recibe información de las propiedades de sus vértices relacionados (y tal vez alguna otra del programador) para determinar el color del pixel.

Lo primero que se hace es crear los shaders como variables de texto dentro del archivo JS.

El siguiente es un ejemplo de un vertex shader (tener cuidado a la hora de copiar y pegar los comentarios):

```javascript
const vsText = `
    // Esto creo que se refiere a si las variables usadas van a ser enteras o floats.
    precision mediump float; 
    
    // Los atributos que se llaman "attribute" son propios de cada vértice, y los suele pasar
    // el programador en masa.
    attribute vec3 vertPosition; // Por ejemplo, la posición del vértice.
    attribute vec2 vertTextureCoord; // Las coordenadas de las texturas que van a usar el vértice.
    attribute vec3 vertNormal; // Las normal del vértices para cuando hagamos iluminación Phong.
    
    // Los atributos que se llaman "varying" son los que se pasan al fragment shader.
    // Dentro de la función main del shader se manipulan.
    varying vec2 fragTextureCoord; // Por ejemplo, aquí pasamos la información de la textura del vértice.
    varying vec3 fragNormal; // Aquí pasamos la información de la normal del vértice.
    
    // Los atributos con tag "uniform" son comunes para todos los vértices.
    uniform mat4 mWorld;
    uniform mat4 mView; // ¿Qué mondá es esto...?
    uniform mat4 mProjection;

    // Método main donde se ejecuta la lógica del shader.
    void main() {
        fragTextureCoord = vertTextureCoord; // Aquí se transmite la información de la textura.
        fragNormal = (mWorld * vec4(vertNormal, 0.0)).xyz; // Se transmite la información de las normales.
        gl_Position = mProjection * mView * mWorld * vec4(vertPosition, 1.0); // Se determina la posición.
    }`;
```

Ahora un ejemplo de un fragment shader:

```javascript
const fsText = `
    // De nuevo determinar qué tipo de variables vamos a usar.
    precision mediump float;
    
    // Lo que entra por parámetro de los vértices cercanos.
    varying vec2 fragTextureCoord;
    varying vec3 fragNormal;
    
    // Un parámetro que el programador pasó directamente a los pixeles.
    uniform sampler2D sampler; // La imagen que representa la textura.
    
    void main() {
        // Variables "quemadas" que no deberían estar acá, sino que se deberían pasar como "uniform".
        vec3 ambientLightIntensity = vec3(0.2, 0.2, 0.2); 
        vec3 sunlightIntensity = vec3(0.9, 0.9, 0.9);
        vec3 sunlightDirection = normalize(vec3(5.0, 0.0, 5.0));

        // Decimos que el fragmento tiene esa posición (y por tanto color) de la textura.
        vec4 texel = texture2D(sampler, fragTextureCoord);

        // Fórmula que determina la intensidad de la luz en el pixel.
        vec3 lightIntensity = ambientLightIntensity + 
            sunlightIntensity * max(dot(fragNormal, sunlightDirection), 0.0);

        // Fórmula final que determina el color del pixel (involucra la intensidad de la luz, y por
        // tanto de la sombra, y su color de textura).
        gl_FragColor = vec4(texel.rgb * lightIntensity, texel.a);
    }`;
```
