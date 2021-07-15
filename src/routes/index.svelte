<script lang="typescript">
	import { onMount, tick } from 'svelte';
	import { tw } from 'twind';
	import Konva from 'konva';
	import defaultBackground from '../../static/background.jpg';

	const sceneWidth = 1080;
	const sceneHeight = 1080;
	let stage;
	let previewImage;

	onMount(async () => {
		stage = new Konva.Stage({
			container: 'canvasEditor',
			width: sceneWidth,
			height: sceneHeight
		});
		const layer = new Konva.Layer({ fill: '#444444' });
		stage.add(layer);

		const img = new Image();
		img.onload = () => {
			const bg = new Konva.Image({
				image: img,
				width: sceneWidth,
				height: sceneHeight
			});
			// bg.zIndex(100);
			layer.add(bg);
			layer.draw();
		};
		img.src = '/static/background.jpg';

		const addText = (newText: string) => {
			const rectCanvas = new Konva.Rect({});
			const textCanvas = new Konva.Text({});
			layer.add(rectCanvas);
			layer.add(textCanvas);
			layer.draw();
		};

		const addImage = (imgPath: string, imgId: string) => {
			const img = new Image();
			img.src = imgPath;
			img.onload = () => {
				const imgCanvas = new Konva.Image({
					id: imgId,
					image: img,
					draggable: true
				});
				layer.add(imgCanvas);
				layer.draw();
			};
		};

		// const personImage = new Image();
		// personImage.onload = () => {
		// 	const person = new Konva.Image({
		// 		image: personImage,
		// 		width: sceneWidth,
		// 		height: sceneHeight
		// 	});
		// 	person.zIndex(100);
		// 	layer.add(person);
		// };
		// personImage.src = '/static/background.jpg';

		const eventResizeHandler = () => {
			console.log('eventresizehandler');
			const container = document.getElementById('canvasParent');
			const c_width = container.offsetWidth;
			const scale = c_width / sceneWidth;
			stage!.width(sceneWidth * scale);
			stage!.height(sceneHeight * scale);
			stage.scale({ x: scale, y: scale });
		};
		window.addEventListener('resize', eventResizeHandler);

		previewImage = (e, imgId) => {
			// const preview: HTMLElement = document.getElementById(imgId);
			// personImage.src = URL.createObjectURL(e.target.files[0]);
			addImage(URL.createObjectURL(e.target.files[0]), imgId);
			// preview.onload = () => URL.revokeObjectURL(preview.src);
		};

		// await tick();
		// layer.draw();
		eventResizeHandler();

		// setTimeout(() => {
		// 	// eventResizeHandler();
		// 	layer.draw();
		// }, 1);
	});

	let title: string = 'International Farming Olympiad';
	let quote: string = 'อยากมีการงานที่มั่นคง จงไปขายหูฟัง';
	let nickname: string = 'ตู่';
	let details: string =
		'พลเอก ประหยัด จันทร์อะไร\nโรงเรียน xxxxxxxxxx\nผู้แทนประเทศไทย วิชาทหาร ประจำปี 2564';
</script>

<main class={tw`h-screen flex flex-col items-center justify-center`}>
	<h1 class={tw`text(3xl) my-2`}>OlymPic</h1>
	<p>Generate รูปคำคมแบบเด็กโอลิมปิควิชาการ</p>
	<div class={tw`w-full flex flex-row w-[80vw] h-3/5 mt-4`}>
		<div id="canvasParent" class={tw`w-1/2 text(center) flex items-start`}>
			<div id="canvasEditor" class={tw`w-full h-full`} />
			<!-- <p class={tw`absolute top-0 z-20`}>{title}</p>
			<p class={tw`absolute top-12 z-20`}>{quote}</p>
			<p class={tw`absolute top-24 z-20`}>{nickname}</p>
			<p class={tw`absolute top-36 z-20 whitespace-pre-line`}>{details}</p>
			<img class={tw`absolute top-0 z-10`} id="person" height="300" alt="" />
			<img class={tw`absolute top-0`} id="background" height="300" alt="" /> -->
		</div>
		<div class={tw`w-1/2 flex-col items-start`}>
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
				รายการแข่ง : <input
					type="text"
					bind:value={title}
					class={tw`mt-1 p-1 focus:ring-indigo-500 focus:border-indigo-500 block w-full shadow-sm sm:text-sm border-gray-300 rounded-md`}
				/>
			</div>
			<div class={tw`my-2`}>
				คำคม : <input
					type="text"
					bind:value={quote}
					class={tw`mt-1 p-1 focus:ring-indigo-500 focus:border-indigo-500 block w-full shadow-sm sm:text-sm border-gray-300 rounded-md`}
				/>
			</div>
			<div class={tw`my-2`}>
				ชื่อเล่น : <input
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
</main>

<style>
</style>
