<template>
  <div>
    <q-btn @click="addObj = true" push>Adauga</q-btn>
    <q-dialog v-model="addObj">
      <q-card>
        <q-btn v-on:click="addObjective" color="green">Salveaza</q-btn>
        <q-input v-model="titleInput" label="Titlu" />
        <q-input v-model="urlPhoto" label="Poza" />
        <q-select v-model="selectedCity" :options="filteredCities" label="Standard" />
      </q-card>
    </q-dialog>
    <p><b>Obiective</b></p>

    <div v-for="(objective, index) in objectives" v-bind:key="index">
      <q-btn v-on:click="removeObjective(index)" color="red">Sterge</q-btn>
      <q-btn v-on:click="updateObjective(objective)" color="blue">Salveaza</q-btn>
      <!-- <q-btn v-on:click="updateObjective(objective.id,objective.title,objective.)" color="blue">Salveaza</q-btn> -->

      <q-input v-model="objective.title" label="Titlu" />
      <q-select
        v-model="objective.selectedCity"
        :options="filteredCities"
        label="Standard"
      />
      <q-checkbox label="Vizitat" v-model="objective.checkbox" />
      <q-rating
        v-model="objective.rating"
        :max="5"
        size="3.5em"
        color="green-5"
        :icon="icons"
      />
      <q-img
        style="border-radius: 10px; width: 10em; height: 6em"
        src="https://picsum.photos/200/300"
      />
    </div>
  </div>
</template>

<script>
import { LocalStorage } from "quasar";
import axios from "axios";
export default {
  name: "MainPage",
  data() {
    return {
      addObj: false,
      id: 0,
      checkbox: false,
      urlPhoto: "",
      titleInput: "",
      citiesResponse: null,
      rating: 0,
      message: "",
      selectedCity: "",
      objectives: LocalStorage.getItem("objectives") || [],
    };
  },
  methods: {
    addObjective() {
      //add to local storage the new objective as an object with strigified data
      //check if local storage exists
      let lastId = 0;
      if (this.objectives.length > 0) {
        lastId = this.objectives[this.objectives.length - 1].id + 1;
      }
      let newObjective = {
        id: lastId,
        title: this.titleInput,
        urlPhoto: this.urlPhoto,
        selectedCity: this.selectedCity,
        checkbox: false,
        rating: 0,
      };
      this.objectives.push(newObjective);
      LocalStorage.set("objectives", this.objectives);

      console.log("addObjective");
    },
    updateObjective(newObjective) {
      let toUpdateObjective = this.objectives.find(
        (objective) => objective.id == newObjective.id
      );
      toUpdateObjective = newObjective;
      LocalStorage.set("objectives", this.objectives);
    },
    removeObjective(index) {
      this.objectives.splice(index, 1);
      LocalStorage.set("objectives", this.objectives);
      console.log("removeObjective");
    },
  },
  mounted() {
    let romanianCountries =
      "https://countriesnow.space/api/v0.1/countries/population/cities/filter";
    let postData = {
      limit: 10,
      order: "asc",
      orderBy: "name",
      country: "romania",
    };

    axios.post(romanianCountries, postData).then((response) => {
      this.citiesResponse = response.data;
    });
  },
  computed: {
    filteredCities: function () {
      if (this.citiesResponse) {
        return this.citiesResponse.data.map((element) => {
          return element.city;
        });
        return this.cities;
      }
      return ["Bucuresti", "Cluj", "Iasi", "Timisoara"];
    },
    objectivesComputed: function () {
      var x = LocalStorage.getItem("objectives") || [];
      var y = x;
      return x;
    },
  },
};
</script>

<style lang="scss" scoped>
.my-card {
  width: 100%;
  max-width: 760px;
}
</style>
