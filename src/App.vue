

<script setup>
import { ref, computed } from 'vue';

const currentStep = ref(1);
const agreeToTerms = ref(false);
const submissionStatus = ref('');
const submissionMessage = ref('');
const workshopError = ref('');

const workshops = ref([
  { name: 'Kiddidrama 02/04/24', selected: false, numChildren: '', ageChildren: '', foodAllergies: '', dietaryRequirements: '' },
  { name: 'Kiddidrama 09/04/24', selected: false, numChildren: '', ageChildren: '', foodAllergies: '', dietaryRequirements: '' },
  { name: 'Blue Elephant Theatre 05/04/24', selected: false, numChildren: '', ageChildren: '', foodAllergies: '', dietaryRequirements: '' },
  { name: 'Blue Elephant Theatre 11/04/24', selected: false, numChildren: '', ageChildren: '', foodAllergies: '', dietaryRequirements: '' },
  { name: 'Improvised Comedy 10/04/24', selected: false, numChildren: '', ageChildren: '', foodAllergies: '', dietaryRequirements: '' },
]);

const parentInfo = ref({
  name: '',
  address: '',
  phone: '',
  email: '',
});

const errors = ref({});

const selectedWorkshops = computed(() => {
  return workshops.value.filter(workshop => workshop.selected);
});

const validateStep = () => {
  errors.value = {};
  workshopError.value = '';

  if (currentStep.value === 1) {
    if (selectedWorkshops.value.length === 0) {
      workshopError.value = 'Please select at least one workshop';
      return false;
    }
    for (const workshop of selectedWorkshops.value) {
      if (!workshop.numChildren || !workshop.ageChildren) {
        workshopError.value = 'Please fill in all required fields for selected workshops';
        return false;
      }
    }
  } else if (currentStep.value === 2) {
    if (!parentInfo.value.name) errors.value.name = 'Name is required';
    if (!parentInfo.value.address) errors.value.address = 'Address is required';
    if (!parentInfo.value.phone) errors.value.phone = 'Phone number is required';
    if (Object.keys(errors.value).length > 0) return false;
  }

  return true;
};

const nextStep = () => {
  if (validateStep()) {
    currentStep.value++;
  }
};

const previousStep = () => {
  currentStep.value--;
};

