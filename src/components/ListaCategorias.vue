<template>
  <div class="container mt-4">
    <h2 class="text-center mb-4">Lista de Categorias</h2>
    <b-button variant="primary" class="mb-3" @click="mostrarModalAgregar = true">
     Agregar Categoría
    </b-button>

    <div>
    <b-table
      striped
      hover
      bordered
      :items="categorias"
      :fields="fields"
      :per-page="perPage"
      :current-page="currentPage"
      @update:current-page="onPageChange"
    >
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

    <!-- Paginación -->
    <b-pagination
      v-model="currentPage"
      :total-rows="totalRows"
      :per-page="perPage"
      align="center"
      class="mt-3"
    />
  </div>

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
  import Swal from 'sweetalert2';

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
      ],
      currentPage: 1, 
      perPage: 8, 
      totalRows: 0 
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
        const data = await response.json();
        this.categorias = data; // Asignar los datos a la tabla
        this.totalRows = data.length; // Total de filas para la paginación
      } catch (error) {
        console.error(error);
      }
    },

    onPageChange(newPage) {
      this.currentPage = newPage;
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

      Swal.fire({
        icon: 'success',
        title: '¡Categoría agregada!',
        text: 'La categoría se ha agregado correctamente.',
      });
    } else {
      console.error("Error al agregar categoría");

      Swal.fire({
        icon: 'error',
        title: '¡Error!',
        text: 'Hubo un problema con la solicitud. Intenta de nuevo.',
      });
    }
  } catch (error) {
    console.error(error);

    Swal.fire({
      icon: 'error',
      title: '¡Error!',
      text: 'No se permiten espacios, numeros y caracteres especiales',
    });
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

      Swal.fire({
        icon: 'success',
        title: '¡Categoría actualizada!',
        text: 'La categoría se ha actualizado correctamente.',
      });
    } else {
      console.error("Error al actualizar categoría");

      Swal.fire({
        icon: 'error',
        title: '¡Error!',
        text: 'Hubo un problema con la solicitud. Intenta de nuevo.',
      });
    }
  } catch (error) {
    console.error(error);

    Swal.fire({
      icon: 'error',
      title: '¡Error!',
      text: 'No se permiten espacios, numeros y caracteres especiales',
    });
  }
},

async eliminarCategoria(id) {
  const result = await Swal.fire({
    title: '¿Estás seguro?',
    text: "¡No podrás revertir esta acción!",
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#3085d6',
    cancelButtonColor: '#d33',
    confirmButtonText: 'Sí, eliminarla!',
    cancelButtonText: 'Cancelar'
  });

  if (result.isConfirmed) {
    try {
      const response = await fetch(`http://127.0.0.1:8000/api/categories/${id}`, {
        method: "DELETE"
      });

      if (response.ok) {
        this.categorias = this.categorias.filter(c => c.id !== id);

        Swal.fire({
          icon: 'success',
          title: '¡Categoría eliminada!',
          text: 'La categoría ha sido eliminada correctamente.',
        });
      } else {
        console.error("Error al eliminar categoría");

        Swal.fire({
          icon: 'error',
          title: '¡Error!',
          text: 'Hubo un problema al eliminar la categoría. Inténtalo de nuevo.',
        });
      }
    } catch (error) {
      console.error(error);

      Swal.fire({
        icon: 'error',
        title: '¡Error!',
        text: 'Hubo un problema con la solicitud. Intenta de nuevo.',
      });
    }
  }
}



  }
};
</script>
