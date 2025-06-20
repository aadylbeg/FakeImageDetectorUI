<template>
	<div class="container mx-auto p-4 max-w-4xl">
		<header class="text-center mb-8">
			<h1 class="text-3xl font-bold mb-2">AI Image Detector</h1>
			<p class="text-gray-600">
				Upload an image or provide a URL to check if it was generated by AI
			</p>
		</header>

		<div class="bg-white rounded-lg shadow-md p-6 mb-8">
			<div class="mb-6">
				<h2 class="text-xl font-semibold mb-4">Upload Image</h2>

				<div class="flex flex-col md:flex-row gap-4">
					<div class="flex-1">
						<div
							class="border-2 border-dashed border-gray-300 rounded-lg p-4 text-center cursor-pointer hover:bg-gray-50"
							@click="triggerFileInput"
							@dragover.prevent
							@drop.prevent="handleFileDrop"
						>
							<div v-if="!imageFile && !imageUrl">
								<svg
									class="mx-auto h-12 w-12 text-gray-400"
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke="currentColor"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										stroke-width="2"
										d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"
									/>
								</svg>
								<p class="mt-2">
									Drag and drop an image here, or click to select
								</p>
							</div>
							<div v-else class="relative">
								<img
									:src="previewUrl"
									class="max-h-64 mx-auto rounded"
									alt="Preview"
								/>
								<button
									@click.stop="clearImage"
									class="absolute top-2 right-2 bg-red-500 text-white rounded-full p-1"
									title="Remove image"
								>
									<svg
										xmlns="http://www.w3.org/2000/svg"
										class="h-5 w-5"
										viewBox="0 0 20 20"
										fill="currentColor"
									>
										<path
											fill-rule="evenodd"
											d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
											clip-rule="evenodd"
										/>
									</svg>
								</button>
							</div>
						</div>
						<input
							type="file"
							ref="fileInput"
							@change="handleFileSelect"
							accept="image/*"
							class="hidden"
						/>
					</div>

					<div class="flex-1">
						<div class="form-group">
							<label class="block text-sm font-medium mb-2"
								>Or enter image URL:</label
							>
							<div class="flex">
								<input
									v-model="imageUrl"
									type="text"
									class="flex-1 border rounded-l px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
									placeholder="https://example.com/image.jpg"
									@input="handleUrlInput"
								/>
								<button
									@click="clearUrl"
									class="bg-red-500 text-white px-2 rounded-r"
									title="Clear URL"
									v-if="imageUrl"
								>
									<svg
										xmlns="http://www.w3.org/2000/svg"
										class="h-5 w-5"
										viewBox="0 0 20 20"
										fill="currentColor"
									>
										<path
											fill-rule="evenodd"
											d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
											clip-rule="evenodd"
										/>
									</svg>
								</button>
							</div>
						</div>
					</div>
				</div>

				<div class="mt-4 text-center">
					<button
						@click="analyzeImage"
						class="bg-blue-600 hover:bg-blue-700 text-white px-6 py-2 rounded-lg transition-colors disabled:bg-gray-400"
						:disabled="isLoading || (!imageFile && !imageUrl)"
					>
						<span v-if="isLoading">
							<svg
								class="animate-spin -ml-1 mr-2 h-4 w-4 text-white inline-block"
								xmlns="http://www.w3.org/2000/svg"
								fill="none"
								viewBox="0 0 24 24"
							>
								<circle
									class="opacity-25"
									cx="12"
									cy="12"
									r="10"
									stroke="currentColor"
									stroke-width="4"
								></circle>
								<path
									class="opacity-75"
									fill="currentColor"
									d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
								></path>
							</svg>
							Analyzing...
						</span>
						<span v-else>Analyze Image</span>
					</button>
				</div>
			</div>
		</div>

		<div v-if="results" class="bg-white rounded-lg shadow-md p-6">
			<h2 class="text-xl font-semibold mb-4">Analysis Results</h2>

			<div class="flex flex-col md:flex-row gap-6">
				<div class="flex-1">
					<img
						:src="previewUrl"
						class="max-h-64 rounded mx-auto"
						alt="Analyzed image"
					/>
				</div>

				<div class="flex-1">
					<div class="mb-4">
						<div class="flex items-center justify-between">
							<span class="text-lg font-medium">
								{{ results.is_ai_generated ? "AI-Generated" : "Likely Real" }}
							</span>
							<span
								:class="[
									'px-3 py-1 rounded-full text-sm font-medium',
									results.is_ai_generated
										? 'bg-red-100 text-red-800'
										: 'bg-green-100 text-green-800',
								]"
							>
								{{ results.confidence }} confidence
								<!-- {{ (results.confidence * 100).toFixed(1) }}% confidence -->
							</span>
						</div>

						<div class="mt-3 w-full bg-gray-200 rounded-full h-2.5">
							<div
								class="h-2.5 rounded-full"
								:class="results.is_ai_generated ? 'bg-red-600' : 'bg-green-600'"
								:style="`width: ${results.confidence}`"
							></div>
						</div>
					</div>

					<h3 class="font-medium mt-6 mb-2">Analysis Metrics</h3>
					<div v-if="results.scores">
						<div
							v-for="(score, metric) in results.scores"
							:key="metric"
							class="mb-2"
						>
							<div class="flex justify-between mb-1">
								<span class="capitalize">{{ metric }} Analysis</span>
								<span>{{ (score * 100).toFixed(1) }}%</span>
							</div>
							<div class="w-full bg-gray-200 rounded-full h-1.5">
								<div
									class="h-1.5 rounded-full bg-blue-600"
									:style="`width: ${score * 100}%`"
								></div>
							</div>
						</div>
					</div>

					<div
						v-if="results.ai_indicators && results.ai_indicators.length > 0"
						class="mt-6"
					>
						<h3 class="font-medium mb-2">AI Indicators Detected</h3>
						<ul class="list-disc list-inside text-sm text-gray-700">
							<li
								v-for="(indicator, index) in results.ai_indicators"
								:key="index"
							>
								{{ indicator }}
							</li>
						</ul>
					</div>
				</div>
			</div>
		</div>

		<div
			v-if="error"
			class="mt-4 bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded"
		>
			<strong class="font-bold">Error:</strong>
			<span class="block sm:inline">{{ error }}</span>
		</div>
	</div>
