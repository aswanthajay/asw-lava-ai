<script lang="ts">
	import { onMount } from 'svelte';
	let image: File | null = null;
	let imageUrl: string | null = null;
	let question: string = '';
	let isDragging: boolean = false;
	let description: string | null = null;
	let isLoading: boolean = false;

	const MAX_WIDTH = 800;
	const MAX_HEIGHT = 800;

	const resizeImage = (file: File): Promise<File> => {
		return new Promise((resolve) => {
			const img = new Image();
			img.src = URL.createObjectURL(file);
			img.onload = () => {
				const canvas = document.createElement('canvas');
				let width = img.width;
				let height = img.height;

				if (width > height) {
					if (width > MAX_WIDTH) {
						height = Math.round((height * MAX_WIDTH) / width);
						width = MAX_WIDTH;
					}
				} else {
					if (height > MAX_HEIGHT) {
						width = Math.round((width * MAX_HEIGHT) / height);
						height = MAX_HEIGHT;
					}
				}

				canvas.width = width;
				canvas.height = height;
				const ctx = canvas.getContext('2d');
				ctx.drawImage(img, 0, 0, width, height);

				canvas.toBlob((blob) => {
					if (blob) {
						resolve(new File([blob], file.name, { type: file.type }));
					}
				}, file.type);
			};
		});
	};

	const handleFileUpload = async (event: Event) => {
		const target = event.target as HTMLInputElement;
		if (target.files && target.files.length > 0) {
			image = target.files[0];
			image = await resizeImage(image);
			imageUrl = URL.createObjectURL(image);
		}
	};

	const handleDrop = async (event: DragEvent) => {
		event.preventDefault();
		isDragging = false;
		if (event.dataTransfer && event.dataTransfer.files.length > 0) {
			image = event.dataTransfer.files[0];
			image = await resizeImage(image);
			imageUrl = URL.createObjectURL(image);
		}
	};

	const handleDragOver = (event: DragEvent) => {
		event.preventDefault();
		isDragging = true;
	};

	const handleDragLeave = () => {
		isDragging = false;
	};

	const handleKeyDown = (event: KeyboardEvent) => {
		if (event.key === 'Enter' || event.key === ' ') {
			event.preventDefault();
			document.getElementById('file-input')?.click();
		}
	};

	const submitForm = async () => {
		if (!image || !question) {
			alert('Please upload an image and ask a question.');
			return;
		}

		isLoading = true;
		const formData = new FormData();
		formData.append('image', image);
		formData.append('question', question);

		try {
			const response = await fetch('/api/ask', {
				method: 'POST',
				body: formData
			});
			type Result = {
				description: string;
			};
			const result: Result = await response.json();
			description = result.description;
		} catch (error) {
			console.error('Error submitting form:', error);
			description = 'An error occurred while processing your request.';
		} finally {
			isLoading = false;
		}
	};
</script>

