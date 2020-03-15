<template>
  <div class="hello">
    <h1>{{ country }}</h1>

    <div v-if="current.matches('success')">
      Success!
      <p>Confirmed: {{ context.countryData.confirmed }}</p>
      <p>Recovered: {{ context.countryData.recovered }}</p>
      <p>Deaths: {{ context.countryData.deaths }}</p>
      <p>Date: {{ context.countryData.date }}</p>
    </div>
    <div v-else-if="current.matches('failure')">
      Error!
      <p>{{ context.error }}</p>
    </div>
    <div v-if="!current.matches('success')">
      <button @click="send('FETCH')">
        {{ current.matches("loading") ? "Loading..." : "Get data" }}
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { Machine } from "xstate";
import { interpret, assign } from "xstate";

const fetchData = (countryName: string) => {
  const baseUrl = "https://covid19.mathdro.id/api";
  const countryDataUrl = `${baseUrl}/countries/${countryName}`;
  const myFetch = fetch(countryDataUrl)
    .then(res => res.json())
    .catch(error => {
      throw new Error(error);
    });
  return myFetch;
};

const countryMachine = Machine<any>(
  {
    id: "country",
    context: {
      countryName: "denmark",
      countryData: {},
      error: ""
    },
    initial: "idle",
    states: {
      idle: {
        on: {
          FETCH: {
            target: "loading"
          }
        }
      },
      loading: {
        invoke: {
          src: context => fetchData(context.countryName),
          onDone: {
            target: "success",
            actions: ["setResults"]
          },
          onError: {
            target: "failure",
            actions: ["setError"]
          }
        }
      },
      success: {
        type: "final"
      },
      failure: {
        on: {
          FETCH: "loading"
        }
      }
    }
  },
  {
    actions: {
      setResults: assign({
        countryData: (context, event) => {
          console.log("RESOLVED");
          const countryData = {
            name: context.countryName,
            confirmed: event.data.confirmed.value,
            recovered: event.data.recovered.value,
            deaths: event.data.deaths.value,
            date: new Date(event.data.lastUpdate)
          };
          return countryData;
        }
      }),
      setError: assign({
        error: (_, event) => {
          console.log("FAILED!!!");
          return event.data.message;
        }
      })
    }
  }
);

export default Vue.extend({
  name: "CountryData",
  props: {
    country: String
  },
  data() {
    return {
      fetchingService: interpret(countryMachine),
      current: countryMachine.initialState,
      context: countryMachine.context
    };
  },
  methods: {
    send(event: any) {
      this.fetchingService.send(event);
      console.log("state", this.current.value);
    }
  },
  created() {
    // Start service on component creation
    this.fetchingService
      .onTransition(state => {
        this.current = state;
        state.context.countryName = this.country;
        console.log("state context", state.context);

        this.context = state.context;
      })
      .start();
  }
  // mounted() {
  //   const baseUrl = "https://covid19.mathdro.id/api";
  //   const countryDataUrl = `${baseUrl}/countries/${this.country}`;
  //   setTimeout(() => {
  //     fetch(countryDataUrl).then(res =>
  //       res.json().then(data => {
  //         const countryData = {
  //           name: this.country,
  //           confirmed: data.confirmed.value,
  //           recovered: data.recovered.value,
  //           deaths: data.deaths.value,
  //           date: new Date(data.lastUpdate)
  //         };
  //         this.countryData = countryData;
  //       })
  //     );
  //   }, 2000);
  // }
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
