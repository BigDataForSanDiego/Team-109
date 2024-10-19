<script>
  import Gemini from './Gemini.svelte';
  import NavBar from './NavBar.svelte';
  import { onMount } from 'svelte';
  import { run, transformTextToHtml } from './gemini-server.js';

  let data = {};
  let error = null;
  let loading = true;
  let isEditing = true;
  let selectedDate = new Date();
  let currentDate = new Date().toLocaleDateString();
  let isDisabled = false;
  let symptoms = "";
  let foods = "";
  let readings = "";

  // Default data structure for user inputs
  let journalData = {
    bloodGlucose: [],
    nutrition: [],
    symptoms: {
      shaking: false,
      sweating: false,
      anxiety: false,
      irritability: false,
      dizziness: false,
      hunger: false
    },
    additionalNotes: ""
  };

  // Function to handle saving all data
  function saveData() {
    isEditing = false; // Disable all input fields, text boxes, and symptom toggles
    isDisabled = true;
    loadData();
    
    setTimeout(() => {
        isDisabled = false;
        isEditing = true;
    }, 10_000)
  }
  // Function to toggle edit mode
  function toggleEdit() {
    isEditing = !isEditing;
  }
  // Function to handle adding blood glucose record
  function addBloodGlucoseRecord() {
    let newRecord = {
      time: new Date().toLocaleTimeString(), 
      level: ''
    };
    journalData.bloodGlucose.push(newRecord);
    journalData.bloodGlucose = journalData.bloodGlucose
  }
  // Function to delete blood glucose record
    function deleteBloodGlucoseRecord(index) {
    journalData.bloodGlucose.splice(index, 1);
    journalData.bloodGlucose = journalData.bloodGlucose;
  }
  // Function to add nutrition record
  function addNutritionRecord() {
    journalData.nutrition.push('');
    journalData.nutrition = journalData.nutrition;
  }
  // Function to delete nutrition record
  function deleteNutritionRecord(index) {
    journalData.nutrition.splice(index, 1);
    journalData.nutrition = journalData.nutrition;
  }
  // Function to toggle symptom status
  function toggleSymptom(symptom) {
    journalData.symptoms[symptom] = !journalData.symptoms[symptom];
  }
  // Change the journal data based on the selected calendar day
  function changeJournalForDay(date) {
    selectedDate = date;
    // Logic to load specific day's journalData
  }
  
  async function loadData() {
    try {
        symptoms = Object.keys(journalData.symptoms).filter(key => journalData.symptoms[key]).join(', ');
        foods = journalData.nutrition.join(', ');
        readings = journalData.bloodGlucose.map(item => `${item.level} mg/dl`).join(', ');
        loading = true;
        data = {};

        // Blood Glucose
        data["Blood Glucose"] = transformTextToHtml(await run(`Is having blood glucose levels of ${readings} within normal range for an individual of 5'2, 120lbs, and 21`));
        // Diet
        if (typeof foods === 'string' && foods.length > 0){
          data["Diet Tracker"] = transformTextToHtml(await run(`Heres a list of food I ate in 1 serving: ${foods}. If more info is needed on type of food, then explicitly say that.`));
        }
        // Symptoms
        if (typeof symptoms === 'string' && symptoms.length > 0){
          data["Symptoms"] = transformTextToHtml(await run(`Here is a list of symptoms i have: ${symptoms}. How do these relate to blood glucose?`));
        }
        // Additional notes
        if (typeof data["Additional notes"] === 'string' && data["Additional notes"].length > 0){
          data["Additional notes"] = transformTextToHtml(await run(journalData.additionalNotes));
        }
      } catch (err) {
        error = "Responses are currently unavailable. <br/> Please try again later :(";
        console.log(err)
    } finally {
        loading = false;
    }
  }
  
  onMount(() => {
      loadData();
  });

</script>

