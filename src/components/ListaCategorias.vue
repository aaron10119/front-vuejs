<template>
  <div class="container mt-4">
    <h2 class="text-center mb-4">Lista de Categorias</h2>
    <b-button variant="primary" class="mb-3" @click="mostrarModalAgregar = true">
     Agregar Categoría
    </b-button>

    <b-table striped hover bordered :items="categorias" :fields="fields">
      <template #cell(name)="data">
        <b>{{ data.value }}</b>
      </template>

      <template #cell(actions)="data">
        <b-button variant="warning" size="sm" @click="editarCategoria(data.item)">
          ✏️
        </b-button>
        <b-button variant="danger" size="sm" class="ml-2" @click="eliminarCategoria(data.item.id)">
          🗑
        </b-button>
      </template>
    </b-table>

    <!-- Modal para agregar categoría -->
    <b-modal v-model="mostrarModalAgregar" title="Agregar Categoría" @ok="agregarCategoria">
      <b-form>
        <b-form-group label="Nombre">
          <b-form-input v-model="nuevaCategoria.name"></b-form-input>
        </b-form-group>
      </b-form>
    </b-modal>

    <!-- Modal para editar categoría -->
    <b-modal v-model="mostrarModal" title="Editar Categoría" @ok="guardarCambios">
      <b-form>
        <b-form-group label="Nombre">
          <b-form-input v-model="categoriaSeleccionada.name"></b-form-input>
        </b-form-group>
      </b-form>
    </b-modal>
  </div>
</template>

<script>
export default {
  data() {
    return {
      categorias: [],
      nuevaCategoria: { name: "" },
      categoriaSeleccionada: { id: null, name: "" },
      mostrarModal: false,
      mostrarModalAgregar: false,
      fields: [
        { key: "name", label: "Nombre" },
        { key: "actions", label: "Acciones" }
      ]
    };
  },
  mounted() {
    this.obtenerCategorias();
  },
  methods: {
    async obtenerCategorias() {
      try {
        const response = await fetch("http://127.0.0.1:8000/api/categories");
        if (!response.ok) throw new Error("Error al obtener categorías");
        this.categorias = await response.json();
      } catch (error) {
        console.error(error);
      }
    },
    async agregarCategoria() {
      try {
        const response = await fetch("http://127.0.0.1:8000/api/categories", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(this.nuevaCategoria)
        });

        if (response.ok) {
          const data = await response.json();
          this.categorias.push(data);
          this.mostrarModalAgregar = false;
          this.nuevaCategoria = { name: "" };
          this.obtenerCategorias();
        } else {
          console.error("Error al agregar categoría");
        }
      } catch (error) {
        console.error(error);
      }
    },
    editarCategoria(categoria) {
      this.categoriaSeleccionada = { ...categoria };
      this.mostrarModal = true;
    },
    async guardarCambios() {
      try {
        const response = await fetch(
          `http://127.0.0.1:8000/api/categories/${this.categoriaSeleccionada.id}`,
          {
            method: "PUT",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(this.categoriaSeleccionada)
          }
        );

        if (response.ok) {
          const index = this.categorias.findIndex(c => c.id === this.categoriaSeleccionada.id);
          this.categorias[index] = { ...this.categoriaSeleccionada };
          this.mostrarModal = false;
          this.obtenerCategorias();
        } else {
          console.error("Error al actualizar categoría");
        }
      } catch (error) {
        console.error(error);
      }
    },

    async eliminarCategoria(id) {
      if (!confirm("¿Seguro que quieres eliminar esta categoría?")) return;

      try {
        const response = await fetch(`http://127.0.0.1:8000/api/categories/${id}`, {
          method: "DELETE"
        });

        if (response.ok) {
          this.categorias = this.categorias.filter(c => c.id !== id);
        } else {
          console.error("Error al eliminar categoría");
        }
      } catch (error) {
        console.error(error);
      }
    }
  }
};
</script>
