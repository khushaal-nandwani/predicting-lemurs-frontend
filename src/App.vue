<template>
  <div class="app-container">
    <div class="hello">
      <h1>Lemur Lifespan Prediction</h1> 
      <img src="/lemur.svg" style="height: 6rem;" alt="Lemur Image" class="lemur-svg" />
    </div>
    
    <p>Please fill out the animal details and submit to predict captive and wild lifespan. </p>
    <p> This is the frontend to the models created in <a href="https://github.com/khushaal-nandwani/predicting-lemurs" target="_blank">this paper</a>.</p>
    <!-- Input form -->
    <form @submit.prevent="submitForm">
      <div class="form-row" v-for="(row, index) in formRows" :key="index">
        <!-- animal id -->
        <div class="form-group">
          <label>Animal ID:</label>
          <input type="text" v-model="row.animal_id" placeholder="e.g., 0011" />
        </div>

        <div class="form-group">
          <label>Sex:</label>
          <select v-model="row.sex">
            <option disabled value="">--Select--</option>
            <option>M</option>
            <option>F</option>
          </select>
        </div>
        <div class="form-group">
          <label>Genus:</label>
          <select v-model="row.genus">
            <option disabled value="">--Select Genus--</option>
            <option v-for="genus in genusList" :key="genus" :value="genus">
              {{ genus }}
            </option>
          </select>
        </div>

        <div class="form-group">
          <label>Species:</label>
          <select v-model="row.species">
            <option disabled value="">--Select Species--</option>
            <option v-for="species in speciesList" :key="species" :value="species">
              {{ species }}
            </option>
          </select>
        </div>

        <!-- <div class="form-group">
          <label>Age (Years):</label>
          <input type="number" step="0.01" v-model.number="row.age" placeholder="e.g., 3.93" />
        </div> -->
        <div class="form-group">
          <label>Month Born:</label>
          <select v-model="row.month_born">
            <option disabled value="">--Select Month--</option>
            <option v-for="month in monthList" :key="month" :value="month">
              {{ month }}
            </option>
          </select>
        </div>

        <button class="remove-btn" type="button" @click="removeRow(index)">Remove</button>
      </div>

      <button class="add-btn" type="button" @click="addRow">Add Another Animal</button>
      <button class="submit-btn" type="submit">Submit</button>



    </form>

    <!-- Display predictions -->
    <div v-if="predictions" class="predictions-container">
      <h2>Predicted Lifespans</h2>
      <table>
        <thead>
          <tr class="labels-color">
            <th>Animal ID</th>
            <th>Predicted Captive Lifespan</th>
            <th>Predicted Wild Lifespan</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(id, idx) in formRows.map(row => row.animal_id)" :key="id">
            <td>{{ id }}</td>
            <td>{{ predictions.predictions['predicted captive lifespan/s'][idx].toFixed(2) }} years</td>
            <td>{{ predictions.predictions['predicted wild lifespan/s'][idx].toFixed(2) }} years</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const speciesList = [
  "GG", "COL", "MOH", "MON", "VV", "RUB", "PYG", "COU", "ZAZ", "RUF", "FUL",
  "ALB", "CAT", "MAC", "SAN", "COQ", "MED", "COR", "FLA", "MAD", "MUR",
  "POT", "TAR"
];

const genusList = ["O", "E", "H", "G", "V", "N", "M", "L", "P", "C", "D"];

//  [1] "Jan" "Oct" "Dec" "Aug" "Jul" "Sep" "Jun" "Mar" "Apr" "Nov" "May"

const monthList = ["Jan", "Oct", "Dec", "Aug", "Jul", "Sep", "Jun", "Mar", "Apr", "Nov", "May"]; ``


const formRows = ref([
  {
    sex: "",
    birth_type: "",
    genus: "",
    species: "",
    age: null,
    month_born: ""
  }
])

const predictions = ref(null)

const addRow = () => {
  formRows.value.push({
    animal_id: "",
    sex: "",
    birth_type: "",
    genus: "",
    species: "",
    age: null,
    month_born: ""
  })
}

const removeRow = (index) => {
  formRows.value.splice(index, 1)
}

const submitForm = async () => {
  // Construct the payload in the expected format
  const payload = {
    data: {
      animal_id: formRows.value.map(r => r.animal_id),
      sex: formRows.value.map(r => r.sex),
      birth_type: formRows.value.map(r => "wild"),
      genus: formRows.value.map(r => r.genus),
      species: formRows.value.map(r => r.species),
      age: formRows.value.map(r => 0),
      month_born: formRows.value.map(r => r.month_born)
    }
  }

  // Replace with your actual backend endpoint
  const url = "http://3.99.183.165:8000/predict"

  try {
    const response = await fetch(url, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload)
    })
    if (!response.ok) {
      throw new Error("Network response was not ok")
    }
    const data = await response.json()
    predictions.value = data
  } catch (error) {
    console.error("Error:", error)
    alert("An error occurred while fetching predictions.")
  }
}
</script>

<style scoped>

.hello {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}

.app-container {
  /* min width full  */
  margin: 2rem auto;
  font-family: Arial, sans-serif;
}

h1 {
  text-align: center;
  margin-bottom: 1rem;
}

p {
  text-align: center;
  margin-bottom: 2rem;
  color: #555;
}

form {
  background: #f9f9f908;
  padding: 1rem;
  border-radius: 8px;
}

.form-row {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-end;
  gap: 1rem;
  margin-bottom: 1rem;
  border-bottom: 1px solid #ddd;
  padding-bottom: 1rem;
}

.labels-color {
  /* text color is light gray */
  color: #f9f9f9;
}

.form-group {
  display: flex;
  flex-direction: column;
  width: 150px;
}

.form-group label {
  font-weight: bold;
  margin-bottom: 0.5rem;
}

input[type="text"],
input[type="number"],
select {
  border: 1px solid #ccc;
  padding: 0.5rem;
  border-radius: 4px;
}

.add-btn,
.submit-btn,
.remove-btn {
  background: #0a3768;
  color: #fff;
  border: none;
  padding: 0.7rem 1rem;
  margin: 0.5rem;
  border-radius: 4px;
  cursor: pointer;
}

.remove-btn {
  background: #4e0a0a;
  margin-left: auto;
}

.add-btn {
  margin: 1rem 0;
}

.add-btn:hover,
.submit-btn:hover,
.remove-btn:hover {
  opacity: 0.9;
}

.predictions-container {
  margin-top: 2rem;
  background: #f9f9f908;
  padding: 1rem;
  border-radius: 8px;
}

.predictions-container h2 {
  margin-bottom: 1rem;
}

table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
}

thead {
  background: #0a3768;
}

th,
td {
  padding: 0.75rem;
  border-bottom: 1px solid #0a3768;
}
</style>
