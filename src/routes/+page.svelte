<script lang="ts">
	import JSZip from 'jszip';

	const formData = {
		name: '',
		short_name: '',
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
		!formData.name ||
		!formData.short_name ||
		!formData.start_url ||
		!formData.background_color ||
		!formData.icons;

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

		for (const size of iconSizes) {
			if (formData.icons) {
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
				if (image.width !== 512 || image.height !== 512) {
					errorImageMessage = 'The image must have a size of 512x512 pixels.';
					return;
				}
				formData.icons = file;
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
</script>

<div class="container mx-auto mt-10 p-10">
	<h2 class="text-2xl font-bold mb-4">Manifest Generator</h2>
	<div class="flex gap-4">
		<div class="basis-2/3">
			<form on:submit|preventDefault={handleSubmit} class="grid grid-cols-2 gap-4">
				<div>
					<label for="name" class="block mb-1 font-semibold"
						>Name <span class="text-red-500">*</span></label
					>
					<input
						id="name"
						type="text"
						bind:value={formData.name}
						placeholder="Name"
						class="input bg-white p-2 rounded border border-gray-300 w-full"
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
						class="input bg-white p-2 rounded border border-gray-300 w-full"
						required
					/>
				</div>
				<div>
					<label for="display" class="block mb-1 font-semibold">Display</label>
					<select
						id="display"
						bind:value={formData.display}
						class="select bg-white p-2 rounded border border-gray-300 w-full"
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
						class="input bg-white p-2 rounded border border-gray-300 w-full"
					/>
				</div>
				<div>
					<label for="applicationScope" class="block mb-1 font-semibold">Application Scope</label>
					<input
						id="applicationScope"
						type="text"
						bind:value={formData.scope}
						placeholder="Application Scope"
						class="input bg-white p-2 rounded border border-gray-300 w-full"
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
						class="input bg-white p-2 rounded border border-gray-300 w-full"
						required
					/>
				</div>
				<div>
					<label for="themeColor" class="block mb-1 font-semibold">Theme Color</label>
					<input
						id="themeColor"
						type="color"
						bind:value={formData.theme_color}
						class="color-picker p-1 h-10 rounded border border-gray-300 w-full"
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
						class="color-picker p-1 h-10 rounded border border-gray-300 w-full"
						required
					/>
				</div>
				<div class="col-span-2">
					<label for="file" class="block mb-1 font-semibold"
						>Icon <span class="text-red-500">*</span></label
					>
					<p class="text-sm text-gray-500 mb-1">
						Please upload a 512x512 image for the icon and we'll generate the remaining sizes
					</p>
					<input
						id="file"
						type="file"
						accept="image/png, image/jpeg"
						on:change={handleFileChange}
						class="file-input bg-white p-2 rounded border border-gray-300 w-full"
						required
					/>
					{#if errorImageMessage}
						<p class="text-red-500">{errorImageMessage}</p>
					{/if}
				</div>
				<button
					type="submit"
					class="btn bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded col-span-2 disabled:opacity-50 disabled:pointer-events-none"
					disabled={isSubmitDisabled}
				>
					Generate Manifest
				</button>
			</form>
		</div>
		<div class="basis-1/3">
			<div class="p-4">
				<h3 class="text-lg font-semibold mb-2">manifest.json preview</h3>
				<pre class="overflow-auto max-h-96 bg-gray-100 p-5"><code>{manifestPreview}</code></pre>
			</div>
		</div>
	</div>
</div>
