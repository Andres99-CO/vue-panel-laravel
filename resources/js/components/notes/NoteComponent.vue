<template>
  <div>
    <form class="col-md-8 mx-auto" @submit.prevent="edit()" v-if="editActive">
      <h5>Editar tareas</h5>
      <div class="form-group">
        <input type="text" placeholder="Nombre" class="form-control" v-model="note.name" />
      </div>
      <div class="form-group">
        <textarea
          type="text"
          placeholder="Descripción"
          class="form-control"
          v-model="note.description"
        />
      </div>
      <button type="submit" class="btn btn-primary btn-block">Actualizar</button>
      <button type="submit" class="btn btn-danger btn-block" @click="cancel()">Cancelar</button>
    </form>

    <form class="col-md-8 mx-auto" @submit.prevent="add()" v-else>
      <h5>Agregar tareas</h5>

      <div class="form-group">
        <input type="text" placeholder="Nombre" class="form-control" v-model="note.name" />
      </div>
      <div class="form-group">
        <textarea
          type="text"
          placeholder="Descripción"
          class="form-control"
          v-model="note.description"
        />
      </div>
      <button type="submit" class="btn btn-success btn-block">Crear</button>
    </form>
    <paginate ref="paginator" name="notes" :list="notes" :per="5">
      <div class="row mt-4 px-2">
        <ul
          class="list-group col-md-6 p-1"
          v-for="(item, index) in paginated('notes')"
          :key="index"
        >
          <li class="list-group-item">
            <span
              class="badge badge-primary float-right"
            >{{new Date(item.created_at).toLocaleDateString("en-US", {year: 'numeric', month: 'long', day: 'numeric' })}}</span>
            <p>{{item.name}} - {{item.description}}</p>
            <button
              type="button"
              class="btn btn-danger btn-sm"
              @click="deleteNote(item, index)"
            >Eliminar</button>
            <button type="button" class="btn btn-primary btn-sm" @click="editNote(item)">Actualizar</button>
          </li>
        </ul>
      </div>
    </paginate>
    <paginate-links
      for="notes"
      :show-step-links="true"
      :simple="{
                    prev: 'Anterior',
                    next: 'Siguiente'  
                }"
    ></paginate-links>
  </div>
</template>

<script>
export default {
  data() {
    return {
      paginate: ["notes"],
      notes: [],
      note: { name: "", description: "" },
      editActive: false,
    };
  },
  created() {
    axios
      .get("/note")
      .then((res) => {
        this.notes = res.data;
      })
      .catch((err) => console.log(err));
  },
  methods: {
    editNote(item) {
      this.editActive = true;
      this.note.name = item.name;
      this.note.description = item.description;
      this.note.id = item.id;
    },
    async add() {
      if (this.note.name.trim() === "" || this.note.description.trim() === "") {
        alert("Debe llenar todos los campos");
        return;
      }
      console.log(this.note.name, this.note.description);
      const params = this.note;
      try {
        const note = await axios.post("/note", params);
        this.notes.push(note.data);
        this.note.name = "";
        this.note.description = "";
      } catch (error) {
        console.log(error);
      }
    },
    deleteNote(item, index) {
      axios.delete(`/note/${item.id}`).then(() => {
        this.notes.splice(index, 1);
      });
    },
    async edit() {
      if (this.note.name.trim() === "" || this.note.description.trim() === "") {
        alert("Debe llenar todos los campos");
        return;
      }
      console.log(this.note.name, this.note.description);
      const params = this.note;
      try {
        const note = await axios.put(`/note/${this.note.id}`, params);
        const index = this.notes.findIndex((item) => item.id === note.data.id);
        Vue.set(this.notes, index, note.data)
        this.note = { name: "", description: "" };
        this.editActive = false;
      } catch (error) {
        console.log(error);
      }
    },
    cancel() {
      this.note = { name: "", description: "" };
      this.editActive = false;
    },
  },
};
</script>

<style>
.paginate-links {
  width: 100%;
  list-style: none;
  text-align: center;
}
.paginate-links li {
  display: inline;
  background-color: #e43a48;
  color: white;
  padding: 0.5rem;
  margin-left: 0.3rem;
  margin-right: 0.3rem;
  cursor: pointer;
  border-radius: 3px;
}
.paginate-result {
  width: 100%;
  text-align: center;
  margin-bottom: 1rem;
}
</style>