<main>
  <NavBar/>
  <div class="content">
    <div class="main-content-wrapper">
      <div class="main-content">
        <!-- Left Section: Daily Journal -->
        <div class="journal-section">
          <h2>
            Daily Journal 
            <!-- <img src="/path/to/icon.png" alt="Journal Icon" /> -->
          </h2>
    
          <!-- Blood Glucose Subsection -->
          <div class="subsection">
            <h3>Blood Glucose</h3>
            {#each journalData.bloodGlucose as record, index}
              <div class="record">
                <input
                  type="time"
                  bind:value={record.time}
                  disabled={!isEditing}
                />
                <input
                  type="number"
                  min="0"
                  bind:value={record.level}
                  placeholder="Blood glucose level (mg/dL)"
                  disabled={!isEditing}
                />
                {#if isEditing}
                  <button on:click={() => deleteBloodGlucoseRecord(index)}>Delete</button>
                {/if}
              </div>
            {/each}
            <button on:click={addBloodGlucoseRecord} disabled={!isEditing}>+</button>
          </div>
          <!-- Nutrition Tracker Subsection -->
          <div class="subsection">
            <h3>Nutrition Tracker</h3>
            {#each journalData.nutrition as food, index}
              <div class="record">
                <input
                  type="text"
                  bind:value={food}
                  placeholder="Enter food"
                  disabled={!isEditing}
                />
                {#if isEditing}
                  <button on:click={() => deleteNutritionRecord(index)}>Delete</button>
                {/if}
              </div>
            {/each}
            <button on:click={addNutritionRecord} disabled={!isEditing}>+</button>
          </div>
    
          <!-- Symptoms Subsection -->
          <div class="subsection">
            <h3>Symptoms</h3>
            <div class="symptoms-grid">
              {#each Object.keys(journalData.symptoms) as symptom}
                <button
                  class="symptom-icon {journalData.symptoms[symptom] ? 'active' : ''}"
                  on:click={() => toggleSymptom(symptom)}
                  disabled={!isEditing}
                >
                  <span>{symptom}</span>
            </button>
              {/each}
            </div>
          </div>
    
          <!-- Additional Notes Subsection -->
          <div class="subsection">
            <h3>Additional Notes</h3>
            <textarea
              bind:value={journalData.additionalNotes}
              placeholder="Type any additional notes here"
              disabled={!isEditing}
              id="prompt" 
              name="prompt" 
              rows="8" 
            ></textarea>
          </div>
          
          <!-- Save/Edit Button -->
          <button on:click={isEditing ? saveData : toggleEdit}>
            {isEditing ? 'Save' : 'Edit'}
          </button>
        </div>
    
        <!-- Right Section -->
        <div class="right-content">
          <!-- Right Section: Calendar -->
          <div class="calendar-section">
            <h2>
              Calendar 
              <!-- <img src="/path/to/icon.png" alt="Calendar Icon" /> -->
            </h2>
            <!-- Placeholder for calendar -->
            <div class="calendar">
              <!-- Calendar logic to show days and allow selecting a day -->
              {#each Array(30) as _, index}
                <button
                  class="day {new Date().getDate() === index + 1 ? 'current-day' : ''}"
                  on:click={() => changeJournalForDay(new Date().setDate(index + 1))}
                >
                  {index + 1}
            </button>
              {/each}
            </div>
          </div>

          <!-- Right Section: Gemini -->
          <div class="gemini-section">
            <Gemini {data} {error} {loading}/>
          </div>
        </div>
      </div>  
    </div>
  </div>

</main>

<style>
  .content{
    margin-left: 5%;
    margin-right: 5%;
    background-color: white;
  }

  .main-content-wrapper{
    padding: 20px;
    padding-top: 85px;
    margin-left: 2%;
    margin-right: 2%;
  }

  @media (min-width: 925px) {
    .main-content {
      display: grid;
      grid-template-columns: 2fr 1fr;;
      gap: 10px;
      padding: 20px;
      border-radius: 10px;
    }
  }

  /* Left section: Daily Journal */
  .journal-section {
    width: auto;
    padding: 20px;
    background-color: #EBF4F6;
    margin-bottom: 10px;
    border-radius: 10px;
  }
  .subsection {
    margin-bottom: 20px;
  }
  .subsection h3 {
    margin-bottom: 10px;
  }

  /* Right section: Calendar */
  .calendar-section {
    width: auto;
    height: auto;
    padding: 20px;
    background-color: #EBF4F6;
    margin-bottom: 10px;
    border-radius: 10px;
  }

  /* Calendar styles */
  .calendar {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 10px;
    margin-top: 20px;
    width: 100%;
  }
  .day {
    padding: 10px;
    text-align: center;
    border: 1px solid #ccc;
    cursor: pointer;
  }

  @media (max-width: 600px) {
    .calendar {
      grid-template-columns: repeat(4, 1fr); /* 4 columns on small screens */
    }
  }

  /* For very small screens, below 400px */
  @media (max-width: 400px) {
    .calendar {
      grid-template-columns: repeat(2, 1fr); /* 2 columns on very small screens */
    }
  }

  .current-day {
    background-color: #007bff;
    color: white;
  }

  /* Right section: Gemini */
  .gemini-section {
    padding: 20px;
    background-color: #EBF4F6;
    height: 300px;
    margin-bottom: 10px;
    border-radius: 10px;
  }

  
  /* Symptom icons */
  .symptoms-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
  }

  @media (max-width: 600px) {
    .symptoms-grid {
      grid-template-columns: repeat(2, 1fr); /* 4 columns on small screens */
    }
  }

  .symptom-icon {
    padding: 10px;
    border: 1px solid #007bff;
    border-radius: 5px;
    text-align: center;
    cursor: pointer;
    /* max-width: 100px; */
  }
  .symptom-icon.active {
    background-color: #007bff;
    color: white;
  }

  /* Additional Notes */
  #prompt{
    /* min-width: 300px; */
    height: 10%;
    width: 98%;
    resize: none;
  }

  h2{
    margin: 0;
    padding: 0;
  }

  :global(body) {
    margin: 0; 
    padding: 0; 
    box-sizing: border-box;
    background-color: #EBF4F6;
  }
</style>
