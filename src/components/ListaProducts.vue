<template>
    <div class="container mt-4">
      <h2 class="text-center mb-4">Lista de Productos</h2>
      <b-button variant="primary" class="mb-3" @click="mostrarFormularioProducto">Agregar Producto</b-button>
  
      <b-table striped hover bordered :items="productos" :fields="fields">
        <template #cell(name)="data">
          <b>{{ data.value }}</b>
        </template>
  
        <template #cell(category)="data">
          <span>{{ data.value }}</span>
        </template>
  
        <template #cell(stock)="data">
          <span :class="{'text-danger': data.value < 3, 'text-success': data.value >= 3}">
            {{ data.value }}
          </span>
        </template>
  
        <template #cell(actions)="data">
          <b-button variant="warning" size="sm" @click="editarProducto(data.item)">
            ✏️
          </b-button>
          <b-button variant="danger" size="sm" class="ml-2" @click="eliminarProducto(data.item.id)">
            🗑
          </b-button>
        </template>
      </b-table>
  
      <b-modal v-model="mostrarModalAgregar" title="Agregar Producto" @ok="agregarProducto">
        <b-form>
          <b-form-group label="Nombre">
            <b-form-input v-model="nuevoProducto.name"></b-form-input>
          </b-form-group>
  
          <b-form-group label="Stock">
            <b-form-input type="number" v-model="nuevoProducto.stock"></b-form-input>
          </b-form-group>
  
          <b-form-group label="Categoría">
            <b-form-select v-model="nuevoProducto.category_id" :options="categorias"></b-form-select>
          </b-form-group>
        </b-form>
      </b-modal>
  
      <b-modal v-model="mostrarModal" title="Editar Producto" @ok="guardarCambios">
        <b-form>
          <b-form-group label="Nombre">
            <b-form-input v-model="productoSeleccionado.name"></b-form-input>
          </b-form-group>
  
          <b-form-group label="Stock">
            <b-form-input type="number" v-model="productoSeleccionado.stock"></b-form-input>
          </b-form-group>
        </b-form>
      </b-modal>
    </div>
  </template>
  

  
  <script>
  export default {
    name: "ListaProducts",
    data() {
      return {
        productos: [], 
        productoSeleccionado: {}, 
        nuevoProducto: { name: "", stock: 0, category_id: null },
        mostrarModal: false, 
        mostrarModalAgregar: false,
        categorias: [], 
        fields: [
          { key: 'name', label: 'Nombre del Producto' },
          { key: 'category', label: 'Categoría' },
          { key: 'stock', label: 'Stock' },
          { key: 'actions', label: 'Acciones' } 
        ]
      };
    },
    mounted() {
      this.obtenerProductos();
      this.obtenerCategorias();
    },
    methods: {
      async obtenerProductos() {
        try {
          const response = await fetch("http://127.0.0.1:8000/api/products/categories");
          const data = await response.json();
          this.productos = data.map(producto => ({
            id: producto.id,
            name: producto.name,
            category: producto.category.name,
            stock: producto.stock
          }));
        } catch (error) {
          console.error("Error al obtener productos:", error);
        }
      },
  
      async obtenerCategorias() {
        try {
          const response = await fetch("http://127.0.0.1:8000/api/categories");
          const data = await response.json();
          this.categorias = data.map(category => ({
            value: category.id,
            text: category.name
          }));
        } catch (error) {
          console.error("Error al obtener categorías:", error);
        }
      },
  
      editarProducto(producto) {
        this.productoSeleccionado = { ...producto }; 
        this.mostrarModal = true;
      },
  
      async guardarCambios() {
        try {
          const response = await fetch(
            `http://127.0.0.1:8000/api/products/${this.productoSeleccionado.id}`, 
            {
            method: "PUT",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
              name: this.productoSeleccionado.name,
              stock: this.productoSeleccionado.stock
            })
          });
  
          if (response.ok) {
      const index = this.productos.findIndex(p => p.id === this.productoSeleccionado.id);
      this.productos[index] = { ...this.productoSeleccionado };
      
      const categoriaIndex = this.categorias.findIndex(c => c.id === this.categoriaSeleccionada.id);
      if (categoriaIndex !== -1) {
        this.categorias[categoriaIndex] = { ...this.categoriaSeleccionada };
      }

      this.mostrarModal = false;
      this.obtenerProductos();
      this.obtenerCategorias();
    } else {
            console.error("Error al actualizar producto");
          }
        } catch (error) {
          console.error("Error en la solicitud PUT:", error);
        }
      },
  
      async eliminarProducto(id) {
        if (!confirm("¿Estás seguro de eliminar este producto?")) return;
  
        try {
          const response = await fetch(`http://127.0.0.1:8000/api/products/${id}`, {
            method: "DELETE"
          });
  
          if (response.ok) {
            this.productos = this.productos.filter(producto => producto.id !== id);
          } else {
            console.error("Error al eliminar producto");
          }
        } catch (error) {
          console.error("Error en la solicitud DELETE:", error);
        }
      },
  
      mostrarFormularioProducto() {
        this.nuevoProducto = { name: "", stock: 0, category_id: null }; 
        this.mostrarModalAgregar = true; 
      },
  


    async agregarProducto() {
  try {
    console.log("Ejecutando agregarProducto...");

    const response = await fetch("http://127.0.0.1:8000/api/products", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(this.nuevoProducto)
    });

    if (response.ok) {
      const data = await response.json();
      console.log("Producto agregado correctamente:", data);

      // Mostrar un mensaje de éxito al usuario (puedes usar alert o un mensaje en la interfaz)
      this.mostrarMensaje("Producto agregado correctamente", "success");
      
      this.obtenerProductos();
      this.mostrarModalAgregar = false;
      this.nuevoProducto = { name: "", stock: 0, category_id: null };
    } else {
      const errorData = await response.json();
      console.error("Error al agregar producto, código:", response.status, errorData.message);

      // Mostrar un mensaje de error al usuario
      this.mostrarMensaje(errorData.message || "Error al agregar producto", "error");
    }
  } catch (error) {
    console.error("Error en la solicitud POST:", error);

    // Mostrar un mensaje de error general al usuario
    this.mostrarMensaje("Error en la solicitud. Intenta de nuevo.", "error");
  }
},

// Método para mostrar un mensaje al usuario (puedes mejorar esta lógica si prefieres mostrarlo en la interfaz)
mostrarMensaje(mensaje, tipo) {
  this.mensaje = mensaje;
  this.tipoMensaje = tipo;  // "success" o "error"
  setTimeout(() => {
    this.mensaje = ''; // Limpiar el mensaje después de 3 segundos
  }, 3000);
}





    }
  };
  </script>
  
<style scoped>
.container {
  max-width: 900px;
}
</style>