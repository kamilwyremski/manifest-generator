<script lang="ts">
	import JSZip from 'jszip';

	const formData = {
		name: 'App Name',
		short_name: 'App',
		display: 'browser',
		description: '',
		scope: '/',
		start_url: '/',
		theme_color: '#000000',
		background_color: '#FFFFFF',
		icons: null as File | null
	};

	let errorImageMessage = '';

	const displayOptions = ['fullscreen', 'standalone', 'minimal-ui', 'browser'];

	$: isSubmitDisabled =
		!formData.name || !formData.short_name || !formData.start_url || !formData.background_color;

	function handleSubmit() {
		if (isSubmitDisabled) {
			return;
		}

		const iconSizes: number[] = [512, 384, 256, 184];
		const icons: { src: string; sizes: string; type: string }[] = [];

		function addIcon(img: HTMLImageElement, size: number): void {
			const canvas: HTMLCanvasElement = document.createElement('canvas');
			canvas.width = size;
			canvas.height = size;
			const ctx: CanvasRenderingContext2D | null = canvas.getContext('2d');
			if (ctx) {
				ctx.drawImage(img, 0, 0, size, size);
				const dataURL: string = canvas.toDataURL('image/png');
				icons.push({
					src: dataURL,
					sizes: `${size}x${size}`,
					type: 'image/png'
				});
			}
		}

		if (formData.icons) {
			for (const size of iconSizes) {
				const img: HTMLImageElement = new Image();
				img.onload = function () {
					addIcon(img, size);

					// After load add icons we create ZIP archive
					if (icons.length === iconSizes.length) {
						const manifest = JSON.stringify(
							{
								...formData,
								icons: icons.map((icon) => ({ ...icon, src: 'icons/icon-' + icon.sizes + '.png' }))
							},
							null,
							2
						);

						const zip = new JSZip();
						const iconFolder = zip.folder('icons');
						for (let i = 0; i < icons.length; i++) {
							iconFolder?.file(
								`icon-${icons[i].sizes}.png`,
								icons[i].src.substr(icons[i].src.indexOf(',') + 1),
								{ base64: true }
							);
						}

						zip.file('manifest.json', manifest);

						zip
							.generateAsync({ type: 'blob' })
							.then((blob) => {
								const url = URL.createObjectURL(blob);
								const a = document.createElement('a');
								a.href = url;
								a.download = 'manifest.zip';
								document.body.appendChild(a);
								a.click();
								document.body.removeChild(a);
							})
							.catch((error) => console.error('Error generating ZIP:', error));
					}
				};
				img.src = URL.createObjectURL(formData.icons);
			}
		} else {
			const blob = new Blob([JSON.stringify({ ...formData, icons: undefined }, null, 2)], {
				type: 'application/json'
			});
			const url = URL.createObjectURL(blob);

			const a = document.createElement('a');
			a.href = url;
			a.download = 'manifest.json';
			document.body.appendChild(a);
			a.click();
			document.body.removeChild(a);
			URL.revokeObjectURL(url);
		}
	}

	function handleFileChange(event: Event) {
		const input = event.target as HTMLInputElement;
		errorImageMessage = '';
		formData.icons = null;
		if (input.files && input.files.length > 0) {
			const file = input.files[0];

			if (!(file.type === 'image/png' || file.type === 'image/jpeg')) {
				errorImageMessage = 'The file is not a PNG or JPEG image.';
				return;
			}

			const image = new Image();

			image.onload = function () {
				const minDimension = Math.min(image.width, image.height);
				if (minDimension < 512) {
					errorImageMessage = 'The image must be at least 512x512 pixels.';
					return;
				}

				// If already square, use original file
				if (image.width === image.height) {
					formData.icons = file;
					return;
				}

				// Crop image to square
				const canvas = document.createElement('canvas');
				canvas.width = minDimension;
				canvas.height = minDimension;
				const ctx = canvas.getContext('2d');
				if (ctx) {
					const offsetX = (image.width - minDimension) / 2;
					const offsetY = (image.height - minDimension) / 2;
					ctx.drawImage(
						image,
						offsetX,
						offsetY,
						minDimension,
						minDimension,
						0,
						0,
						minDimension,
						minDimension
					);
					canvas.toBlob((blob) => {
						if (blob) {
							formData.icons = new File([blob], file.name, { type: 'image/png' });
						}
					}, 'image/png');
				}
			};
			image.src = URL.createObjectURL(file);
		}
	}

	$: manifestPreview = JSON.stringify(
		{
			name: formData.name,
			short_name: formData.short_name,
			display: formData.display,
			description: formData.description,
			scope: formData.scope,
			start_url: formData.start_url,
			theme_color: formData.theme_color,
			background_color: formData.background_color
		},
		null,
		2
	);

	function copyToClipboard() {
		navigator.clipboard.writeText(manifestPreview);
	}
</script>

