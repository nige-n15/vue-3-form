

<script setup>
import { ref, computed } from 'vue';
import backgroundImageUrl from '/flowersbg.webp';

const currentStep = ref(1);
const agreeToTerms = ref(false);
const submissionStatus = ref('');
const submissionMessage = ref('');
const workshopError = ref('');

const workshops = ref([
  { name: 'Summer Drama Project - 12th - 17th August 10am - 4pm', selected: true, numChildren: '', ageChildren: '', foodAllergies: '', dietaryRequirements: '' },
]);

const parentInfo = ref({
  name: '',
  address: '',
  phone: '',
  email: '',
});

const finished = ref(false);

const errors = ref({});
const photoConsentList = ref([])
const newEntry = ref({
  parentName: '',
  childOrParentName: '',
  contactNumber: ''
});

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
        workshopError.value = 'Please fill in all required fields.';
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
const backgroundImage = ref(backgroundImageUrl);
console.log(backgroundImage.value)
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

  photoConsentList.value.forEach((entry, index) => {
    formData.append(`photo_consent_parent${index + 1}`, entry.parentName);
    formData.append(`photo_consent_child_or_parent${index + 1}`, entry.childOrParentName);
    formData.append(`photo_consent_contact${index + 1}`, entry.contactNumber);
  })

  try {
    // submissionStatus.value = 'success';
    // submissionMessage.value = 'Form submitted successfully!  Please wait - You will be redirected back to dgra.co.uk';
    // setTimeout(() => window.location.href="https://dgra.co.uk",3000)
    // return
    const response = await fetch('https://api.n1g3.com/api/drama-form', {
      method: 'POST',
      body: formData,
    });

    const data = await response.json();

    if (response.ok) {
      finished.value=true
      submissionStatus.value = 'success';
      submissionMessage.value = 'Form submitted successfully!  Please wait - You will be redirected back to dgra.co.uk';
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
      setTimeout(() => window.location.href="https://dgra.co.uk",3000)
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

const addToPhotoConsentList = () => {
  photoConsentList.value.push({ ...newEntry.value });
  newEntry.value = { parentName: '', childOrParentName: '', contactNumber: '' };
};

const removeEntry = (index) => {
  // remove entry from photoConsentList at the given index
  photoConsentList.value.splice(index, 1);

}

</script>
<template>
  <div class="bg-container min-w-full">
  <div class="container">
    <h2 class="text-2xl font-bold text-center mb-4">Summer Drama Project Registration - Summer 2024</h2>

    <div v-if="currentStep === 1">
      <h3 class="text-xl font-semibold mb-2">Step 1: Confirm Attendee Requirements</h3>
      <div v-for="(workshop, index) in workshops" :key="index" class="mb-2">
        <label class="flex items-center space-x-2">
          <input type="checkbox" v-model="workshop.selected" class="hidden checkbox checkbox-primary" />
          <span class="text-center w-full">{{ workshop.name }}</span>
        </label>
        <div v-if="workshop.selected" class="ml-24 mt-2 space-y-2 flex flex-col mx-auto w-full ">
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
      <h3 class="text-xl font-semibold mb-2">Step 3: Photographic/Video/Film Consent</h3>
      <div class="space-y-4">
        <p class="font-light">
          Doddington Grove Resident Association (DGRA) is asking for your permission to take and publish photographs or films of you or
          your child in electronic and printed publications. Photographs or films of you or your child will only be used with your consent and in
          the correct context by our organisation and/or Southwark Council.
        </p>
        <p>
          I give Doddington Grove Resident Association and Southwark Council permission to publish photographs or films bearing
          my image or the image of my child. I understand that I can withdraw my permission at any time.
          <span class="font-bold">Photographer/videographer: DGRA and Blue Elephant Theatre</span>
        </p>
        <div>
          <div class="mt-2 space-y-2 flex flex-col mx-auto w-full">
            <input v-model="newEntry.parentName" type="text" placeholder="Parent Name" class="input input-bordered w-full max-w-full" />
            <input v-model="newEntry.childOrParentName" type="text" placeholder="Child's/Parent's name (person to be photographed/filmed)" class="input input-bordered w-full max-w-full" />
            <input v-model="newEntry.contactNumber" type="text" placeholder="Contact Number" class="input input-bordered w-full max-w-full" />
          </div>
          <div class="flex mt-3">
            <button @click="addToPhotoConsentList" class="text-sm">Add to Photo Consent List</button>
          </div>
        </div>

        <div class="overflow-x-auto">
          <table v-if="photoConsentList.length" class="table w-2/3">
            <thead>
            <tr>
              <th>Child's/Parent's Name</th>
              <th>Action</th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(entry, index) in photoConsentList" :key="index">
              <td>{{ entry.childOrParentName }}</td>
              <td>
                <button @click="removeEntry(index)" class="btn btn-sm btn-error">Remove</button>
              </td>
            </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <div v-if="currentStep === 4">
      <h3 class="text-xl font-semibold mb-2">Step 4: Review and Submit</h3>
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
            <span class="label-text">You agree to receive e-mail communication from us by filling out this form and
understand that your contact information will be stored with us in relation to this project.</span>
            <input v-model="agreeToTerms" type="checkbox" class=" checked checkbox checkbox-primary" />
          </label>
        </div>
      </div>
    </div>



    <div v-if="submissionStatus" class="text-zinc-50" :class="['alert', submissionStatus === 'success' ? 'alert-success' : 'alert-error', 'mt-4']">
      <span>{{ submissionMessage }}</span>
    </div>
    <div class="mt-4 space-x-2">
      <button v-if="currentStep > 1" @click="previousStep">Previous</button>
      <button v-if="currentStep < 4 && !finished" @click="nextStep">Next</button>
      <button v-if="currentStep === 4" @click="submitForm" :disabled="!agreeToTerms">Submit</button>
    </div>
  </div>

  </div>
</template>
<style scoped>
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
}

.container {
  width: 100%;
  @apply bg-zinc-50 mx-auto p-10 rounded-3xl items-center justify-center mt-12;
}

button {
  background-color:rgb(252,175,59);
  color: rgb(1,1,1);
  text-transform: uppercase;
  font-weight:700;
}

.bg-container {
  /* background-image: url('/flowersbg.webp'); */
  background-color: rgb(237,237,237);
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  background-blend-mode: exclusion;
  width: 100%;
  height: 100vh;
  min-height: 100%;
  position: fixed;
  top: 0;
  left: 0;
}
.bg-container {
  background-color: rgb(237,237,237);

/*   background-image: url('/flowersbg.webp');*/
  background-size: cover;
  background-position: center;
  background-repeat: repeat;
  width:100% !important;
  height:100vh;
}
</style>