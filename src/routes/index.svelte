<script lang="typescript">
	import { onMount, tick } from 'svelte';
	import { tw } from 'twind';
	import Konva from 'konva';
	import { Facebook, Twitter } from 'svelte-share-buttons-component';

	const url = 'https://olym-pic.vercel.app';

	const sceneWidth = 1080;
	const sceneHeight = 1080;
	let stage;
	let previewImage: (event: Event, imageId: 'background' | 'person') => void;
	let previewText: (newText: string, textType: 'title' | 'quote' | 'nickname' | 'details') => void;

	class ImageSwitcher {
		constructor(private image: Konva.Image, private layer: Konva.Layer) {}
		switchImage(url: string) {
			const img = new Image();
			img.onload = () => {
				this.image.image(img);
				this.layer.batchDraw();
			};
			img.src = url;
		}
	}
	class TitleSwitcher {
		constructor(private text: string, private layer: Konva.Layer) {
			this.group = new Konva.Group();
			this.rect = new Konva.Rect({
				draggable: true,
				opacity: 0
			});
			this.text = new Konva.Text({
				text
			});
			this.group.add(rect);
			this.group.add(text);
			layer.add(group);
		}
	}

	onMount(async () => {
		stage = new Konva.Stage({
			container: 'canvasEditor',
			width: sceneWidth,
			height: sceneHeight
		});

		const layer = new Konva.Layer({ fill: '#eeeeee' });
		stage.add(layer);

		// Background image
		const backgroundImage = new Konva.Image({
			image: undefined as any,
			width: sceneWidth,
			height: sceneHeight
		});
		layer.add(backgroundImage);
		const backgroundSwitcher = new ImageSwitcher(backgroundImage, layer);
		backgroundSwitcher.switchImage('/static/background.jpg');

		// Person image
		const personImage = new Konva.Image({
			image: undefined as any,
			draggable: true
		});
		layer.add(personImage);
		const personSwitcher = new ImageSwitcher(personImage, layer);

		// Title
		const titleText = new Konva.Text({
			fontSize: 64,
			fontFamily: 'ThaiSansNeue',
			fontStyle: 'bold',
			fill: '#ffa',
			text: quote,
			stroke: '#000',
			strokeWidth: 3,
			shadowColor: 'white',
			shadowBlur: 20,
			shadowOffset: { x: 0, y: 0 },
			shadowOpacity: 1,
			draggable: true
		});
		layer.add(titleText);

		// Quote
		const quoteText = new Konva.Text({
			fontSize: 120,
			fontFamily: 'ThaiSansNeue',
			fontStyle: 'bold',
			fill: '#ffa',
			text: quote,
			stroke: '#000',
			strokeWidth: 3,
			shadowColor: 'white',
			shadowBlur: 20,
			shadowOffset: { x: 0, y: 0 },
			shadowOpacity: 1,
			draggable: true
		});
		layer.add(quoteText);

		// Name
		const nameText = new Konva.Text({
			fontSize: 200,
			fontFamily: 'ThaiSansNeue',
			fontStyle: 'bold',
			fill: '#ffa',
			text: nickname,
			stroke: '#000',
			strokeWidth: 3,
			shadowColor: 'white',
			shadowBlur: 20,
			shadowOffset: { x: 0, y: 0 },
			shadowOpacity: 1,
			draggable: true
		});
		layer.add(nameText);

		// Details
		const detailsText = new Konva.Text({
			fontSize: 64,
			fontFamily: 'ThaiSansNeue',
			fontStyle: 'bold',
			fill: '#ffa',
			text: nickname,
			stroke: '#000',
			strokeWidth: 3,
			shadowColor: 'white',
			shadowBlur: 20,
			shadowOffset: { x: 0, y: 0 },
			shadowOpacity: 1,
			draggable: true
		});
		layer.add(detailsText);

		const eventResizeHandler = () => {
			const container = document.getElementById('canvasParent');
			const c_width = container.offsetWidth;
			const scale = c_width / sceneWidth;
			stage!.width(sceneWidth * scale);
			stage!.height(sceneHeight * scale);
			stage.scale({ x: scale, y: scale });
		};
		window.addEventListener('resize', eventResizeHandler);

		previewImage = (e, imageId) => {
			const input = e.target as HTMLInputElement;
			const url = URL.createObjectURL(input.files[0]);
			const switcher = imageId === 'background' ? backgroundSwitcher : personSwitcher;
			switcher.switchImage(url);
		};
		previewText = (newText, textType) => {
			if (textType === 'title') {
				const textShape = titleText;
				textShape.text(newText);
				layer.batchDraw();
			}
			if (textType === 'quote') {
				const textShape = quoteText;
				textShape.text(newText);
				layer.batchDraw();
			}
			if (textType === 'nickname') {
				const textShape = nameText;
				textShape.text(newText);
				layer.batchDraw();
			}
			if (textType === 'details') {
				const textShape = detailsText;
				textShape.text(newText);
				layer.batchDraw();
			}
		};
		eventResizeHandler();
	});

	let title: string = 'International Farming Olympiad';
	let quote: string = 'อยากมีการงานที่มั่นคง จงไปขายหูฟัง';
	let nickname: string = 'ตู่';
	let details: string =
		'พลเอก ประหยัด จันทร์อะไร\nโรงเรียน xxxxxxxxxx\nผู้แทนประเทศไทย วิชาทหาร ประจำปี 2564';

	$: previewText?.(title, 'title');
	$: previewText?.(quote, 'quote');
	$: previewText?.(nickname, 'nickname');
	$: previewText?.(details, 'details');
