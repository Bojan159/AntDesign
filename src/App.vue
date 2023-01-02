<template>
  <div>
    <a-form layout="vertical">
      <a-form-item label="Marka" name="Marka">
        <a-input v-model:value="state.formdata.marka" />
      </a-form-item>
      <a-form-item label="Model" name="Model">
        <a-input v-model:value="state.formdata.model" /> </a-form-item
      ><a-form-item label="Komponente" name="Komponente">
        <a-input v-model:value="state.formdata.komponente" />
      </a-form-item>
    </a-form>
    <a-button type="primary" @click="submitform">Submit</a-button>
    <a-button type="primary" @click="onReset">Reset</a-button>

    <div class="offset-2" v-if="state.listaLaptopa.length > 0">
      <a-card v-for="laptop in state.listaLaptopa" :key="laptop.id">
        <p>
          Marka: {{ laptop.marka }}
          <a-divider />
        </p>
        <p>Model: {{ laptop.model }} <a-divider /></p>
        <p>Komponente: {{ laptop.komponente }} <a-divider /></p>
        <p>
          <a-button type="primary" @click="deleteLaptope(laptop.id)"
            >Izbrisi</a-button
          >
          <a-button type="primary" @click="setEdit(laptop)">Edit</a-button>
        </p>
      </a-card>
    </div>
    <a-modal v-model:visible="state.edit">
      <a-form name="basic">
        <a-form-item label="Marka" name="Marka">
          <a-input v-model:value="state.editLaptopaPodaci.marka" />
        </a-form-item>
        <a-form-item label="Model" name="Model">
          <a-input
            v-model:value="state.editLaptopaPodaci.model"
          /> </a-form-item
        ><a-form-item label="Komponente" name="Komponente">
          <a-input v-model:value="state.editLaptopaPodaci.komponente" />
        </a-form-item>
      </a-form>
      <template #footer>
        <a-button key="back" @click="state.edit = false">Cancel</a-button>
        <a-button
          key="submit"
          type="primary"
          @click="
            editForm();
            state.edit = false;
          "
          >Submit</a-button
        >
      </template>
    </a-modal>
  </div>
</template>

<script>
import { reactive, onMounted } from "vue";
import {
  addDoc,
  collection,
  query,
  onSnapshot,
  deleteDoc,
  doc,
  updateDoc,
} from "firebase/firestore";
import { db } from "./firebase.js";

export default {
  name: "laptopInfo",
  setup() {
    const state = reactive({
      formdata: {
        marka: "",
        model: "",
        komponente: "",
      },
      poruka: "",
      listaLaptopa: [],
      edit: false,
      editLaptopaPodaci: null,
    });

    async function submitform() {
      await addDoc(collection(db, "laptop"), {
        marka: state.formdata.marka,
        model: state.formdata.model,
        komponente: state.formdata.komponente,
      });
      state.formdata.marka = " ";
      state.formdata.model = " ";
      state.formdata.komponente = " ";
    }
    const editForm = async () => {
      await updateDoc(doc(db, "laptop", state.editLaptopaPodaci.id), {
        marka: state.editLaptopaPodaci.marka,
        model: state.editLaptopaPodaci.model,
        komponente: state.editLaptopaPodaci.komponente,
      });
    };

    let unsubscribe;

    const dohvatiLaptope = async () => {
      const q = query(collection(db, "laptop"));
      unsubscribe = onSnapshot(q, (querySnapshot) => {
        state.listaLaptopa = [];
        querySnapshot.forEach((doc) => {
          let laptop = {
            id: doc.id,
            marka: doc.data().marka,
            model: doc.data().model,
            komponente: doc.data().komponente,
          };

          state.listaLaptopa.push(laptop);
        });
      });
    };

    const deleteLaptope = async (id) => {
      await deleteDoc(doc(db, "laptop", id));
    };

    onMounted(() => {
      dohvatiLaptope();
    });

    const onReset = () => {
      state.formdata.marka = " ";
      state.formdata.model = " ";
      state.formdata.komponente = " ";
    };

    const setEdit = (laptop) => {
      state.edit = true;
      state.editLaptopaPodaci = {
        id: laptop.id,
        marka: laptop.marka,
        model: laptop.model,
        komponente: laptop.komponente,
      };
    };

    return {
      submitform,
      onReset,
      state,
      deleteLaptope,
      editForm,
      setEdit,
    };
  },
};
</script>
