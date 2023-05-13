<template>
  <v-container class="fill-height">
    <v-responsive class="d-flex align-center text-center fill-height">
      <v-row>
        <v-col>
          <v-textarea
            clearable
            h="full"
            label="Type any language..."
            variant="outlined"
            v-model="question"
            @keyup.enter="createCompletion"
          ></v-textarea>
        </v-col>
      </v-row>
      <v-row v-if="errorMessage">
        <v-alert color="#C51162" theme="dark" icon="mdi-alert" border>
          {{ errorMessage }}
        </v-alert>
      </v-row>

      <v-row v-for="(display, index) in displays" :key="index">
        <v-col>
          <v-alert
            border="top"
            :border-color="display.role == 'user' ? 'black' : 'success'"
            elevation="2"
          >
            <pre>{{ display.content }}</pre>
          </v-alert>
        </v-col>
      </v-row>
      <v-row v-if="loading">
        <v-col>
          <v-progress-circular
            indeterminate
            color="primary"
          ></v-progress-circular>
        </v-col>
      </v-row>
    </v-responsive>
  </v-container>
</template>

<script setup>
import { Configuration, OpenAIApi } from "openai";
import { ref } from "vue";
const configuration = new Configuration({
  apiKey: import.meta.env.VITE_APP_OPENAI_API_KEY,
});
const openai = new OpenAIApi(configuration);
const response = ref({});
const errorMessage = ref("");
const question = ref("");
const loading = ref(false);
const displays = ref([]);

async function createChatCompletion() {
  loading.value = true;
  try {
    displays.value.push({
      role: "user",
      content: question.value,
    });
    question.value = "";

    response.value = await openai.createChatCompletion({
      model: "gpt-3.5-turbo-0301",
      messages: displays.value,
      temperature: 1,
    });
    displays.value.push({
      role: "assistant",
      content: response.value.data.choices[0].message.content,
    });
  } catch (error) {
    if (error.response) {
      errorMessage.value = error.response.data;
    } else {
      errorMessage.value = error.message;
    }
  } finally {
    loading.value = false;
  }
}

async function createCompletion() {
  loading.value = true;
  try {
    const prompt = question.value;
    question.value = "";

    const response = await openai.createCompletion({
      model: "text-davinci-003",
      prompt: prompt,
      temperature: 0.7,
      top_p: 1,
      frequency_penalty: 0,
      presence_penalty: 0,
    });

    displays.value.push({
      role: "assistant",
      content: response.data.choices[0].text,
    });
  } catch (error) {
    if (error.response) {
      errorMessage.value = error.response.data;
    } else {
      errorMessage.value = error.message;
    }
  } finally {
    loading.value = false;
  }
}
</script>
