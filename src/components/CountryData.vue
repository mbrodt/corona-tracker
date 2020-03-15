<template>
  <div class="mx-auto flex flex-col">
    <p class="uppercase text-gray-500 tracking-wide font-bold">
      COVID-19 Coronavirus epidemi
    </p>
    <h1 class="text-3xl lg:text-4xl xl:text-5xl font-bold text-gray-800">
      Danmark - Nøgletal
    </h1>

    <div class="mt-12" v-if="current.matches('success')">
      <div>
        <p>Senest opdateret:</p>
        <p class="font-bold text-2xl text-gray-800 inline">
          {{ context.countryData.date | formatDate }}
        </p>
      </div>
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-8 mt-6">
        <div
          class="bg-white shadow-xl flex flex-col justify-center items-center h-32"
        >
          <p class="text-xl font-semibold text-gray-700">
            Bekræftede tilfælde
          </p>
          <p class="text-3xl font-bold text-orange-600">
            {{ context.countryData.confirmed }}
          </p>
        </div>
        <div
          class="bg-white shadow-xl flex flex-col justify-center items-center h-32"
        >
          <p class="text-xl font-semibold text-gray-700">Helbredt</p>
          <p class="text-3xl font-bold text-green-700">
            {{ context.countryData.recovered }}
          </p>
        </div>
        <div
          class="bg-white shadow-xl flex flex-col justify-center items-center h-32"
        >
          <p class="text-xl font-semibold text-gray-700">Døde</p>
          <p class="text-3xl font-bold text-red-600">
            {{ context.countryData.deaths }}
          </p>
        </div>
      </div>
    </div>
    <div v-else-if="current.matches('failure')">
      Error!
      <p>{{ context.error }}</p>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { TrinityRingsSpinner } from "epic-spinners";
import { Machine } from "xstate";
import { interpret, assign } from "xstate";

const fetchData = async (countryName: string) => {
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
      countryName: "Denmark"
    },
    initial: "loading",
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
          return event.data.message;
        }
      })
    }
  }
);

export default Vue.extend({
  name: "CountryData",
  // components: {
  //   TrinityRingsSpinner
  // },
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
    }
  },
  filters: {
    formatDate(date) {
      if (!date) {
        return "";
      }
      const dateOptions = {
        year: "numeric",
        month: "long",
        day: "numeric"
      };

      const timeOptions = {
        hour: "2-digit",
        minute: "2-digit"
      };

      const day = date.toLocaleDateString("da-DK", dateOptions);
      const time = date.toLocaleTimeString("da-DK", timeOptions);
      return `${day} kl. ${time}`;
    }
  },
  created() {
    // Start service on component creation
    this.fetchingService
      .onTransition(state => {
        this.current = state;
        this.context = state.context;
      })
      .start();
  }
});
</script>