</template>

<script>
export default {
	name: "App",
	data() {
		return {
			imageFile: null,
			imageUrl: "",
			previewUrl: "",
			isLoading: false,
			results: null,
			error: null,
		};
	},
	methods: {
		triggerFileInput() {
			this.$refs.fileInput.click();
		},

		handleFileSelect(event) {
			this.imageUrl = "";
			const file = event.target.files[0];
			if (file) {
				this.imageFile = file;
				this.previewUrl = URL.createObjectURL(file);
			}
		},

		handleFileDrop(event) {
			this.imageUrl = "";
			const file = event.dataTransfer.files[0];
			if (file && file.type.startsWith("image/")) {
				this.imageFile = file;
				this.previewUrl = URL.createObjectURL(file);
			}
		},

		handleUrlInput() {
			if (this.imageUrl) {
				this.imageFile = null;
				this.previewUrl = this.imageUrl;
			}
		},

		clearImage() {
			this.imageFile = null;
			if (this.previewUrl && !this.imageUrl) {
				URL.revokeObjectURL(this.previewUrl);
				this.previewUrl = "";
			}
		},

		clearUrl() {
			this.imageUrl = "";
			if (!this.imageFile) {
				this.previewUrl = "";
			}
		},

		async analyzeImage() {
			this.error = null;
			this.results = null;
			this.isLoading = true;

			try {
				const formData = new FormData();
				let response;

				if (this.imageFile) {
					formData.append("image", this.imageFile);
					response = await fetch("http://localhost:5000/get-image", {
						method: "POST",
						body: formData,
					});
				} else if (this.imageUrl) {
					response = await fetch("http://localhost:5000/get-image", {
						method: "POST",
						headers: {
							"Content-Type": "application/json",
						},
						body: JSON.stringify({ url: this.imageUrl }),
					});
				} else {
					throw new Error("No image selected");
				}

				if (!response.ok) {
					const errorData = await response.json();
					throw new Error(errorData.error || "Failed to analyze image");
				}

				this.results = await response.json();
				console.log(this.results);
			} catch (err) {
				this.error = err.message;
			} finally {
				this.isLoading = false;
			}
		},
	},
};
</script>
