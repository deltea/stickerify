<script lang="ts">
    const validFileTypes = [
    "image/png",
    "image/jpeg",
    "image/jpg",
    "image/webp",
  ];

  let canvas: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;
  let fileInput: HTMLInputElement;
  let input = $state<HTMLImageElement | null>(null);
  let imgSrc = $state("");

  function onUpload(file: File) {
    const img = new Image();
    input = img;
    imgSrc = URL.createObjectURL(file);
    img.src = imgSrc;

    img.onload = () => {
      ctx = canvas.getContext("2d")!;
      canvas.width = img.naturalWidth;
      canvas.height = img.naturalHeight;

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(img, 0, 0);

      addOutline(ctx, canvas.width, canvas.height, 8, [0, 0, 0, 255]);
    }
  }

  function onDrop(e: DragEvent) {
    e.preventDefault();
    const files = e.dataTransfer?.files;
    if (files && files.length > 0) {
      if (validFileTypes.includes(files[0].type)) {
        onUpload(files[0]);
      } else {
        console.error("invalid file type");
      }
    } else {
      console.error("no file dropped");
    }
  }

  function fileInputChange(e: Event) {
    const target = e.target as HTMLInputElement;
    const files = target.files;
    if (files && files.length > 0) {
      if (validFileTypes.includes(files[0].type)) {
        onUpload(files[0]);
      } else {
        console.error("invalid file type");
      }
    }
  }

  function addOutline(
    ctx: CanvasRenderingContext2D,
    w: number,
    h: number,
    thickness: number,
    color: [number, number, number, number]
  ) {
    const src = ctx.getImageData(0, 0, w, h);
    const dst = ctx.createImageData(w, h);

    const srcData = src.data;
    const dstData = dst.data;

    for (let y = 0; y < h; y++) {
      for (let x = 0; x < w; x++) {
        const i = (y * w + x) * 4;
        const alpha = srcData[i + 3];
        if (alpha === 0) continue;

        for (let dy = -thickness; dy <= thickness; dy++) {
          for (let dx = -thickness; dx <= thickness; dx++) {
            const nx = x + dx;
            const ny = y + dy;
            if (nx < 0 || ny < 0 || nx >= w || ny >= h) continue;

            const dist = Math.sqrt(dx * dx + dy * dy);
            if (dist > thickness) continue;

            const ni = (ny * w + nx) * 4;
            if (dstData[ni + 3] !== 0) continue;

            dstData[ni + 0] = color[0];
            dstData[ni + 1] = color[1];
            dstData[ni + 2] = color[2];
            dstData[ni + 3] = color[3];
          }
        }
      }
    }

    for (let i = 0; i < srcData.length; i += 4) {
      if (srcData[i + 3] === 0) {
        srcData[i] = dstData[i];
        srcData[i + 1] = dstData[i + 1];
        srcData[i + 2] = dstData[i + 2];
        srcData[i + 3] = dstData[i + 3];
      }
    }

    ctx.putImageData(dst, 0, 0);
    ctx.putImageData(src, 0, 0);
  }

  function reset() {
    input = null;
    imgSrc = "";
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  }
</script>

<main
  ondrop={onDrop}
  ondragover={e => e.preventDefault()}
  class="h-screen p-8 flex justify-center items-center"
>
  <div class="flex flex-col gap-6 items-start">
    <div class="flex justify-between w-full">
      <h1 class="font-extrabold text-accent">stickerify</h1>

      {#if input}
        <button
          onclick={reset}
          class="flex gap-2 items-center text-text opacity-50 cursor-pointer hover:opacity-75 duration-100"
        >
          <iconify-icon icon="mingcute:repeat-fill" class="text-xl"></iconify-icon>
          <span class="hidden sm:block">restart</span>
        </button>
      {/if}
    </div>

    <div class="sm:w-xl w-[20rem] relative">
      <canvas
        bind:this={canvas}
        class="size-full aspect-square bg-bg shadow-lg outline-10 outline-surface rounded-2xl"
      ></canvas>

      {#if !input}
        <span class="opacity-50 absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2">drop + select an image</span>
      {/if}
    </div>

    <div class="flex gap-4 justify-between items-center w-full">

    </div>
  </div>
</main>

<input
  onchange={fileInputChange}
  bind:this={fileInput}
  accept={validFileTypes.join(",")}
  type="file"
  name="image"
  id="image"
  class="hidden"
  multiple={false}
/>
