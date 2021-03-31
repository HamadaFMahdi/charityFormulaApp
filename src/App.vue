<template>
    <v-app>
      <v-main>
        <NavBar />
        <v-row class="mt-6">
          <v-col cols="12" md="8" offset-md="2">
            <v-card outlined class="pa-12">
              <v-container>
                <v-form
                  ref="form"
                  v-model="valid"
                >
                  <v-row>
                    <v-col cols="12" md="12">
                      <v-text-field
                        v-model="name"
                        outlined
                        :rules="[v => !!v || 'Required']"
                        label="Project name"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" md="6">
                      <v-select
                          outlined
                          v-model="type"
                          :items="typeOptions"
                          item-text="option"
                          return-object
                          :rules="[v => !!v || 'Required']"
                          label="Charity type"
                          :hint="`Weighting: ${typeOptions[1].weight*100}%`"
                          persistent-hint
                        ></v-select>
                    </v-col>
                    <v-col cols="12" md="6">
                      <v-select
                        outlined
                        v-model="longevity"
                        :items="longevityOptions"
                        item-text="option"
                        return-object
                        :rules="[v => !!v || 'Required']"
                        label="Longevity"
                        :hint="`Weighting: ${longevityOptions[0].weight*100}%`"
                        persistent-hint
                      ></v-select>
                    </v-col>
                    <v-col cols="12" md="6">
                      <v-select
                        outlined
                        v-model="people"
                        :items="peopleOptions"
                        item-text="option"
                        return-object
                        :rules="[v => !!v || 'Required']"
                        label="Number of beneficiaries"
                        :hint="`Weighting: ${peopleOptions[0].weight*100}%`"
                        persistent-hint
                      ></v-select>
                    </v-col>
                    <v-col cols="12" md="6">
                      <v-select
                        outlined
                        v-model="cost"
                        :items="costOptions"
                        item-text="option"
                        return-object
                        :rules="[v => !!v || 'Required']"
                        label="Cost"
                        :hint="`Weighting: ${costOptions[0].weight*100}%`"
                        persistent-hint
                      ></v-select>
                    </v-col>
                    <v-col align="center" cols="12">
                      <v-sheet
                        class="score"
                        :color="getScoreColor(calc.score)"
                        elevation="3"
                        height="111"
                        width="229"
                      >
                        <animated-number
                          :value="calc.score"
                          :duration="700"
                          :formatValue="formatToPrice"
                        />
                      </v-sheet>
                    </v-col>
                    <v-col style="text-align: center;" cols="12">
                      <v-btn
                        x-large
                        :disabled="!valid"
                        :loading="loading"
                        color="success"
                        @click="save"
                      >
                        Save project
                      </v-btn>
                    </v-col>
                  </v-row>
                </v-form>
              </v-container>
            </v-card>
          </v-col>
        </v-row>
        <v-row class="mt-6 mx-5">
          <v-col cols="12" >
            <v-card outlined class="pa-12">
              <v-container>
                <v-data-table
                  :headers="headers"
                  :items="projects"
                  caption="Saved Projects"
                ></v-data-table>
              </v-container>
            </v-card>
          </v-col>
        </v-row>
        <v-snackbar
          v-model="snackbarShow"
          top right 
          color="error"
        >
          {{snackbarText}}
        </v-snackbar>
      </v-main>
    </v-app>
</template>

<script>
import NavBar from "./components/NavBar";
import AnimatedNumber from "animated-number-vue";

