<script lang="ts">
	const formData = {
		name: '',
		short_name: '',
		display: '',
		description: '',
		scope: '/',
		start_url: '/',
		theme_color: '#ffffff',
		background_color: '#ffffff',
		icons: null as File | null
	};

	const displayOptions = ['fullscreen', 'standalone', 'minimal-ui', 'browser'];

	function handleSubmit() {
		if (
			!formData.name ||
			!formData.short_name ||
			!formData.start_url ||
			!formData.background_color
		) {
			alert('Please fill in all required fields.');
			return;
		}

		if (formData.icons && formData.icons.size > 0) {
			// Sprawdzamy, czy ikona jest kwadratem 512x512 px, można to zrobić po stronie klienta lub serwera.
			// Tutaj skupimy się na przekazaniu pliku bez walidacji wymiarów.
		}

		const manifest = {
			name: formData.name,
			short_name: formData.short_name,
			display: formData.display || 'standalone',
			description: formData.description,
			scope: formData.scope,
			start_url: formData.start_url,
			theme_color: formData.theme_color,
			background_color: formData.background_color,
			icons: formData.icons
				? [
						{
							src: 'path/to/your/icon.png',
							sizes: '512x512',
							type: 'image/png'
						}
					]
				: []
		};

		const blob = new Blob([JSON.stringify(manifest, null, 2)], { type: 'application/json' });
		const url = URL.createObjectURL(blob);
		const a = document.createElement('a');
		a.href = url;
		a.download = 'manifest.json';
		document.body.appendChild(a);
		a.click();
		document.body.removeChild(a);
	}

	function handleFileChange(event: Event) {
		const input = event.target as HTMLInputElement;
		if (input.files && input.files.length > 0) {
			formData.icons = input.files[0];
		}
	}

	$: manifestPreview = JSON.stringify(
		{
			name: formData.name,
			short_name: formData.short_name,
			display: formData.display || 'standalone',
			description: formData.description,
			scope: formData.scope,
			start_url: formData.start_url,
			theme_color: formData.theme_color,
			background_color: formData.background_color,
			icons: [
				{
					src: 'path/to/your/icon.png',
					sizes: '512x512',
					type: 'image/png'
				}
			]
		},
		null,
		2
	);
</script>

<div class="container mx-auto mt-10">
	<h2 class="text-2xl font-bold mb-4">Manifest Generator</h2>
	<div class="flex gap-4">
		<div class="flex-1">
			<form on:submit|preventDefault={handleSubmit} class="grid grid-cols-2 gap-4">
				<input
					type="text"
					bind:value={formData.name}
					placeholder="Name"
					class="input bg-white p-2 rounded border border-gray-300"
					required
				/>
				<input
					type="text"
					bind:value={formData.short_name}
					placeholder="Short Name"
					class="input bg-white p-2 rounded border border-gray-300"
					required
				/>
				<select
					bind:value={formData.display}
					class="select bg-white p-2 rounded border border-gray-300 col-span-2"
				>
					<option value="" disabled selected>Display</option>
					{#each displayOptions as option}
						<option value={option}>{option}</option>
					{/each}
				</select>
				<input
					type="text"
					bind:value={formData.description}
					placeholder="Description"
					class="input bg-white p-2 rounded border border-gray-300 col-span-2"
				/>
				<input
					type="text"
					bind:value={formData.scope}
					placeholder="Application Scope"
					class="input bg-white p-2 rounded border border-gray-300"
				/>
				<input
					type="text"
					bind:value={formData.start_url}
					placeholder="Start URL"
					class="input bg-white p-2 rounded border border-gray-300"
					required
				/>
				<input
					type="color"
					bind:value={formData.theme_color}
					class="color-picker p-2 rounded border border-gray-300"
				/>
				<input
					type="color"
					bind:value={formData.background_color}
					class="color-picker p-2 rounded border border-gray-300"
					required
				/>
				<input
					type="file"
					accept="image/png"
					on:change={handleFileChange}
					class="file-input bg-white p-2 rounded border border-gray-300 col-span-2"
				/>
				<button
					type="submit"
					class="btn bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded col-span-2"
				>
					Generate Manifest
				</button>
			</form>
		</div>
		<div class="flex-1 bg-gray-100 p-4 rounded">
			<h3 class="text-lg font-semibold mb-2">Manifest Preview</h3>
			<pre class="overflow-auto max-h-96"><code>{manifestPreview}</code></pre>
		</div>
	</div>
</div>
