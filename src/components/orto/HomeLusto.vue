<template>
  <div class="ortoBox">
    <div class="ortoForm">
      <v-form ref="form" v-model="valid" lazy-validation>
        <v-select
          v-model="selected.zona"
          :items="zone"
          :rules="[(v) => !!v || 'Item is required']"
          label="Zone"
          required
        ></v-select>

        <v-select
          v-model="selected.luce"
          :items="luci"
          label="Disponibilità luce"
        ></v-select>

        <v-select
          v-model="selected.acqua"
          :items="acque"
          label="Disponibilità acqua"
        ></v-select>

        <v-select
          v-model="selected.dimensione"
          :items="dimensioni"
          label="DImensione terreno"
        ></v-select>

        <v-select
          v-model="selected.tempo"
          :items="tempi"
          label="Tempo ore settimanali"
        ></v-select>

        <v-select
          v-model="selected.budget"
          :items="budgets"
          label="Budget annuo"
        ></v-select>

        <!-- <v-checkbox
          v-model="checkbox"
          :rules="[(v) => !!v || 'You must agree to continue!']"
          label="Do you agree?"
          required
        ></v-checkbox> -->

        <v-btn
          :disabled="!valid"
          color="success"
          class="mr-4"
          @click="filterOrtaggi"
        >
          Cerca
        </v-btn>
      </v-form>
    </div>

    <table class="table mt-4">
      <thead>
        <tr>
          <th scope="col">nome</th>
          <th scope="col">priorità</th>
          <th scope="col">stagione</th>
          <th scope="col">zona</th>
          <th scope="col">luce</th>
          <th scope="col">acqua</th>
          <th scope="col">ph</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(ortaggio, i) in ortaggi" :key="i + '_ortaggio'" class="">
          <td class="">{{ ortaggio.title.rendered }}</td>
          <td
            v-for="(cf, i) in ortaggio.acf.ortaggi_fields"
            :key="i + '_ortaggio_cf'"
          >
            <span v-if="cf !== ''">{{ cf }}</span>
          </td>
        </tr>
      </tbody>
    </table>

    <div class="filteredValues">filtered values</div>
    <table class="table mt-4">
      <thead>
        <tr>
          <th scope="col">nome</th>
          <th scope="col">priorità</th>
          <th scope="col">stagione</th>
          <th scope="col">zona</th>
          <th scope="col">luce</th>
          <th scope="col">acqua</th>
          <th scope="col">ph</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(ortaggio, i) in ortaggi_filtered"
          :key="i + '_ortaggio'"
          class=""
        >
          <td class="">{{ ortaggio.title.rendered }}</td>
          <td
            v-for="(cf, i) in ortaggio.acf.ortaggi_fields"
            :key="i + '_ortaggio_cf'"
          >
            {{ cf }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<script>
import axios from "axios";
import _ from "lodash";

export default {
  data() {
    return {
      loaded: false,
      ortaggi: null,
      ortaggi_filtered: null,
      valid: true,
      checkbox: false,
      selected: {
        zona: null,
        acqua: null,
        luce: null,
        dimensione: null,
        tempo: null,
        budget: null,
      },
      zone: [
        "ligure-tirrenica",
        "mediterranea",
        "pianura padana",
        "adriatica",
        "alpina",
        "appennino",
      ],
      acque: ["normale", "scarsa"],
      luci: ["normale", "scarsa"],
      dimensioni: ["0-25mq", "26-50mq", "50-75mq"],
      tempi: ["1", "1-3", "3-6", "6-10", ">10"],
      budgets: ["0-100", "100-200", "200-300", "300-400", ">400"],
    };
  },
  created() {
    this.getOrtaggi();
  },
  methods: {
    filterOrtaggi() {
      this.validate();
      var arr = this.ortaggi;
      //   var filtered = _.filter(arr, {
      //     zona: this.selected.zona,
      //     // acqua: this.selected.acqua,
      //     // luce: this.selected.luce,
      //   });
      var filtered = _.filter(arr, (or) => {
        return (
          or.acf.ortaggi_fields.zona == this.selected.zona &&
          or.acf.ortaggi_fields.classificazione_acqua == this.selected.acqua &&
          or.acf.ortaggi_fields.categoria_luce == this.selected.luce
        );
      });
      console.log(filtered);
      this.ortaggi_filtered = filtered;
    },
    async getOrtaggi() {
      this.loaded = false;
      try {
        var ortaggi = (
          await axios.get(
            //in alternativa posso usare la mia api custom:
            // `https://endorphinoutdoor.com/wp-json/customendpoint/v1/posts-with-cf`
            `https://endorphinoutdoor.com/wp-json/wp/v2/posts?categories=57`
          )
        ).data;

        // elimino campi vuoti acf
        ortaggi.forEach((or) => {
          for (const [key, value] of Object.entries(or.acf.ortaggi_fields)) {
            if (value == "") {
              delete or.acf.ortaggi_fields[key];
            }
          }
        });

        this.ortaggi = ortaggi;
      } catch (error) {
        console.log(error);
      }

      this.loaded = true;
    },
    validate() {
      this.$refs.form.validate();
    },
    reset() {
      this.$refs.form.reset();
    },
    resetValidation() {
      this.$refs.form.resetValidation();
    },
  },
};
</script>
<style lang="scss">
.ortoBox {
  padding: 100px;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
.ortoRow {
  width: 100%;
  display: flex;
  justify-content: space-around;
  align-items: center;
  flex-direction: row;
}
.filteredValues {
  width: 100%;
  display: flex;
  justify-content: flex-start;
  font-size: 25px;
  font-weight: bold;
}
</style>