const submitForm = async () => {
  if (!validateStep()) return;

  const formData = new FormData();

  selectedWorkshops.value.forEach((workshop, index) => {
    formData.append(`workshop${index + 1}YN`, workshop.name);
    formData.append(`num_children${index + 1}`, workshop.numChildren);
    formData.append(`age_children${index + 1}`, workshop.ageChildren);
    formData.append(`food_allergies${index + 1}`, workshop.foodAllergies);
    formData.append(`dietary_requirements${index + 1}`, workshop.dietaryRequirements);
  });

  formData.append('parent_name', parentInfo.value.name);
  formData.append('address', parentInfo.value.address);
  formData.append('telephone', parentInfo.value.phone);
  formData.append('email', parentInfo.value.email || 'noemail@noemail.com');

  try {
    const response = await fetch('https://api.n1g3.com/api/drama-form', {
      method: 'POST',
      body: formData,
    });

    const data = await response.json();

    if (response.ok) {
      submissionStatus.value = 'success';
      submissionMessage.value = 'Form submitted successfully!';
      // Reset form
      workshops.value.forEach(workshop => {
        workshop.selected = false;
        workshop.numChildren = '';
        workshop.ageChildren = '';
        workshop.foodAllergies = '';
        workshop.dietaryRequirements = '';
      });
      parentInfo.value = { name: '', address: '', phone: '', email: '' };
      currentStep.value = 1;
      agreeToTerms.value = false;
    } else {
      submissionStatus.value = 'error';
      submissionMessage.value = data.message || 'An error occurred while submitting the form.';
    }
  } catch (error) {
    console.error('Error:', error);
    submissionStatus.value = 'error';
    submissionMessage.value = 'An error occurred while submitting the form.';
  }
};
</script>
<template>
  <div class="container mx-auto p-4">
    <h2 class="text-2xl font-bold text-center mb-4">Youth Drama Club Registration - Easter 2024</h2>

    <div v-if="currentStep === 1">
      <h3 class="text-xl font-semibold mb-2">Step 1: Select Workshops</h3>
      <div v-for="(workshop, index) in workshops" :key="index" class="mb-2">
        <label class="flex items-center space-x-2">
          <input type="checkbox" v-model="workshop.selected" class="checkbox checkbox-primary" />
          <span>{{ workshop.name }}</span>
        </label>
        <div v-if="workshop.selected" class="ml-6 mt-2 space-y-2">
          <input v-model="workshop.numChildren" type="number" placeholder="Number of Children" class="input input-bordered w-full max-w-xs" />
          <input v-model="workshop.ageChildren" type="text" placeholder="Age of Children" class="input input-bordered w-full max-w-xs" />
          <input v-model="workshop.foodAllergies" type="text" placeholder="Food Allergies (if any)" class="input input-bordered w-full max-w-xs" />
          <input v-model="workshop.dietaryRequirements" type="text" placeholder="Dietary Requirements" class="input input-bordered w-full max-w-xs" />
        </div>
      </div>
      <div v-if="workshopError" class="text-error mt-2">{{ workshopError }}</div>
    </div>

    <div v-if="currentStep === 2">
      <h3 class="text-xl font-semibold mb-2">Step 2: Parent Information</h3>
      <div class="space-y-4">
        <div>
          <label class="label">
            <span class="label-text">Name of Parent / Guardian:</span>
          </label>
          <input v-model="parentInfo.name" type="text" placeholder="Full Name" class="input input-bordered w-full" />
          <div v-if="errors.name" class="text-error mt-1">{{ errors.name }}</div>
        </div>
        <div>
          <label class="label">
            <span class="label-text">Address:</span>
          </label>
          <textarea v-model="parentInfo.address" placeholder="Address" class="textarea textarea-bordered w-full"></textarea>
          <div v-if="errors.address" class="text-error mt-1">{{ errors.address }}</div>
        </div>
        <div>
          <label class="label">
            <span class="label-text">Phone:</span>
          </label>
          <input v-model="parentInfo.phone" type="tel" placeholder="Phone Number" class="input input-bordered w-full" />
          <div v-if="errors.phone" class="text-error mt-1">{{ errors.phone }}</div>
        </div>
        <div>
          <label class="label">
            <span class="label-text">Email:</span>
          </label>
          <input v-model="parentInfo.email" type="email" placeholder="Email Address" class="input input-bordered w-full" />
          <div v-if="errors.email" class="text-error mt-1">{{ errors.email }}</div>
        </div>
      </div>
    </div>

    <div v-if="currentStep === 3">
      <h3 class="text-xl font-semibold mb-2">Step 3: Review and Submit</h3>
      <div class="space-y-4">
        <div>
          <h4 class="font-semibold">Selected Workshops:</h4>
          <ul class="list-disc list-inside">
            <li v-for="workshop in selectedWorkshops" :key="workshop.name">
              {{ workshop.name }} - {{ workshop.numChildren }} child(ren), Age(s): {{ workshop.ageChildren }}
            </li>
          </ul>
        </div>
        <div>
          <h4 class="font-semibold">Parent Information:</h4>
          <p>Name: {{ parentInfo.name }}</p>
          <p>Address: {{ parentInfo.address }}</p>
          <p>Phone: {{ parentInfo.phone }}</p>
          <p>Email: {{ parentInfo.email }}</p>
        </div>
        <div class="form-control">
          <label class="label cursor-pointer">
            <span class="label-text">I agree to receive email communication and understand that my contact information will be stored in relation to this project.</span>
            <input v-model="agreeToTerms" type="checkbox" class="checkbox checkbox-primary" />
          </label>
        </div>
      </div>
    </div>

    <div class="mt-4 space-x-2">
      <button v-if="currentStep > 1" @click="previousStep" class="btn btn-secondary">Previous</button>
      <button v-if="currentStep < 3" @click="nextStep" class="btn btn-primary">Next</button>
      <button v-if="currentStep === 3" @click="submitForm" class="btn btn-primary" :disabled="!agreeToTerms">Submit</button>
    </div>

    <div v-if="submissionStatus" :class="['alert', submissionStatus === 'success' ? 'alert-success' : 'alert-error', 'mt-4']">
      <span>{{ submissionMessage }}</span>
    </div>
  </div>
</template>
