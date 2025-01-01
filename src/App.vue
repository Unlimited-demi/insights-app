<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-purple-50">
    <!-- Main Container -->
    <div class="container mx-auto px-4 py-8">
      <!-- Header -->
      <header class="text-center mb-12">
        <h1 class="text-5xl font-bold text-gray-800">
          Data<span class="text-purple-600">Insights</span>
        </h1>
        <p class="mt-2 text-gray-600">
          Upload your CSV file to get actionable insights.
        </p>
      </header>

      <!-- Upload Card -->
      <div
        class="max-w-2xl mx-auto bg-white rounded-xl shadow-lg overflow-hidden"
      >
        <div
          class="p-8 border-2 border-dashed border-gray-300 rounded-lg hover:border-purple-500 transition-all duration-300"
          @dragover.prevent
          @drop.prevent="handleFileDrop"
        >
          <div v-if="!file" class="text-center">
            <p class="mt-4 text-gray-600">Drag & drop your CSV file here</p>
            <button
              @click="$refs.fileInput.click()"
              class="mt-4 px-6 py-2 bg-purple-500 text-white rounded-lg hover:bg-purple-600 transition duration-200"
            >
              Browse Files
            </button>
            <input
              class="hidden"
              type="file"
              ref="fileInput"
              @change="handleFileUpload"
              accept=".csv"
            />
          </div>
          <div v-else class="flex items-center justify-between">
            <div class="flex items-center space-x-4">
              <span class="text-gray-700">{{ file.name }}</span>
            </div>
            <button
              @click="file = null"
              class="text-red-500 hover:text-red-600"
            >
              Remove
            </button>
          </div>
        </div>

        <!-- Model Selection Dropdown -->
        <div class="p-4 border-t border-gray-200">
          <label for="modelType" class="block text-sm font-medium text-gray-700"
            >Select Model</label
          >
          <select
            id="modelType"
            v-model="modelType"
            class="mt-1 block w-full py-2 px-3 border border-gray-300 bg-white rounded-md shadow-sm focus:outline-none focus:ring-purple-500 focus:border-purple-500 sm:text-sm"
          >
            <option value="gbm">GBM (Classification)</option>
            <option value="gmm">GMM (Clustering)</option>
          </select>
        </div>

        <!-- Progress Bar -->
        <div v-if="loading" class="p-4">
          <div class="w-full bg-gray-200 rounded-full h-2.5">
            <div
              class="bg-purple-500 h-2.5 rounded-full"
              :style="{ width: progress + '%' }"
            ></div>
          </div>
          <p class="mt-2 text-sm text-gray-600 text-center">
            {{ progressMessage }}
          </p>
        </div>

        <!-- Submit Button -->
        <button
          class="w-full py-3 bg-gradient-to-r from-purple-500 to-blue-500 text-white font-semibold hover:from-purple-600 hover:to-blue-600 transition-all duration-300"
          @click="submitFile"
          :disabled="!file || loading"
        >
          <span v-if="!loading">Get Insights</span>
          <div v-else class="flex items-center justify-center space-x-2">
            <span>Processing...</span>
          </div>
        </button>
      </div>

      <!-- Insights Section -->
      <transition
        enter-active-class="transition-all duration-700 ease-out"
        enter-from-class="opacity-0 transform translate-y-20"
        enter-to-class="opacity-100 transform translate-y-0"
      >
        <div
          v-if="insights"
          class="mt-12 grid grid-cols-1 md:grid-cols-2 gap-6"
        >
          <!-- GBM Insights -->
          <template v-if="modelType === 'gbm'">
            <!-- Target Variable Card -->
            <div
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow duration-300"
            >
              <h2 class="text-xl font-bold text-gray-800 mb-4">
                Target Variable
              </h2>
              <p class="text-gray-700">{{ insights.target_variable }}</p>
            </div>

            <!-- Recommendations Card -->
            <div
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow duration-300 col-span-1 md:col-span-2"
            >
              <h2 class="text-xl font-bold text-gray-800 mb-4">Recommendations</h2>
              <div v-if="insights.model_insights.recommendations">
                <div
                  v-for="(recommendation, index) in insights.model_insights.recommendations"
                  :key="index"
                  class="mb-6 p-4 bg-gray-50 rounded-lg"
                >
                  <h3 class="text-lg font-semibold text-purple-600 mb-2">
                    {{ recommendation.area }}
                  </h3>
                  <p class="text-gray-700 mb-2">
                    <strong>Action:</strong> {{ recommendation.action }}
                  </p>
                  <p class="text-gray-700">
                    <strong>Recommendation:</strong> {{ recommendation.recommendation }}
                  </p>
                </div>
              </div>
            </div>

            <!-- Top Features Card -->
            <div
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow duration-300"
            >
              <h2 class="text-xl font-bold text-gray-800 mb-4">Top Features</h2>
              <ul class="space-y-2">
                <li
                  v-for="feature in insights.top_features"
                  :key="feature.feature"
                  class="text-gray-700"
                >
                  <span class="font-medium">{{ feature.feature }}</span>
                  (Importance: {{ feature.importance.toFixed(4) }})
                </li>
              </ul>
            </div>

            <!-- Model Insights Card -->
            <div
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow duration-300 col-span-1 md:col-span-2"
            >
              <h2 class="text-xl font-bold text-gray-800 mb-4">
                Model Insights
              </h2>
              <div v-if="insights.model_insights">
                <!-- Display insights -->
                <div v-if="insights.model_insights.insights" class="mb-4">
                  <h3 class="text-lg font-semibold text-gray-800 mb-2">
                    Insights
                  </h3>
                  <ul class="space-y-2">
                    <li
                      v-for="(text, title) in insights.model_insights.insights"
                      :key="title"
                    >
                      <strong class="text-purple-600">{{ title }}:</strong>
                      {{ text }}
                    </li>
                  </ul>
                </div>
              </div>
            </div>

            <!-- Performance Metrics Card -->
            <div
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow duration-300"
            >
              <h2 class="text-xl font-bold text-gray-800 mb-4">
                Performance Metrics
              </h2>
              <p class="text-gray-700">
                Accuracy: {{ insights.performance_metrics.accuracy.toFixed(4) }}
              </p>
            </div>

            <!-- Sample Predictions Card -->
            <div
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow duration-300"
            >
              <h2 class="text-xl font-bold text-gray-800 mb-4">
                Sample Predictions
              </h2>
              <ul class="space-y-2">
                <li
                  v-for="(prediction, index) in insights.sample_predictions
                    .predictions"
                  :key="index"
                  class="text-gray-700"
                >
                  Prediction {{ index + 1 }}: {{ prediction.toFixed(6) }}
                </li>
              </ul>
            </div>
          </template>

          <!-- GMM Insights -->
          <template v-if="modelType === 'gmm'">
            <!-- Number of Clusters Card -->
            <div
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow duration-300"
            >
              <h2 class="text-xl font-bold text-gray-800 mb-4">
                Number of Clusters
              </h2>
              <p class="text-gray-700">{{ insights.n_components }}</p>
            </div>

            <!-- Cluster Information Card -->
            <div
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow duration-300"
            >
              <h2 class="text-xl font-bold text-gray-800 mb-4">
                Cluster Information
              </h2>
              <ul class="space-y-2">
                <li
                  v-for="(cluster, id) in insights.cluster_info"
                  :key="id"
                  class="text-gray-700"
                >
                  <span class="font-medium">Cluster {{ id }}:</span>
                  Size: {{ cluster.size }}, Centroid:
                  {{ JSON.stringify(cluster.centroid) }}
                </li>
              </ul>
            </div>

            <!-- Cluster Descriptions Card -->
            <div
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow duration-300 col-span-1 md:col-span-2"
            >
              <h2 class="text-xl font-bold text-gray-800 mb-4">
                Cluster Descriptions
              </h2>
              <ul class="space-y-2">
                <li
                  v-for="(description, id) in insights.cluster_descriptions"
                  :key="id"
                  class="text-gray-700"
                >
                  <strong class="text-purple-600">Cluster {{ id }}:</strong>
                  {{ description }}
                </li>
              </ul>
            </div>
          </template>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const API_URL = import.meta.env.VITE_API_URL || "http://localhost:8080";