<div class="container">
	<!-- Navigation Bar -->
	<nav class="navbar">
		<div class="navbar-brand">Aswanth Ajay</div>
		<div class="navbar-links">
			<a href="#home">Home</a>
			<a href="#about">About</a>
			<a href="#contact">Contact</a>
		</div>
	</nav>

	<h1>Vision Ai by Aswanth</h1>

	<div class="upload-area {isDragging ? 'dragging' : ''}" role="button" tabindex="0" on:drop={handleDrop} on:dragover={handleDragOver} on:dragleave={handleDragLeave} on:click={() => document.getElementById('file-input')?.click()} on:keydown={handleKeyDown} aria-label="Upload Area: Drag & Drop Image or Click to Upload">
		<input type="file" accept="image/*" on:change={handleFileUpload} style="display: none;" id="file-input" />
		{#if imageUrl}
			<img src={imageUrl} alt="Upload preview" />
		{:else}
			<label for="file-input">Drag & Drop Image or Click to Upload</label>
		{/if}
	</div>

	<input type="text" class="question-input" placeholder="Ask a question about the photo..." bind:value={question} />
	<button class="submit-button" on:click={submitForm}>Submit</button>

	{#if isLoading}
		<div class="loading-indicator">Processing your request...</div>
	{/if}

	{#if description}
		<div class="description">{description}</div>
	{/if}

	<div class="footer">
		<p>Built by Aswanth, Beta Version 0.11</p>
		<p>Learn more about <a href="https://votioncloud.online/workers-ai/privacy/" target="_blank">AI data and privacy</a></p>
		<p>👀 my code on github public by 2025</p>
	</div>
</div>

<style>
	/* Global Styling */
	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	body {
		font-family: 'Arial', sans-serif;
		background: linear-gradient(45deg, #ff6b6b, #f7b7a3, #f6d365);
		background-size: 400% 400%;
		animation: gradientAnimation 15s ease infinite;
		color: #fff;
	}

	/* Navigation Bar */
	.navbar {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 20px;
		background: rgba(0, 0, 0, 0.7);
		border-radius: 8px;
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
		position: sticky;
		top: 0;
		width: 100%;
	}

	.navbar-brand {
		font-size: 24px;
		font-weight: bold;
		color: #fff;
	}

	.navbar-links a {
		color: #fff;
		text-decoration: none;
		margin: 0 15px;
		font-size: 18px;
		transition: color 0.3s ease;
	}

	.navbar-links a:hover {
		color: #ff6b6b;
	}

	/* Content Styling */
	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 40px 20px;
		max-width: 1000px;
		margin: 0 auto;
	}

	h1 {
		color: #fff;
		font-size: 36px;
		margin-bottom: 30px;
	}

	.upload-area {
		width: 100%;
		max-width: 450px;
		height: 300px;
		border: 3px dashed #fff;
		display: flex;
		justify-content: center;
		align-items: center;
		margin-bottom: 30px;
		transition: background-color 0.3s;
		cursor: pointer;
		font-size: 20px;
		border-radius: 8px;
		background-color: rgba(255, 255, 255, 0.1); /* Added background to enhance visibility */
	}

	.upload-area.dragging {
		background-color: rgba(255, 255, 255, 0.2); /* Slightly darker when dragging */
	}

	.upload-area img {
		max-width: 100%;
		max-height: 100%;
		display: block;
		border-radius: 8px;
	}

	.question-input {
		width: 100%;
		max-width: 450px;
		padding: 15px;
		margin-bottom: 30px;
		font-size: 18px;
		border-radius: 8px;
		border: none;
		background-color: rgba(0, 0, 0, 0.3); /* Darker background */
		color: #fff;
	}

	.submit-button {
		padding: 15px 30px;
		background-color: #ff6b6b;
		color: white;
		border: none;
		border-radius: 5px;
		cursor: pointer;
		font-size: 18px;
		transition: background-color 0.3s ease;
	}

	.submit-button:hover {
		background-color: #ff3b3b;
	}

	.loading-indicator {
		margin-top: 30px;
		font-size: 18px;
		color: #fff;
	}

	.description {
		margin-top: 30px;
		font-size: 20px;
		font-weight: bold;
		padding: 20px;
		border: 3px solid #fff;
		border-radius: 5px;
		background-color: rgba(255, 255, 255, 0.1);
		color: #ff6b6b;
		max-width: 80%;
		text-align: center;
	}

	.footer {
		margin-top: 60px;
		text-align: center;
		font-size: 16px;
		color: #fff;
	}

	.footer p {
		margin: 10px 0;
	}

	.footer a {
		color: #ff6b6b;
		text-decoration: none;
	}

	.footer a:hover {
		text-decoration: underline;
	}

	@keyframes gradientAnimation {
		0% {
			background-position: 0% 50%;
		}
		50% {
			background-position: 100% 50%;
		}
		100% {
			background-position: 0% 50%;
		}
	}
</style>