<div class="container mx-auto p-5 md:p-10">
	<h1 class="text-2xl font-bold mb-4">Manifest Generator</h1>
	<div class="flex flex-col gap-4 mb-10 md:flex-row">
		<div class="w-full md:basis-2/3">
			<form on:submit|preventDefault={handleSubmit} class="grid grid-cols-1 sm:grid-cols-2 gap-4">
				<div>
					<label for="name" class="block mb-1 font-semibold"
						>Name <span class="text-red-500">*</span></label
					>
					<input
						id="name"
						type="text"
						bind:value={formData.name}
						placeholder="Name"
						class="input bg-white p-2 rounded border border-gray-300 dark:bg-gray-700 dark:text-white w-full"
						required
					/>
				</div>
				<div>
					<label for="shortName" class="block mb-1 font-semibold"
						>Short Name <span class="text-red-500">*</span></label
					>
					<input
						id="shortName"
						type="text"
						bind:value={formData.short_name}
						placeholder="Short Name"
						class="input bg-white p-2 rounded border border-gray-300 dark:bg-gray-700 dark:text-white w-full"
						required
					/>
				</div>
				<div>
					<label for="display" class="block mb-1 font-semibold">Display</label>
					<select
						id="display"
						bind:value={formData.display}
						class="select bg-white p-2 rounded border border-gray-300 dark:bg-gray-700 dark:text-white w-full"
					>
						{#each displayOptions as option}
							<option value={option}>{option}</option>
						{/each}
					</select>
				</div>
				<div>
					<label for="description" class="block mb-1 font-semibold">Description</label>
					<input
						id="description"
						type="text"
						bind:value={formData.description}
						placeholder="Description"
						class="input bg-white p-2 rounded border border-gray-300 dark:bg-gray-700 dark:text-white w-full"
					/>
				</div>
				<div>
					<label for="applicationScope" class="block mb-1 font-semibold">Application Scope</label>
					<input
						id="applicationScope"
						type="text"
						bind:value={formData.scope}
						placeholder="Application Scope"
						class="input bg-white p-2 rounded border border-gray-300 dark:bg-gray-700 dark:text-white w-full"
					/>
				</div>
				<div>
					<label for="startUrl" class="block mb-1 font-semibold"
						>Start URL <span class="text-red-500">*</span></label
					>
					<input
						id="startUrl"
						type="text"
						bind:value={formData.start_url}
						placeholder="Start URL"
						class="input bg-white p-2 rounded border border-gray-300 dark:bg-gray-700 dark:text-white w-full"
						required
					/>
				</div>
				<div>
					<label for="themeColor" class="block mb-1 font-semibold">Theme Color</label>
					<input
						id="themeColor"
						type="color"
						bind:value={formData.theme_color}
						class="color-picker p-1 h-10 rounded border border-gray-300 dark:bg-gray-700 dark:text-white w-full"
					/>
				</div>
				<div>
					<label for="backgroundColor" class="block mb-1 font-semibold"
						>Background Color <span class="text-red-500">*</span></label
					>
					<input
						id="backgroundColor"
						type="color"
						bind:value={formData.background_color}
						class="color-picker p-1 h-10 rounded border border-gray-300 dark:bg-gray-700 dark:text-white w-full"
						required
					/>
				</div>
				<div class="sm:col-span-2">
					<label for="file" class="block mb-1 font-semibold">Icon</label>
					<p class="text-sm text-gray-500 mb-1">
						Please upload a image with a minimum size of 512x512 pixels for the icon, and we'll
						generate the remaining sizes.
					</p>
					<input
						id="file"
						type="file"
						accept="image/png, image/jpeg"
						on:change={handleFileChange}
						class="file-input bg-white p-2 rounded border border-gray-300 dark:bg-gray-700 dark:text-white w-full"
					/>
					{#if errorImageMessage}
						<p class="text-red-500">{errorImageMessage}</p>
					{/if}
				</div>
				<button
					type="submit"
					class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded sm:col-span-2 disabled:opacity-50 disabled:pointer-events-none"
					disabled={isSubmitDisabled}
				>
					Generate Manifest
				</button>
			</form>
		</div>
		<div class="w-full md:basis-1/3">
			<div class="p-2">
				<h3 class="text-lg font-semibold mb-2">manifest.json preview</h3>
				<div class="relative">
					<pre class="overflow-auto max-h-96 bg-gray-100 dark:bg-gray-600 p-5"><code
							class="dark:text-white">{manifestPreview}</code
						></pre>
					<button
						on:click={copyToClipboard}
						class="absolute top-0 right-0 bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4"
						>Copy</button
					>
				</div>
			</div>
		</div>
	</div>
	<p class="mb-2">
		The Manifest File Generator is an online tool that facilitates the creation of a manifest file
		for your web application. The JSON manifest is a configuration file that defines basic
		information about your application such as name, icon, colors, and other properties. With our
		tool, you can quickly configure the manifest, customizing it to suit your application's
		requirements.
	</p>
	<p class="mb-2">
		Our tool is SEO-friendly, meaning the generated manifest.json file is optimized for search
		engines. This enhances the recognition of your application by search engines, potentially
		improving its visibility in search results.
	</p>
	<p class="mb-2">
		Moreover, our tool is user-friendly and requires no technical knowledge. Simply fill out the
		form, specify your application preferences, and then download the ready-made manifest.json file,
		ready to be added to your project.
	</p>
	<p class="mb-2">Start using our tool today and enhance the visibility of your web application!</p>
	<p class="mb-10">
		This project is Open Source. You can see source code here: <a
			href="https://github.com/kamilwyremski/manifest-generator"
			title="Manifest Generator Source Code">https://github.com/kamilwyremski/manifest-generator</a
		>
	</p>
	<p class="text-sm">
		Project 2024 - 2026 by <a href="http://wyremski.pl/en" title="Web Developer">Kamil Wyremski</a>
	</p>
</div>