export default {
  name: "App",
  components: {
    NavBar,
    AnimatedNumber
  },
  data: () => ({
    headers: [
      {
        text: 'Name',
        align: 'start',
        sortable: false,
        value: 'name',
      },
      { text: 'Charity Type', value: 'type' },
      { text: 'Longevity', value: 'longevity' },
      { text: 'Beneficiaries', value: 'people' },
      { text: 'Cost', value: 'cost' },
      { text: 'Created', value: 'created' },
      { text: 'Score', value: 'score' },
    ],
    valid: true,
    loading: false,
    rules: [
      v => !!v || 'Required',
    ],
    name: '',
    type: '',
    typeOptions: [
      { header: 'Class A' },
      {option: 'Masjid Construction', score: 5, weight: 0.2},
      {option: 'Water well', score: 5, weight: 0.2},
      {option: 'Orphanage', score: 5, weight: 0.2},
      {option: 'Islamic center', score: 5, weight: 0.2},
      { divider: true },
      { header: 'Class B' },
      {option: 'Orphan sponsorship', score: 4, weight: 0.2},
      {option: 'Hifdh sponsorship', score: 4, weight: 0.2},
      { divider: true },
      { header: 'Class C' },
      {option: 'Mushaf handouts', score: 3, weight: 0.2},
      {option: 'Other', score: 3, weight: 0.2},
    ],
    longevity: '',
    longevityOptions: [
      {option: '0-6 Months', score: 1, weight: 0.35},
      {option: '6-12 Months', score: 2, weight: 0.35},
      {option: '1-2 years', score: 3, weight: 0.35},
      {option: '2-5 years', score: 4, weight: 0.35},
      {option: '5+ year', score: 5, weight: 0.35},
    ],
    people: '',
    peopleOptions: [
      {option: '1-10 people', score: 1, weight: 0.35},
      {option: '11-100 people', score: 2, weight: 0.35},
      {option: '101-500 people', score: 3, weight: 0.35},
      {option: '501-1000 people', score: 4, weight: 0.35},
      {option: '1000+ people', score: 5, weight: 0.35},
    ],
    cost: '',
    costOptions: [
      {option: '> £10000', score: 1, weight: 0.1},
      {option: '£5001 - £10000', score: 2, weight: 0.1},
      {option: '£3001 - £5000', score: 3, weight: 0.1},
      {option: '£1000 - £3000', score: 4, weight: 0.1},
      {option: '< £1000', score: 5, weight: 0.1},
    ],
    finalScore: null,
    projects: [],
    snackbarShow: false,
    snackbarText: ''
  }),
  mounted() {
    this.fetchProjects()
  },
  computed: {
    calc() {
      if(this.valid) {
        let vars = [
          this.type,
          this.longevity,
          this.people,
          this.cost,
        ]

        let sumOfWeightedScore = vars.map(x => x.score * x.weight).reduce((x,y) => x+y)
        console.log(sumOfWeightedScore)
        let sumOfWeights = vars.map(x => x.weight).reduce((x,y) => x+y)
        console.log(sumOfWeights)
        let finalScore = (sumOfWeightedScore / sumOfWeights).toFixed(1)
        console.log(typeof finalScore)
        // formula: sum weigh * score / sum weight
        return {
          name: this.name.option,
          type: this.type.option,
          longevity: this.longevity.option,
          people: this.people.option,
          cost: this.cost.option,
          score: finalScore
        }
      } else {
        return {score: 0}
      }
    },
  },
  methods: {
    formatToPrice(value) {
      return `${value.toFixed(1)}`;
    },
    getScoreColor(score) {
      if(score >= 4) {
        return "primary"
      } else if(score >= 2) {
        return "orange" 
      } else if(score >= 0) {
        return "error" 
      }
    },
    save() {
      // save to memory
      let currentProjects = JSON.parse(localStorage.getItem("projects"))
      // check name doesn't already exist
      let check = currentProjects.find(x => x.name === this.name)
      console.log(check)
      if(!check) {
        currentProjects.push({
          ...this.calc,
          created: new Date().toLocaleString(),
          name: this.name
        })
        localStorage.setItem("projects",JSON.stringify(currentProjects))
        this.fetchProjects()
      } else {
        this.snackbarText = 'Project exists'
        this.snackbarShow = true
      }
    },
    fetchProjects() {
      if(!localStorage.getItem("projects")){
        localStorage.setItem("projects", "[]")
      } else {
        this.projects = JSON.parse(localStorage.projects)
      }
    }
  },
};
</script>

<style scoped>
  .score {
    font-size: 4em;
    font-family: monospace;
    color: white;
  }
</style>
