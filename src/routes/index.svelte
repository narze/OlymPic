<script lang="ts">
  import { onMount } from 'svelte';
  import { tw } from 'twind';
  import Konva from 'konva';
  import { Facebook, Twitter } from 'svelte-share-buttons-component';
  import Kofi from '../lib/Kofi.svelte';
  import defaultBackground from '../../static/background.jpg';
  import defaultPerson from '../../static/person.png';

  const url = 'https://olym-pic.vercel.app';

  const sceneWidth = 1080;
  const sceneHeight = 1080;
  let stage;
  let previewImage: (event: Event, imageId: 'background' | 'person') => void;
  let previewText: (newText: string, textType: 'title' | 'quote' | 'nickname' | 'details') => void;
  let updateOpacity: (opacity: number) => void;
  let backgroundImage: Konva.Image;
  let titleTextTr: Konva.Transformer;
  let quoteTextTr: Konva.Transformer;
  let nameTextTr: Konva.Transformer;
  let detailsTextTr: Konva.Transformer;
  let personImageTr: Konva.Transformer;

  class ImageSwitcher {
    constructor(private image: Konva.Image, private layer: Konva.Layer) {}
    switchImage(url: string) {
      const img = new Image();
      img.onload = () => {
        this.image.image(img);
        this.layer.batchDraw();
      };
      personImageTr?.remove();
      personImageTr = addTransformer(this.layer, this.image, false);
      img.src = url;
    }
  }

  function addTransformer(layer, el, isText = true) {
    const MIN_WIDTH = 20;
    const tr = new Konva.Transformer({
      nodes: [el],
      padding: 5,
      rotateEnabled: false,
      enabledAnchors: [
        'top-left',
        'top-center',
        'top-right',
        // 'middle-right',
        // 'middle-left',
        'bottom-left',
        'bottom-center',
        'bottom-right',
      ],
      boundBoxFunc: (oldBox, newBox) => {
        if (newBox.width < MIN_WIDTH) {
          return oldBox;
        }
        return newBox;
      },
    });
    layer.add(tr);
    el.on('transform', () => {
      if (isText) {
        el.setAttrs({
          width: Math.max(el.width() * el.scaleX(), MIN_WIDTH),
          height: Math.max(el.height() * el.scaleY(), MIN_WIDTH),
          scaleX: 1,
          scaleY: 1,
          fontSize: el.fontSize() * el.scaleX(),
        });
      } else {
        el.setAttrs({
          width: Math.max(el.width() * el.scaleX(), MIN_WIDTH),
          height: Math.max(el.height() * el.scaleY(), MIN_WIDTH),
          scaleX: 1,
          scaleY: 1,
        });
      }
    });

    return tr;
  }

  onMount(async () => {
    stage = new Konva.Stage({
      container: 'canvasEditor',
      width: sceneWidth,
      height: sceneHeight,
    });

    const layer = new Konva.Layer({ fill: '#eeeeee' });
    stage.add(layer);

    // Background image
    backgroundImage = new Konva.Image({
      image: undefined as any,
      width: sceneWidth,
      height: sceneHeight,
    });
    layer.add(backgroundImage);
    const backgroundSwitcher = new ImageSwitcher(backgroundImage, layer);
    backgroundSwitcher.switchImage(defaultBackground);

    // Person image
    const personImage = new Konva.Image({
      image: undefined as any,
      draggable: true,
      x: 300,
      y: 230,
      scale: { x: 1.7, y: 1.7 },
    });
    layer.add(personImage);
    const personSwitcher = new ImageSwitcher(personImage, layer);
    personSwitcher.switchImage(defaultPerson);

    // Title
    const titleText = new Konva.Text({
      fontSize: 64,
      fontFamily: 'ThaiSansNeue',
      fontStyle: 'bold',
      fill: '#000',
      text: quote,
      stroke: '#fff',
      strokeWidth: 5,
      fillAfterStrokeEnabled: true,
      shadowColor: 'white',
      shadowBlur: 15,
      shadowOffset: { x: 0, y: 0 },
      shadowOpacity: 1,
      draggable: true,
      x: 100,
      y: 30,
    });
    layer.add(titleText);
    titleTextTr = addTransformer(layer, titleText);

    // Quote
    const quoteText = new Konva.Text({
      fontSize: 100,
      fontFamily: 'ThaiSansNeue',
      fontStyle: 'bold',
      fill: '#ffa',
      text: quote,
      stroke: '#000',
      strokeWidth: 8,
      fillAfterStrokeEnabled: true,
      shadowColor: 'white',
      shadowBlur: 15,
      shadowOffset: { x: 0, y: 0 },
      shadowOpacity: 1,
      draggable: true,
      x: 50,
      y: 200,
    });
    layer.add(quoteText);
    quoteTextTr = addTransformer(layer, quoteText);

    // Name
    const nameText = new Konva.Text({
      fontSize: 200,
      fontFamily: 'ThaiSansNeue',
      fontStyle: 'bold',
      fill: '#000',
      text: nickname,
      stroke: '#fff',
      strokeWidth: 8,
      fillAfterStrokeEnabled: true,
      shadowColor: 'white',
      shadowBlur: 15,
      shadowOffset: { x: 0, y: 0 },
      shadowOpacity: 1,
      draggable: true,
      x: 50,
      y: 500,
    });
    layer.add(nameText);
    nameTextTr = addTransformer(layer, nameText);

    // Details
    const detailsText = new Konva.Text({
      fontSize: 64,
      fontFamily: 'ThaiSansNeue',
      fontStyle: 'bold',
      fill: '#000',
      text: nickname,
      stroke: '#fff',
      strokeWidth: 8,
      fillAfterStrokeEnabled: true,
      shadowColor: 'white',
      shadowBlur: 15,
      shadowOffset: { x: 0, y: 0 },
      shadowOpacity: 1,
      draggable: true,
      x: 50,
      y: 720,
    });
    layer.add(detailsText);
    detailsTextTr = addTransformer(layer, detailsText);

    // Credit
    const creditsText = new Konva.Text({
      fontSize: 32,
      fontFamily: 'ThaiSansNeue',
      fontStyle: 'bold',
      fill: '#000',
      text: nickname,
      stroke: '#fff',
      strokeWidth: 8,
      fillAfterStrokeEnabled: true,
      shadowColor: 'white',
      shadowBlur: 15,
      shadowOffset: { x: 0, y: 0 },
      shadowOpacity: 1,
      draggable: true,
      x: 790,
      y: 1040,
    });
    layer.add(creditsText);
    creditsText.text('https://olym-pic.vercel.app');

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
    updateOpacity = (opacity) => {
      backgroundImage.opacity(opacity);
      layer.batchDraw();
    };
    eventResizeHandler();
  });

  let title: string = 'International Meme Olympiad (IMO2021)';
  let quote: string = 'หนทางหมื่นลี้\n  เริ่มพรุ่งนี้ก็แล้วกัน';
  let nickname: string = 'จอห์น';
  let details: string =
    'นายจอห์น ชาวไร่\nโรงเรียนมัธยมหมีใหญ่\nผู้แทนประเทศไทย\nวิชาลูุกเสือสามัญประจำบ้าน\nประจำปี 2564';
  let bgOpacity: number = 0.5;

  $: previewText?.(title, 'title');
  $: previewText?.(`"${quote}"`, 'quote');
  $: previewText?.(nickname, 'nickname');
  $: previewText?.(details, 'details');
  $: updateOpacity?.(bgOpacity);

  function downloadURI(uri, name) {
    const link = document.createElement('a');
    link.download = name;
    link.href = uri;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  }

  function download() {
    titleTextTr.hide();
    quoteTextTr.hide();
    nameTextTr.hide();
    detailsTextTr.hide();
    personImageTr.hide();

    const dataURL = stage.toDataURL({ pixelRatio: 3 });
    downloadURI(dataURL, 'OlymPic-download.jpg');

    setTimeout(() => {
      titleTextTr.show();
      quoteTextTr.show();
      nameTextTr.show();
      detailsTextTr.show();
      personImageTr.show();
    }, 300);
  }
