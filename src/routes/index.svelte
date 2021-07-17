<script lang="ts">
  import { onMount } from 'svelte';
  import { tw } from 'twind';
  import Konva from 'konva';
  import { Facebook, Twitter } from 'svelte-share-buttons-component';
  import Kofi from '../lib/Kofi.svelte';
  import greenBackground from '../../static/green.jpg';
  import blueBackground from '../../static/blue.jpg';
  import orangeBackground from '../../static/orange.jpg';
  import grayBackground from '../../static/gray.jpg';
  import defaultPerson from '../../static/person.png';

  const url = 'https://olym-pic.vercel.app';

  const sceneWidth = 1080;
  const sceneHeight = 1080;
  let stage: Konva.Stage;
  let exportStage: Konva.Stage;
  let layer: Konva.Layer;
  let usePresetBackgroundImage: (image: string) => void;
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

    layer = new Konva.Layer({ fill: '#eeeeee' });
    stage.add(layer);

    // Static Background
    const background = new Konva.Rect({
      x: 0,
      y: 0,
      width: stage.width(),
      height: stage.height(),
      fill: 'white',
      zOffset: -1,
      // remove background from hit graph for better perf
      // because we don't need any events on the background
      listening: false,
    });
    layer.add(background);

    // Background image
    backgroundImage = new Konva.Image({
      image: undefined as any,
      width: sceneWidth,
      height: sceneHeight,
    });
    layer.add(backgroundImage);
    const backgroundSwitcher = new ImageSwitcher(backgroundImage, layer);
    backgroundSwitcher.switchImage(greenBackground);

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
    usePresetBackgroundImage = (image: string) => {
      backgroundSwitcher.switchImage(image);
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

    exportStage = new Konva.Stage({
      container: 'export',
      width: sceneWidth,
      height: sceneHeight,
      scale: { x: 1, y: 1 },
    });
  });

  let title: string = 'International Meme Olympiad (IMO2021)';
  let quote: string = 'หนทางหมื่นลี้\n  เริ่มพรุ่งนี้ก็แล้วกัน';
  let nickname: string = 'จอห์น';
  let details: string =
    'นายจอห์น ชาวไร่\nโรงเรียนมัธยมหมีใหญ่\nผู้แทนประเทศไทย\nวิชาลูุกเสือสามัญประจำบ้าน\nประจำปี 2564';
  let bgOpacity: number = 1.0;

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
    const exportLayer = layer.clone({ listening: false });
    exportStage.add(exportLayer);

    const dataURL = exportStage.toDataURL({
      mimeType: 'image/jpeg',
    });
    downloadURI(dataURL, 'OlymPic-download');
    exportStage.removeChildren();

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
  <div class={tw`w-full flex flex-col lg:flex-row w-[96vw] lg:w-[80vw] h-3/5 mt-4`}>
    <div id="canvasParent" class={tw`w-full lg:w-1/2 text(center) flex items-start mb-4`}>
      <div id="canvasEditor" class={tw`w-full h-full`} />
    </div>

    <div class={tw`w-full lg:w-1/2 pl-0 lg:pl-4 flex-col items-start`}>
      <div class={tw`my-2`}>
        ใช้พื้นหลังจาก<a
          href="https://www.facebook.com/permalink.php?story_fbid=4162146883854761&id=486688764733943"
          target="_blank"
          rel="noreferrer"
          class="text-bold underline">เพจ Olympic สสวท.</a
        >
        <button
          on:click={() => usePresetBackgroundImage(greenBackground)}
          class={tw`bg-green-200 border rounded px-4 py-1 ml-1`}>สีเขียว</button
        >
        <button
          on:click={() => usePresetBackgroundImage(blueBackground)}
          class={tw`bg-blue-200 border rounded px-4 py-1 ml-1`}>สีฟ้า</button
        >
        <button
          on:click={() => usePresetBackgroundImage(orangeBackground)}
          class={tw`bg-yellow-500 border rounded px-4 py-1 ml-1`}>สีส้ม</button
        >
        <button
          on:click={() => usePresetBackgroundImage(grayBackground)}
          class={tw`bg-gray-200 border rounded px-4 py-1 ml-1`}>สีเทา</button
        >
      </div>
      <div class={tw`mb-2`}>
        <label class="block text-lg text-gray-700" for="background"
          >หรือ อัปโหลดรูปพื้นหลังเอง
          <input
            type="file"
            id="background"
            accept="image/*"
            on:change={(event) => previewImage(event, 'background')}
            class={tw`text-base`}
          /></label
        >
      </div>
      <div class={tw`my-2`}>
        พื้นหลังโปร่งใส <input type="range" min="0" max="1" bind:value={bgOpacity} step="0.01" />
      </div>
      <div class={tw`my-2`}>
        <label class="block text-lg text-gray-700" for="person"
          >รูปคน <input
            type="file"
            id="person"
            accept="image/*"
            on:change={(event) => previewImage(event, 'person')}
            class={tw`text-base`}
          /></label
        >
        <div class={tw`text-gray-500`}>
          หากต้องการลบพื้นหลัง ให้ใช้เว็บ <a
            href="https://remove.bg"
            target="_blank"
            rel="noreferrer"
            class={tw`font-bold underline text-black`}>remove.bg</a
          >
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

      <div class={tw`mt-4 mb-20 lg:mb-0`}>
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
    <a
      href="https://github.com/narze/OlymPic"
      target="_blank"
      rel="noreferrer"
      class={tw`inline-block relative top-[5px] bg-white`}
    >
      <svg
        width="2.5rem"
        height="2.5rem"
        viewBox="0 0 256 250"
        version="1.1"
        xmlns="http://www.w3.org/2000/svg"
        xmlns:xlink="http://www.w3.org/1999/xlink"
        preserveAspectRatio="xMidYMid"
      >
        <g>
          <path
            d="M128.00106,0 C57.3172926,0 0,57.3066942 0,128.00106 C0,184.555281 36.6761997,232.535542 87.534937,249.460899 C93.9320223,250.645779 96.280588,246.684165 96.280588,243.303333 C96.280588,240.251045 96.1618878,230.167899 96.106777,219.472176 C60.4967585,227.215235 52.9826207,204.369712 52.9826207,204.369712 C47.1599584,189.574598 38.770408,185.640538 38.770408,185.640538 C27.1568785,177.696113 39.6458206,177.859325 39.6458206,177.859325 C52.4993419,178.762293 59.267365,191.04987 59.267365,191.04987 C70.6837675,210.618423 89.2115753,204.961093 96.5158685,201.690482 C97.6647155,193.417512 100.981959,187.77078 104.642583,184.574357 C76.211799,181.33766 46.324819,170.362144 46.324819,121.315702 C46.324819,107.340889 51.3250588,95.9223682 59.5132437,86.9583937 C58.1842268,83.7344152 53.8029229,70.715562 60.7532354,53.0843636 C60.7532354,53.0843636 71.5019501,49.6441813 95.9626412,66.2049595 C106.172967,63.368876 117.123047,61.9465949 128.00106,61.8978432 C138.879073,61.9465949 149.837632,63.368876 160.067033,66.2049595 C184.49805,49.6441813 195.231926,53.0843636 195.231926,53.0843636 C202.199197,70.715562 197.815773,83.7344152 196.486756,86.9583937 C204.694018,95.9223682 209.660343,107.340889 209.660343,121.315702 C209.660343,170.478725 179.716133,181.303747 151.213281,184.472614 C155.80443,188.444828 159.895342,196.234518 159.895342,208.176593 C159.895342,225.303317 159.746968,239.087361 159.746968,243.303333 C159.746968,246.709601 162.05102,250.70089 168.53925,249.443941 C219.370432,232.499507 256,184.536204 256,128.00106 C256,57.3066942 198.691187,0 128.00106,0 Z M47.9405593,182.340212 C47.6586465,182.976105 46.6581745,183.166873 45.7467277,182.730227 C44.8183235,182.312656 44.2968914,181.445722 44.5978808,180.80771 C44.8734344,180.152739 45.876026,179.97045 46.8023103,180.409216 C47.7328342,180.826786 48.2627451,181.702199 47.9405593,182.340212 Z M54.2367892,187.958254 C53.6263318,188.524199 52.4329723,188.261363 51.6232682,187.366874 C50.7860088,186.474504 50.6291553,185.281144 51.2480912,184.70672 C51.8776254,184.140775 53.0349512,184.405731 53.8743302,185.298101 C54.7115892,186.201069 54.8748019,187.38595 54.2367892,187.958254 Z M58.5562413,195.146347 C57.7719732,195.691096 56.4895886,195.180261 55.6968417,194.042013 C54.9125733,192.903764 54.9125733,191.538713 55.713799,190.991845 C56.5086651,190.444977 57.7719732,190.936735 58.5753181,192.066505 C59.3574669,193.22383 59.3574669,194.58888 58.5562413,195.146347 Z M65.8613592,203.471174 C65.1597571,204.244846 63.6654083,204.03712 62.5716717,202.981538 C61.4524999,201.94927 61.1409122,200.484596 61.8446341,199.710926 C62.5547146,198.935137 64.0575422,199.15346 65.1597571,200.200564 C66.2704506,201.230712 66.6095936,202.705984 65.8613592,203.471174 Z M75.3025151,206.281542 C74.9930474,207.284134 73.553809,207.739857 72.1039724,207.313809 C70.6562556,206.875043 69.7087748,205.700761 70.0012857,204.687571 C70.302275,203.678621 71.7478721,203.20382 73.2083069,203.659543 C74.6539041,204.09619 75.6035048,205.261994 75.3025151,206.281542 Z M86.046947,207.473627 C86.0829806,208.529209 84.8535871,209.404622 83.3316829,209.4237 C81.8013,209.457614 80.563428,208.603398 80.5464708,207.564772 C80.5464708,206.498591 81.7483088,205.631657 83.2786917,205.606221 C84.8005962,205.576546 86.046947,206.424403 86.046947,207.473627 Z M96.6021471,207.069023 C96.7844366,208.099171 95.7267341,209.156872 94.215428,209.438785 C92.7295577,209.710099 91.3539086,209.074206 91.1652603,208.052538 C90.9808515,206.996955 92.0576306,205.939253 93.5413813,205.66582 C95.054807,205.402984 96.4092596,206.021919 96.6021471,207.069023 Z"
            fill="#161614"
          />
        </g>
      </svg>
    </a>
  </div>
  <div id="export" class={tw`hidden`} />
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
