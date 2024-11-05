<!--REFERENCES:- CHATGPT - How to convert code from options to composition API? Can you help me convert the code to Composition API.-->

<template>
    <div>
      <form @submit.prevent="fetchRemoteData">
        <div class="form-group">
          <h2>Weather Data</h2>
          <p>Please fill in the form and use the submit button to load data.</p>
          <label for="">Location</label>
          <br>
          <!-- The input field for location is bound to formInput.location -->
          <input class="form-control rounded" v-model="formInput.location" type="text" required />
        </div>
  
        <div class="form-group">
          <label for="">Start Date</label>
          <br>
          <!-- The input field for start date is bound to formInput.startDt -->
          <input class="form-control rounded" type="date" v-model="formInput.startDt" required />
        </div>
  
        <div class="form-group">
          <label for="">End Date</label>
          <br>
          <!-- The input field for end date is bound to formInput.endDt -->
          <input class="form-control rounded" type="date" v-model="formInput.endDt" required />
        </div>
        <br>
        <div>
          <button class="btn btn-danger" type="submit">Submit</button>
        </div>
      </form>
  
      <br /><br />
  
      <!-- Display chart only when dataLoaded is true -->
      <div v-if="dataLoaded">
        <Bar :data="chartData" :options="chartOptions" />
      </div>
  
      <!-- Display weather data in a table format when dataLoaded is true -->
      <div v-if="dataLoaded">
        <hr>
        <h5>Weather Data in a Table</h5>
        <table class="table">
          <thead>
            <tr>
              <th v-for="adate in keys" :key="adate">{{ adate }}</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td v-for="atemp in values" :key="atemp">{{ atemp }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  import axios from 'axios';
  
  // Import required Chart.js modules and components
  import {
    Chart as ChartJS,
    Title,
    Tooltip,
    Legend,
    BarElement,
    CategoryScale,
    LinearScale
  } from 'chart.js';
  import { Bar } from 'vue-chartjs';
  
  // Register Chart.js modules
  ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend);
  
  // Reactive state for the form inputs, chart data, and loaded data status
  const formInput = ref({
    startDt: '', // Start date for weather data
    endDt: '',   // End date for weather data
    location: '' // Location for weather data
  });
  
  // Chart configuration options
  const chartOptions = {
    responsive: true,
    maintainAspectRatio: true,
    scales: {
      x: {
        title: {
          display: true,
          text: 'Time' // Label for x-axis
        }
      },
      y: {
        title: {
          display: true,
          text: 'Temperature (F)' // Label for y-axis
        }
      }
    }
  };
  
  // Reactive state for chart data, data loading status, and table data
  const chartData = ref({
    labels: [], // Dates for x-axis
    datasets: [
      {
        label: 'Average Temperature', // Label for the dataset
        backgroundColor: '#f87979',    // Color of the bar
        data: []                       // Temperature data for y-axis
      }
    ]
  });
  
  const dataLoaded = ref(false); // Tracks if data is loaded to conditionally render elements
  const keys = ref([]);          // Stores dates for table headers
  const values = ref([]);        // Stores temperatures for table cells
  
  // Function to fetch weather data using axios
  async function fetchRemoteData() {
    const options = {
      method: 'GET',
      url: 'https://weatherapi-com.p.rapidapi.com/history.json',
      params: {
        q: formInput.value.location,
        dt: formInput.value.startDt,
        end_dt: formInput.value.endDt,
        lang: 'en'
      },
      headers: {
        'X-RapidAPI-Key': 'YOUR_API_KEY', // Replace with your actual API key
        'X-RapidAPI-Host': 'weatherapi-com.p.rapidapi.com'
      }
    };
  
    try {
      // Send API request and process the response
      const resp = await axios.request(options);
      const wData = resp.data.forecast.forecastday; // Extract forecast data from API response
      
      // Map dates and average temperatures for table and chart display
      keys.value = wData.map(item => item.date);
      values.value = wData.map(item => item.day.avgtemp_f);
  
      // Update chart data with new labels and data
      chartData.value = {
        labels: keys.value,
        datasets: [
          {
            label: 'Average Temperature',
            backgroundColor: '#f87979',
            data: values.value
          }
        ]
      };
  
      dataLoaded.value = true; // Set dataLoaded to true to display chart and table
    } catch (error) {
      console.error("Error fetching weather data:", error);
    }
  }
  </script>
  
  <style>
  /* Styling for table and table elements */
  table, th, td {
    border: 1px solid black;
    border-collapse: collapse;
  }
  </style>
  
