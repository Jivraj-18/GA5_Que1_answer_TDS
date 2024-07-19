<template>
  <div class="proxy-query-container">
    <h2>Proxy Query</h2>
    <form @submit.prevent="submitQuery">
      <div class="input-group">
        <label for="apiToken">API Token:</label>
        <input
          type="password"
          id="apiToken"
          v-model="apiToken"
          required
        />
      </div>
      <div class="input-group">
        <label for="question">Question:</label>
        <input
          type="textarea"
          id="question"
          v-model="question"
          required
        />
      </div>
      <button type="submit" :disabled="loading">
        {{ loading ? 'Loading...' : 'Submit' }}
      </button>
    </form>
    <div v-if="result" class="result">
      <h3>Result:</h3>
      <p>{{ result }}</p>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';

export default {
  name: 'App',
  props: {
    proxyUrl: {
      type: String,
      required: true,
      default: "https://aiproxy.sanand.workers.dev/openai/v1/models"
    }
  },
  setup(props) {
    const apiToken = ref('');
    const question = ref('');
    const result = ref('');
    const loading = ref(false);

    const submitQuery = async () => {
      loading.value = true;
      result.value = '';
      console.log(props)
      try {
        // Uncomment and configure the fetch request if needed
         const response = await fetch(props.proxyUrl, {
           method: 'POST',
           headers: {
             'Content-Type': 'application/json',
             'Authorization': `Bearer ${apiToken.value}`
           },
           body: JSON.stringify({ question: question.value })
         });

         if (!response.ok) {
           throw new Error('Failed to fetch data');
         }

         const data = await response.json();
         const models = data.data;

        // Use hardcoded models for demonstration
        
        const params = extractParameters(question.value);

        // Filter models created before the specified date and sort them
        // const filteredModels = models
        //   .filter(model => model.created < new Date(params.parameter1).getTime() / 1000)
        //   .sort((a, b) => b.created - a.created);

        // Calculate points based on parameters
        const points = evaluateQuestion(models, params);

        result.value = `Total points: ${points}`;
      } catch (error) {
        result.value = `Error: ${error.message}`;
      } finally {
        loading.value = false;
      }
    };

    const extractParameters = (question) => {
      const regex = /created before ([\d-]+).+4 points if ([\w-]+) was created on ([\d-]+).+2 points if ([\w-]+) is located at index (\d+).+1 point if ([\w-]+) was created (\d+) models before ([\w-]+)/s;
      const match = question.match(regex);
      if (match) {
        return {
          parameter1: match[1],
          parameter2: match[2],
          parameter3: match[3],
          parameter4: match[4],
          parameter5: parseInt(match[5], 10),
          parameter6: match[6],
          parameter7: parseInt(match[7], 10),
          parameter8: match[8]
        };
      }
      
      return {};
    };

    const evaluateQuestion = (models, params) => {
      let points = 0;

      // Check first condition
      const model2 = models.find(model => model.id === params.parameter2);
      if (model2 && new Date(model2.created * 1000).toISOString().split('T')[0] === params.parameter3) {
        points += 4;
        console.log("condition1-model name",model2)
        console.log("condition1-machine date", new Date(model2.created * 1000).toISOString().split('T')[0])
        console.log("condition1-target date", params.parameter3)
      }

      // Check second condition
      console.log("condition2 ",params.parameter4)
      console.log("condition2- what is on 15th index",models.findIndex(model => model.id == params.parameter4))
      console.log("condition2-we are looking for this on 15th index",params.parameter5)
      
      if (models.findIndex(model => model.id === params.parameter4) === params.parameter5) {
        points += 2;
        console.log("condition2 true")
      }

      // Check third condition
      console.log("condition3-created before", params.parameter6)
      console.log(models.findIndex(model => model.id === params.parameter6))
      console.log("condition3- created after", params.parameter8)
      console.log(models.findIndex(model => model.id === params.parameter8))
      console.log(params.parameter7)
      const model6Index = models.findIndex(model => model.id === params.parameter6);
      const model8Index = models.findIndex(model => model.id === params.parameter8);
      
      let value = model6Index - model8Index
      
      console.log(params)
      if (value >= 0) { 
        console.log("model6-model8 = param7       ",model6Index, "-", model8Index, "=", params.parameter7)
        if (model6Index != -1 && model8Index != -1 && (value == params.parameter7)) {
          points += 1;
          console.log("condition3 true")
        }
      }
      return points;
    };

    return {
      apiToken,
      question,
      result,
      loading,
      submitQuery
    };
  }
};
</script>

<style scoped>
.proxy-query-container {
  font-family: Arial, sans-serif;
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f0f0f0;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h2 {
  text-align: center;
  color: #333;
}

.input-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

button {
  width: 100%;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.result {
  margin-top: 20px;
  padding: 10px;
  background-color: #e9ecef;
  border-radius: 4px;
}
</style>
