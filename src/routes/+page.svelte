<script lang="ts">
	import Konva from 'konva';
	import { tick } from 'svelte';

	let collapse = $state(false);
	let inputFileRef: HTMLInputElement;
	let imageUrl = $state<string>();
	let canvasRef: HTMLDivElement;
	let stage: Konva.Stage;
	let image: Konva.Image;
	let imageLayer: Konva.Layer;
	let point: { x: number, y: number } = $state({ x: 0, y: 0 });

	function pickImgFile() {
		inputFileRef.click();
	}

	async function fileChange(e: Event) {
		// 获取第一个文件
		const file = (e.target as HTMLInputElement)?.files?.[0];
		if (file) {
			imageUrl = URL.createObjectURL(file);
			await tick();
			loadImage();
		}
	}

	function loadImage() {
		if (!stage) {
			stage = new Konva.Stage({
				container: canvasRef,
				width: canvasRef.offsetWidth,
				height: canvasRef.offsetHeight
			});
		}
		if (!imageLayer) {
			imageLayer = new Konva.Layer({
				draggable: true
			});
		}
		const img = new Image();
		img.onload = () => {
			image = new Konva.Image({
				image: img
			});
			imageLayer.add(image);
		};
		img.src = imageUrl;
		stage.add(imageLayer);
	}

	function createPoint() {
		const dot = new Konva.Circle({
			x: Number(point.x ?? 0),
			y: Number(point.y ?? 0),
			radius: 2,
			fill: 'red',
			id: 'point'
		});
		imageLayer.add(dot);
	}

	function resetPoint() {
		imageLayer.findOne('#point')?.destroy?.();
	}

</script>

<main class="w-full h-screen bg-background flex text-foreground">
	<aside class="h-full">
		<div class={['h-full flex flex-col transition-all box-border', collapse ? 'w-0 ' :'w-[200px] gap-3 p-2']}>
			<div class="text-lg h-20 grid place-content-center text-center font-bold leading-tight">
				Picture Point
			</div>

			<div class="p-2 flex flex-col transition-all box-border gap-2">
				<div class="text-sm pl-2">X</div>
				<input class="border-border bg-transparent rounded-lg w-full" bind:value={point.x}>
			</div>

			<div class="p-2 flex flex-col transition-all box-border gap-2">
				<div class="text-sm pl-2">Y</div>
				<input class="border-border bg-transparent rounded-lg w-full" bind:value={point.y}>
			</div>

			<button class="py-2 border border-border border-solid rounded-lg hover:bg-card text-center text-sm"
							onclick={createPoint}>Find Point
			</button>
			<button class="py-2 border border-border border-solid rounded-lg hover:bg-card text-center text-sm"
							onclick={resetPoint}>Reset
			</button>
		</div>
	</aside>

	<div class={['flex-1 w-full transition-all box-border', !collapse ? 'py-2' : 'py-0']}>
		<div
			class="border-solid bg-card rounded-lg h-full w-full border border-border overflow-hidden"
		>
			<button class={["w-full h-full place-content-center", !imageUrl ? 'gird':'hidden' ]} onclick={pickImgFile}>
				点击或拖拽上传文件
			</button>
			<div bind:this={canvasRef} class={["w-full h-full place-content-center", imageUrl ? 'gird':'hidden' ]}></div>
		</div>

		<input type="file" class="hidden" bind:this={inputFileRef} onchange={fileChange} />
	</div>
</main>
