cada microfrontend tiene un ciclo de vida independiente
cada microfrontend corre aislado

* durante el desarrollo de un microfrontend, nosotros podemos tener un public/index.html y usar webpackdevserver para levantar nuestro microfrontend aisladamente (sin tener que levantar todo)
* en producción, va a existir el container, que tendra su index.html y levantará todos los microfrontends usando module federation (build-time modules) o ES Modules(in-browser modules).

## compartir dependencias



# TODO

* variables de entorno
* compartir dependencias entre microfronts