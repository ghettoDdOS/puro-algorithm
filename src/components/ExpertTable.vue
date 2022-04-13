<template>
  <div class="container">
    <div class="table-wrapper">
      <div class="table">
        <form
          @submit.prevent="
            experts.push(newExpert), (newExpert = { name: '', values: [] })
          "
        >
          <table>
            <thead>
              <tr>
                <th>
                  <input
                    v-model="newExpert.name"
                    placeholder="Эксперт"
                    type="text"
                    name="expert"
                    required
                  />
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="value in newExpert.values">
                <td>
                  {{ value }}
                  <button
                    @click="
                      newExpert.values.splice(
                        newExpert.values.indexOf(value),
                        1
                      )
                    "
                    type="button"
                  >
                    Удалить
                  </button>
                </td>
              </tr>
              <tr v-if="selectableAlphabet.length > 0">
                <td>
                  <button
                    @click="newExpert.values.push(letter)"
                    name="new-value"
                    v-for="letter in selectableAlphabet"
                    type="button"
                  >
                    {{ letter }}
                  </button>
                </td>
              </tr>
              <tr v-else-if="newExpert.name !== ''">
                <td>
                  <input value="Добавить" type="submit" />
                </td>
              </tr>
              <tr>
                <td>
                  <button @click="experts = []" type="reset">Сброс</button>
                </td>
              </tr>
            </tbody>
          </table>
        </form>
      </div>
    </div>
    <div class="table-wrapper">
      <div class="table" v-if="experts.length > 0">
        <table>
          <thead>
            <tr>
              <th>объекты</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="letter in alphabet">
              <td>{{ letter }}</td>
            </tr>
          </tbody>
        </table>
        <table v-for="expert in experts">
          <thead>
            <tr>
              <th>
                {{ expert.name }}
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="value in expert.values">
              <td>{{ value }}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="table" v-if="rangeExperts.length > 0">
        <table>
          <thead>
            <tr>
              <th>объекты</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(value, i) in alphabet">
              <td>{{ prefix + String(i + 1) + value }}</td>
            </tr>
          </tbody>
        </table>
        <table v-for="expert in rangeExperts">
          <thead>
            <tr>
              <th>
                {{ expert.name }}
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="variant in expert.variants">
              <td>{{ variant }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div class="table-wrapper" v-if="sortedExperts.length > 0">
      <div class="table" v-for="expert in sortedExperts">
        <table>
          <thead>
            <tr>
              <th>{{ expert.name }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(value, i) in alphabet">
              <td>{{ prefix + String(i + 1) + value }}</td>
            </tr>
          </tbody>
        </table>
        <table v-for="(variantValue, i) in expert.variantValues">
          <thead>
            <tr>
              <th>{{ prefix + String(i + 1) }}</th>
            </tr>
          </thead>
          <tbody v-for="value in variantValue">
            <tr>
              <td>
                {{ value }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="table" v-if="resultExperts.length > 0">
        <table>
          <thead>
            <tr>
              <th>D<sub>ij</sub></th>
            </tr>
          </thead>
          <tbody v-for="expert in experts">
            <tr>
              <td>
                {{ expert.name }}
              </td>
            </tr>
          </tbody>
        </table>
        <table v-for="expert in experts">
          <thead>
            <tr>
              <th>{{ expert.name }}</th>
            </tr>
          </thead>
          <tbody
            v-for="value in resultExperts.filter((e) => e.name === expert.name)"
          >
            <tr>
              <td>
                {{ value.count }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from "vue";

const prefix = "V";
const alphabet = ["a", "b", "c", "d", "h"];
const experts = ref([
  { name: "Э1", values: ["b", "a", "c", "d", "h"] },
  { name: "Э2", values: ["b", "a", "c", "h", "d"] },
  { name: "Э3", values: ["d", "h", "b", "c", "a"] },
]);
const newExpert = ref({ name: "", values: [] });
const tmpValue = ref("");
const selectableAlphabet = computed(() =>
  alphabet.filter((letter) => newExpert.value.values.indexOf(letter) === -1)
);
const rangeExperts = computed(() =>
  experts.value.map((expert) => ({
    ...expert,
    variants: expert.values.map(
      (value) => prefix + String(alphabet.indexOf(value) + 1)
    ),
  }))
);
const sortedExperts = computed(() =>
  rangeExperts.value.map((expert) => ({
    ...expert,
    variantValues: expert.values.map((v) =>
      [...expert.variants]
        .sort()
        .map((variant) =>
          [...expert.variants].sort().indexOf(variant) ===
          expert.values.indexOf(v)
            ? 0
            : expert.variants.indexOf(variant) <
              expert.variants.indexOf(
                prefix + String(expert.values.indexOf(v) + 1)
              )
            ? 1
            : -1
        )
    ),
  }))
);
const resultExperts = computed(() => {
  const result = [];

  for (let i = 0; i < sortedExperts.value.length; i++) {
    for (let j = 0; j < sortedExperts.value.length; j++) {
      let counter = 0;
      for (let k = 0; k < alphabet.length; k++) {
        for (let z = 0; z < alphabet.length; z++) {
          counter += Number(
            sortedExperts.value[i].variantValues[k][z] !==
              sortedExperts.value[j].variantValues[k][z]
          );
        }
      }
      result.push({ name: sortedExperts.value[i].name, count: counter });
    }
  }

  return result;
});
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
}
html {
  /* background: linear-gradient(-45deg, #ee7752, #e73c7e, #23a6d5, #23d5ab); */
  /* background-size: 400% 400%; */
  /* animation: gradient 8s ease infinite; */
}
.table-wrapper {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
}
.table {
  display: flex;
  flex: 1 1 50%;
  justify-content: center;
  padding: 10px;
}
table {
  border: 2px solid #13a17b;
  border-radius: 3px;
  background-color: rgba(87, 73, 73, 0.151);
  /* backdrop-filter: blur(12px); */
}

th {
  background-color: #13a17b;
  color: #fff;
}

td {
  color: #000000;
  font-weight: 700;
  background-color: #f9f9f956;

  text-align: center;
}

th,
td {
  min-width: 80px;
  padding: 10px 20px;
}
@keyframes gradient {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
</style>