const file = ref(null);
const insights = ref(null);
const loading = ref(false);
const progress = ref(0);
const progressMessage = ref("");
const modelType = ref("gbm");

const handleFileDrop = (event) => {
  const droppedFile = event.dataTransfer.files[0];
  if (droppedFile && droppedFile.type === "text/csv") {
    file.value = droppedFile;
  }
};

const handleFileUpload = (event) => {
  file.value = event.target.files[0];
};

const submitFile = async () => {
  if (!file.value) {
    alert("Please select a file to upload.");
    return;
  }

  loading.value = true;
  progress.value = 0;
  progressMessage.value = "Starting processing...";
  insights.value = null;

  const formData = new FormData();
  formData.append("file", file.value);

  try {
    const eventSource = new EventSource(
      `${API_URL}/stream-insights?model_type=${modelType.value}`
    );

    eventSource.onmessage = (event) => {
      if (event.data.startsWith("data: ")) {
        const data = event.data.replace("data: ", "").trim();
        if (data.startsWith("{")) {
          // Parse JSON insights
          insights.value = JSON.parse(data);
          loading.value = false;
          eventSource.close();
        } else {
          // Update progress
          progressMessage.value = data;
          if (data.includes("step")) {
            progress.value = (parseInt(data.match(/\d+/)[0]) / 5) * 100;
          }
        }
      }
    };

    eventSource.onerror = () => {
      eventSource.close();
      loading.value = false;
      alert("Failed to stream insights. Please try again.");
    };
  } catch (error) {
    console.error("Error streaming insights:", error);
    alert("Failed to stream insights. Please try again.");
    loading.value = false;
  }
};
</script>

<style>
/* Add custom styles if needed */
</style>