</script>

<main class={tw`h-screen flex flex-col items-center mt-8`}>
  <h1 class={tw`text(6xl) my-2`}>OlymPic</h1>
  <p>Generate รูปคำคมแบบเด็กโอลิมปิควิชาการ</p>
  <div class={tw`w-full flex flex-row w-[80vw] h-3/5 mt-4`}>
    <div id="canvasParent" class={tw`w-1/2 text(center) flex items-start`}>
      <div id="canvasEditor" class={tw`w-full h-full`} />
    </div>
    <div class={tw`w-1/2 pl-4 flex-col items-start`}>
      <div class={tw`mb-2`}>
        <label class="block text-lg text-gray-700" for="background">รูปพื้นหลัง</label>
        <div class="mt-1 flex">
          <input
            type="file"
            id="background"
            accept="image/*"
            on:change={(event) => previewImage(event, 'background')}
            class={tw`text-base`}
          />
        </div>
      </div>
      <div class={tw`my-2`}>
        พื้นหลังโปร่งใส <input type="range" min="0" max="1" bind:value={bgOpacity} step="0.01" />
      </div>
      <div class={tw`my-2`}>
        <label class="block text-lg text-gray-700" for="person">รูปคน</label>
        <div class="mt-1 flex">
          <input
            type="file"
            id="person"
            accept="image/*"
            on:change={(event) => previewImage(event, 'person')}
            class={tw`text-base`}
          />
        </div>
      </div>
      <div class={tw`my-2`}>
        รายการแข่ง <input
          type="text"
          bind:value={title}
          class={tw`mt-1 p-1 focus:ring-indigo-500 focus:border-indigo-500 block w-full shadow-sm border border-gray-300 rounded-md`}
        />
      </div>
      <div class={tw`my-2`}>
        คำคม <textarea
          bind:value={quote}
          class={tw`mt-1 p-1 focus:ring-indigo-500 focus:border-indigo-500 block w-full shadow-sm border border-gray-300 rounded-md`}
          rows="2"
        />
      </div>
      <div class={tw`my-2`}>
        ชื่อเล่น <input
          type="text"
          bind:value={nickname}
          class={tw`mt-1 p-1 focus:ring-indigo-500 focus:border-indigo-500 block w-full shadow-sm border border-gray-300 rounded-md`}
        />
      </div>
      <div class={tw`my-2`}>
        ข้อมูล
        <textarea
          bind:value={details}
          class={tw`mt-1 p-1 focus:ring-indigo-500 focus:border-indigo-500 block w-full shadow-sm border border-gray-300 rounded-md`}
          rows="5"
        />
      </div>

      <div class={tw`mt-4`}>
        <button
          class={tw`w-full py-4 px-8 rounded bg-gray-300 text-2xl font-bold`}
          on:click={() => download()}>เซฟรูป</button
        >
      </div>
    </div>
  </div>

  <div class={tw`fixed bottom-4 right-4 z-20`}>
    <Facebook class={tw`h-10 w-10`} {url} />
    <Twitter class={tw`h-10 w-10`} text="" {url} />
  </div>
</main>

<Kofi name="narze" />

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
    font-size: 1.2rem;
  }
</style>
