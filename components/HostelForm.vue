<script setup>
import {onMounted, reactive, ref, watch} from 'vue';
import { z } from 'zod';

const form = reactive({
  levels: {},
});
const errors = reactive({});

const isPopupVisible = ref(false);

const previousQuestions = [
  {
    label: "Building Name",
    type: "text",
    placeholder: "Building name",
    id : "name"
  },
  {
    label: "Gender",
    type: "select",
    options: [{value : "male", label: "Male"}, {value : "female", label: "Female"}],
    placeholder: "Select your gender",
    id : "gender"
  },
  {
    label: "Capacity",
    type: "select",
    options: [{value: "1", label: "One"},{value: "2", label: "Two"}, {value: "3", label: "Three"}, {value: "4", label: "Four"}],
    placeholder: "Select capacity",
    required: true,
    id : "capacity"
  },
  {
    label: "Number of Rooms for Each Level",
    type: "multi-select",
    levels: [
      { label: "Level 1", key: "level1" },
      { label: "Level 2", key: "level2" },
      { label: "Level 3", key: "level3" },
      { label: "Level 4", key: "level4" },
    ],
    placeholder: "Select Rooms for Each Level",
    required: true,
    id :"level"
  },
];

const formSchema = z.object({
  "name": z
      .string()
      .regex(/^\d{2}[A-Z]$/, "Hostel name must be two numbers followed by one uppercase letter")
      .nonempty("Hostel Name is required"),
  "gender" : z.string().nonempty("Gender is required"),
  "capacity" : z.string().nonempty("Capacity is required"),
});

previousQuestions.forEach((question) => {
  if (question.type === 'multi-select') {
    question.levels.forEach((level) => {
      form.levels[level.key] = "";
      errors[level.key] = "";
    });
  } else {
    form[question.id] = "";
    errors[question.id] = "";
  }
});

function validateField(field, value) {
  try {
    formSchema.shape[field]?.parse(value || form[field]);
    errors[field] = "";
  } catch (error) {
    errors[field] = error.errors ? error.errors[0].message : error.message;
  }
}

previousQuestions.forEach((question) => {
  if (question.type === 'multi-select') {
    question.levels.forEach((level) => {
      watch(() => form.levels[level.key], () => validateField(level.key, form.levels[level.key]));
    });
  }  else {
    watch(() => form[question.id], () => validateField(question.id, form[question.id]));
  }
});

async function handleSubmit() {
  const api = useApi();
  form.Date = new Date().toLocaleDateString("en-GB");

  const validationResults = formSchema.safeParse(form);
  if (validationResults.success) {
    try {
      console.log("Sending API Request...");

      const payload = {
        name: form.name,
        capacity: form.capacity,
        gender: form.gender,
        levels: []
      }

      for (const level in form.levels) {
        payload.levels.push({
          number: parseInt(level.replace('level', '')),
          rooms: form.levels[level]
        })
      }

      const response = await api.post("/hostels/", payload);
      console.log("Response Data:", response.data);
      isPopupVisible.value = true;
      Object.keys(form).forEach((key) => (form[key] = ""));
      location.reload()
    } catch (error) {
      isPopupVisible.value = false;
      console.error("Error occurred:", error);
      if (error.response) {
        isPopupVisible.value = false;
        console.error("Backend Error:", error.response.data);
        alert(`Error: ${error.response.data.detail || "Unable to submit the form."}`);
        console.log("Response Data:", response.data.value);
      } else if (error.request) {
        isPopupVisible.value = false;
        console.error("No response from the server:", error.request);
        alert("Server is not responding. Please try again later.");
      } else {
        isPopupVisible.value = false;
        console.error("Request Setup Error:", error.message);
        alert("An error occurred while submitting the form. Please try again.");
      }
    }
  } else {
    console.log('Validation Errors:', validationResults.error.errors);
    isPopupVisible.value = false;
    alert("Please correct the errors in the form.");
  }
}

definePageMeta({
  middleware: 'auth',
});

</script>

<template>
  <div class="new-student-sec">
    <div class="container">
      <div class="form-header">
        <h2>New Building Form</h2>
      </div>
      <div class="box-form">
        <form @submit.prevent="handleSubmit">
          <div class="form-container">
            <div
                class="info"
                v-for="(question, index) in previousQuestions"
                :key="index"
            >
              <label
                  class="question-title"
                  :for="question.label"
              >
                {{ question.label }}
              </label>

              <input
                  v-if="question.type === 'text'"
                  :type="question.type"
                  v-model="form[question.id]"
                  :placeholder="question.placeholder"
                  :id="question.id"
                  @input="validateField(question.id)"
              />

              <select
                  v-if="question.type === 'select'"
                  v-model="form[question.id]"
                  :id="question.id"
                  @change="validateField(question.id)"
              >
                <option
                    value=""
                    disabled
                >
                  {{ question.placeholder }}
                </option>
                <option
                    v-for="option in question.options"
                    :key="option.label"
                    :value="option.value"
                >
                  {{ option.label }}
                </option>
              </select>

              <div v-else-if="question.type === 'multi-select'">
                <div
                    v-for="level in question.levels"
                    :key="level.key"
                    class="level-input"
                >
                  <label class="form-label">
                    {{ level.label }}
                  </label>
                  <select
                      v-model="form.levels[level.key]"
                      :required="question.required"
                      class="form-select"
                      @change="validateField(level.key)"
                  >
                    <option
                        value=""
                        disabled
                    >
                      {{ question.placeholder }}
                    </option>
                    <option
                        v-for="i in 30"
                        :key="i"
                        :value="i"
                    >
                      {{ i }}
                    </option>
                  </select>
                  <span
                      v-if="errors[level.id]"
                      class="error"
                  >
                    {{ errors[level.id] }}
                  </span>
                </div>
              </div>

              <span
                  v-if="errors[question.id]"
                  class="error"
              >
                {{ errors[question.id] }}
              </span>
            </div>
          </div>

          <div>
            <button
                class="submit"
                type="submit"
            >
              Submit
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: block;
  margin: 0;
  padding: 0;
  width: 100%;
  max-width: 1300px;
}

@media (max-width: 1200px) {
  .container {
    margin: 3rem 0;
    width: 100%;
  }
}

.form-header {
  width: 100%;
  text-align: center;
  margin: 0;
  padding: 0.5rem 0;
  background-color: var(--text-hover-color);
  font-size: 1.5rem;
  color: var(--primary-color);
}

.container .box-form {
  width: 90%;
  margin: 0 auto;
  padding: 1rem 0;
}

@media (max-width: 1200px) {
  .container div {
    display: block;
  }
}

.form-container {
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  gap: 0.5rem 1rem;
}

.info {
  flex-basis: calc(100% - 10px);
  display: block;
}

.form-container .question-title {
  font-size: 1rem;
  color: var(--primary-hover-color);
}

.form-container input,
.form-container select {
  width: 100%;
  padding: 0.5rem 1rem;
  border: 2px solid var(--text-light-color);
  border-radius: 5px;
  outline: none;
}

.error {
  color: red;
  font-size: 1rem;
}

.submit {
  width: 100%;
  margin-top: 2rem;
  padding: 0.5rem 2rem;
  font-size: 1.2rem;
  border-radius: 1rem 0;
  background-color: var(--primary-hover-color);
  color: var(--text-hover-color);
}

.submit:hover {
  background-color: var(--primary-color);
  transition: 0.3s ease-in-out;
}
</style>