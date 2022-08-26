<template>
  <div class="my-card">
    <q-btn @click="addObj = true" push>Adauga Obiectiv</q-btn>
    <q-dialog v-model="addObj">
      <div class="my-popup">
      <q-card>
        <h6 style="margin:0px 0px 0px 15px">Obiectiv</h6>
        <q-input style="margin:0px 15px" v-model="titleInput" label="Titlu" />
        <q-input style="margin:0px 15px" v-model="urlPhoto" label="Poza" />
        <q-select style="margin:0px 15px" v-model="selectedCity" :options="filteredCities" label="Oras" />
        <q-btn style="margin:15px" v-on:click="addObjective" color="green">Salveaza</q-btn>
      </q-card>
      </div>
    </q-dialog>
    <h5 style="margin-top: 15px"><b>Obiective</b></h5>
    <div style="margin-bottom: 50px" class="row">
      <div class="col">
        <q-select
          @update:model-value="sortObjectives"
          v-model="selectedSort"
          :options="sortOptions"
          label="Sorteaza dupa"
        />
      </div>

      <div class="col">
        <q-select
          v-model="selectedCityFilter"
          :options="citiesResponseFilterOptions"
          label="Filtreaza dupa oras"
        />
      </div>
    </div>
    <div
      class="my-row"
      v-for="(objective, index) in filterObjectivesByCity"
      v-bind:key="index"
    >
      <q-img
        style="border-radius: 10px; width: 10em; height: 6em"
        :src="objective.urlPhoto"
      >
      <q-popup-edit v-model="objective.urlPhoto" auto-save v-slot="scope">
        <q-input v-model="scope.value" dense autofocus counter @keyup.enter="scope.set" />
      </q-popup-edit>
      </q-img>


      <q-input v-model="objective.title" label="Titlu" />
      <q-select v-model="objective.selectedCity" :options="filteredCities" label="Oras" />
      <q-checkbox label="Vizitat" v-model="objective.checkbox" />
      <q-rating
        v-model="objective.rating"
        :max="5"
        size="3.5em"
        color="green-5"
        :icon="icons"
      />

      <q-btn v-on:click="removeObjective(index)" style="margin-left:50px" color="red">Sterge</q-btn>
      <q-btn v-on:click="updateObjective(objective)" style="margin-left:10px" color="blue">Salveaza</q-btn>
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
      citiesResponseFilterOptions: null,
      rating: 0,
      message: "",
      selectedCity: "",
      objectives: LocalStorage.getItem("objectives") || [],
      selectedCityFilter: "Selecteaza oras",
      selectedSort: "Nou",
      sortOptions: ["Nou", "Vechi", "Rating Mare", "Rating Mic"],
    };
  },
  methods: {
    addObjective() {
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
      this.titleInput = "";
      this.urlPhoto = "";
      this.selectedCity = "";
      this.addObj = false;
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
    sortObjectives(selectedSort) {
      switch (selectedSort) {
        case "Nou":
          this.objectives.sort((first, second) => {
            return first.id < second.id ? 1 : -1;
          });
          break;
        case "Vechi":
          this.objectives.sort((first, second) => {
            return first.id > second.id ? 1 : -1;
          });
          break;
        case "Rating Mare":
          this.objectives.sort((first, second) => {
            return first.rating < second.rating ? 1 : -1;
          });
          break;
        case "Rating Mic":
          this.objectives.sort((first, second) => {
            return first.rating > second.rating ? 1 : -1;
          });
          break;
      }
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
      this.citiesResponseFilterOptions = ["Selecteaza oras"];
      this.citiesResponse.data.forEach((city) => {
        this.citiesResponseFilterOptions.push(city.city);
      });
    });
  },
  computed: {
    filteredCities: function () {
      if (this.citiesResponse) {
        return this.citiesResponse.data.map((element) => {
          return element.city;
        });
      }
      return ["Bucuresti", "Cluj", "Iasi", "Timisoara"];
    },
    filterObjectivesByCity() {
      if (this.selectedCityFilter == "Selecteaza oras") {
        return this.objectives;
      }
      return this.objectives.filter((objective) => {
        return (
          objective.selectedCity.toLowerCase() == this.selectedCityFilter.toLowerCase()
        );
      });
    },
    objectivesComputed: function () {
      var x = LocalStorage.getItem("objectives") || [];
      return x;
    },
  },
};
</script>

<style lang="scss" scoped>
.my-row {
  margin-top: 20px;
}
.my-popup {
  width:100%;
  padding:30px;
  border-radius:10px;
  background-color: white;
}
.my-card {
  margin: 50px auto 50px;
  width: 100%;

  max-width: 760px;
  box-shadow: 0px 0px 3px grey;
  border-radius: 1%;
  padding: 1rem;
}
</style>