</script>

<main class={tw`h-screen flex flex-col items-center justify-center`}>
	<h1 class={tw`text(3xl) my-2`}>OlymPic</h1>
	<p>Generate รูปคำคมแบบเด็กโอลิมปิควิชาการ</p>
	<div class={tw`w-full flex flex-row w-[80vw] h-3/5 mt-4`}>
		<div id="canvasParent" class={tw`w-1/2 text(center) flex items-start`}>
			<div id="canvasEditor" class={tw`w-full h-full`} />
		</div>
		<div class={tw`w-1/2 pl-4 flex-col items-start`}>
			<div class={tw`my-2`}>
				<label class="block text-sm font-medium text-gray-700" for="background">รูปพื้นหลัง</label>
				<div class="mt-1 flex">
					<input
						type="file"
						id="background"
						accept="image/*"
						on:change={(event) => previewImage(event, 'background')}
					/>
				</div>
			</div>
			<div class={tw`my-2`}>
				<label class="block text-sm font-medium text-gray-700" for="person">รูปคน</label>
				<div class="mt-1 flex">
					<input
						type="file"
						id="person"
						accept="image/*"
						on:change={(event) => previewImage(event, 'person')}
					/>
				</div>
			</div>
			<div class={tw`my-2`}>
				รายการแข่ง <input
					type="text"
					bind:value={title}
					class={tw`mt-1 p-1 focus:ring-indigo-500 focus:border-indigo-500 block w-full shadow-sm sm:text-sm border-gray-300 rounded-md`}
				/>
			</div>
			<div class={tw`my-2`}>
				คำคม <textarea
					bind:value={quote}
					class={tw`mt-1 p-1 focus:ring-indigo-500 focus:border-indigo-500 block w-full shadow-sm sm:text-sm border-gray-300 rounded-md`}
					rows="4"
				/>
			</div>
			<div class={tw`my-2`}>
				ชื่อเล่น <input
					type="text"
					bind:value={nickname}
					class={tw`mt-1 p-1 focus:ring-indigo-500 focus:border-indigo-500 block w-full shadow-sm sm:text-sm border-gray-300 rounded-md`}
				/>
			</div>
			<div class={tw`my-2`}>
				ข้อมูล
				<textarea
					bind:value={details}
					class={tw`mt-1 p-1 focus:ring-indigo-500 focus:border-indigo-500 block w-full shadow-sm sm:text-sm border-gray-300 rounded-md`}
					rows="4"
				/>
			</div>
		</div>
	</div>

	<div class={tw`fixed bottom-4 right-4 z-20 sm:hidden`}>
		<Facebook class={tw`h-10 w-10`} {url} />
		<Twitter class={tw`h-10 w-10`} text="" {url} />
	</div>
</main>

<style>
	@font-face {
		font-family: 'ThaiSansNeue';
		font-style: normal;
		font-weight: normal;
		src: url('../../static/fonts/ThaiSansNeue-Regular.otf') format('opentype');
	}

	@font-face {
		font-family: 'ThaiSansNeue';
		font-style: normal;
		font-weight: bold;
		src: url('../../static/fonts/ThaiSansNeue-Bold.otf') format('opentype');
	}

	* {
		font-family: 'ThaiSansNeue';
	}
</style>
