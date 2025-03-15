
# Proyecto VUEjs FRONT REST

Este proyecto utiliza Vue.js para consumir una API de Laravel.

## Pasos para iniciar

### 1. Clona el repositorio de Git:
```
   git clone <URL_DEL_REPOSITORIO>
```

### 2. Ejecuta el proyecto con el siguiente comando:

```
   npm run serve
```
### 3. En la carpeta de components encontrarás las funciones fetch para consumir la API de Laravel. Ejemplo de uso:

```
methods: {
  async obtenerCategorias() {
    try {
      const response = await fetch("http://127.0.0.1:8000/api/categories");
      if (!response.ok) throw new Error("Error al obtener categorías");
      const data = await response.json();
      this.categorias = data; // Asignar los datos a la tabla
      this.totalRows = data.length; // Total de filas para la paginación
    } catch (error) {
      console.error(error);
    }
  },
}
```

De ser necesario, cambia el DNS http://127.0.0.1:8000/api/categories a la URL correcta de tu API.


NOTA: Se estarán realizando cambios en los commits para encontrar errores, mejorar el código o agregar contenido adicional al proyecto.

### Por Aaron Hernandez Bueno