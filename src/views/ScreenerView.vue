<template>
  <v-container class="d-flex flex-column align-center justify-center" style="min-height: 100vh; max-width: 600px;">
    <v-card elevation="8" class="pa-6" style="width: 100%;">
      <div class="d-flex justify-space-between align-center mb-4">
        <h2>About Your Household</h2>
        <v-btn 
          color="error" 
          variant="outlined" 
          size="small"
          @click="startOver"
        >
          Start Over
        </v-btn>
      </div>
      
      <form @submit.prevent="onNext">
        <!-- S1 -->
        <div class="mb-6">
          <h3 class="mb-4">What is the square footage of your current house or apartment? (Do not include outdoor space)</h3>
          <v-radio-group v-model="s1_response_type" class="mb-4">
            <v-radio value="enter" label="Enter a value"></v-radio>
            <v-radio value="dont_know" label="Don't know"></v-radio>
          </v-radio-group>
          
          <v-text-field
            v-if="s1_response_type === 'enter'"
            v-model="s1"
            label="Square footage"
            type="number"
            min="0"
            :rules="[v => v === '' || v >= 0 || 'Must be a positive number']"
            class="mb-4"
            placeholder="e.g. 1200"
          />
        </div>
        
        <!-- S1a -->
        <div class="mb-6">
          <h3 class="mb-4">Which of the following best describes where you live?</h3>
          <v-radio-group v-model="s1a" class="mb-4">
            <v-radio value="1" label="Apartment in a shared building"></v-radio>
            <v-radio value="2" label="Condominium or townhouse (attached unit you own)"></v-radio>
            <v-radio value="3" label="Single-family detached house"></v-radio>
            <v-radio value="4" label="Duplex, triplex, or row house (shared walls, separate units)"></v-radio>
            <v-radio value="5" label="Mobile or manufactured home"></v-radio>
            <v-radio value="6" label="Tiny home or accessory dwelling unit (ADU)"></v-radio>
            <v-radio value="96" label="Other (please specify)"></v-radio>
            <v-radio value="98" label="Don't Know"></v-radio>
          </v-radio-group>
          
          <v-text-field
            v-if="s1a === '96'"
            v-model="s1a_other"
            label="Please specify:"
            class="mb-4"
          />
        </div>
        
        <!-- S2 -->
        <div class="mb-6">
          <h3 class="mb-4">Including yourself, how many people currently live or stay in your household?</h3>
          <v-radio-group v-model="s2_response_type" class="mb-4">
            <v-radio value="enter" label="Enter a value"></v-radio>
            <v-radio value="dont_know" label="Don't know"></v-radio>
          </v-radio-group>
          
          <v-text-field
            v-if="s2_response_type === 'enter'"
            v-model="s2"
            label="Number of people"
            type="number"
            min="1"
            :rules="[v => v === '' || v > 0 || 'Must be at least 1']"
            class="mb-4"
            placeholder="e.g. 3"
          />
        </div>
        
        <div class="d-flex justify-space-between mt-6">
          <v-btn color="secondary" @click="onBack">Back</v-btn>
          <v-btn color="primary" type="submit">Next</v-btn>
        </div>
      </form>
    </v-card>
  </v-container>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { openDB } from 'idb'

const router = useRouter()

const s1 = ref('')
const s1_response_type = ref('enter')
const s1_special = ref(null)
const s1a = ref(null)
const s1a_other = ref('')
const s2 = ref('')
const s2_response_type = ref('enter')
const s2_special = ref(null)

const s1SpecialOptions = [
  { text: 'Web Blank', value: 97 },
  { text: 'Don\'t Know', value: 98 },
]
const s1aOptions = [
  { text: 'Apartment in a shared building', value: 1 },
  { text: 'Condominium or townhouse (attached unit you own)', value: 2 },
  { text: 'Single-family detached house', value: 3 },
  { text: 'Duplex, triplex, or row house (shared walls, separate units)', value: 4 },
  { text: 'Mobile or manufactured home', value: 5 },
  { text: 'Tiny home or accessory dwelling unit (ADU)', value: 6 },
  { text: 'Other (please specify)', value: 96 },
  { text: 'Web Blank', value: 97 },
  { text: 'Don\'t Know', value: 98 },
]
const s2SpecialOptions = [
  { text: 'Web Blank', value: 97 },
]

async function startOver() {
  try {
    // Clear user database
    const userDB = await openDB('user-db', 1)
    await userDB.clear('user-info')
    
    // Clear Suspra database if it exists
    try {
      const suspraDB = await openDB('SuspraDB', 1)
      const storeNames = ['food', 'water', 'energy', 'goods', 'habitat', 'movement', 'community', 'screener']
      
      for (const storeName of storeNames) {
        if (suspraDB.objectStoreNames.contains(storeName)) {
          await suspraDB.clear(storeName)
        }
      }
    } catch (error) {
      // SuspraDB might not exist yet, which is fine
      console.log('SuspraDB not found or already cleared')
    }
    
    // Navigate back to intro screen
    router.push({ name: 'home' })
  } catch (error) {
    console.error('Error starting over:', error)
    // Still navigate back even if clearing fails
    router.push({ name: 'home' })
  }
}

function onNext() {
  // Placeholder: Save data and go to next section
  router.push({ name: 'food' })
}
function onBack() {
  router.push({ name: 'home' })
}
</script> 