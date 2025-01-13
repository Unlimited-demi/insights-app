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

            <!-- Recommended Features Card -->
            <div
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow duration-300"
            >
              <h2 class="text-xl font-bold text-gray-800 mb-4">
                Recommended Features
              </h2>
              <ul class="list-disc list-inside text-gray-700">
                <li
                  v-for="feature in insights.recommended_features"
                  :key="feature"
                >
                  {{ feature }}
                </li>
              </ul>
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

                <!-- Display recommendations -->
                <div v-if="insights.model_insights.recommendations">
                  <h3 class="text-lg font-semibold text-gray-800 mb-2">
                    Recommendations
                  </h3>
                  <ul class="space-y-2">
                    <li
                      v-for="(text, topic) in insights.model_insights
                        .recommendations"
                      :key="topic"
                    >
                      <strong class="text-purple-600">{{ topic }}:</strong>
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

            <!-- Evaluation Metrics Card -->
            <div
              class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow duration-300"
            >
              <!-- <h2 class="text-xl font-bold text-gray-800 mb-4">
                Evaluation Metrics
              </h2>
              <p class="text-gray-700">
                Silhouette Score:
                {{ insights.evaluation_metrics.silhouette_score.toFixed(4) }}
              </p>
              <p class="text-gray-700">
                Davies-Bouldin Index:
                {{
                  insights.evaluation_metrics.davies_bouldin_index.toFixed(4)
                }}
              </p> -->
            </div>
          </template>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

console.log("Environment Variables:", import.meta.env);

// Use VITE_API_URL from .env
const API_URL = import.meta.env.VUE_API_URL || "http://localhost:8080";
console.log("API URL:", API_URL);

const file = ref(null);
const insights = ref(null);
const loading = ref(false);
const fileInput = ref(null);
const modelType = ref("gbm"); // Default to GBM

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
  const formData = new FormData();
  formData.append("file", file.value);

  try {
    const endpoint =
      modelType.value === "gbm" ? "/insights" : "/clustering-insights";
    const response = await axios.post(
      `${API_URL}/insights${endpoint}`,
      formData,
      {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      }
    );
    console.log("Insights fetched successfully:", response.data);
    insights.value = response.data;
  } catch (error) {
    console.error("Error fetching insights:", error);
    alert("Failed to fetch insights. Please try again.");
  } finally {
    loading.value = false;
  }
};
</script>
