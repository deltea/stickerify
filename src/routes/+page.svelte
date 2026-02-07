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

  function findDimensions(width: number, height: number) {
    if (width > height) {
      return {
        width, height: height / (height / width),
        offsetX: 0, offsetY: (width - height) / 2
      };
    } else if (height > width) {
      return {
        width: width / (width / height), height,
        offsetX: (height - width) / 2, offsetY: 0
      };
    } else {
      return { width, height, offsetX: 0, offsetY: 0 };
    }
  }

  function onUpload(file: File) {
    const img = new Image();
    input = img;
    imgSrc = URL.createObjectURL(file);
    img.src = imgSrc;

    img.onload = () => {
      ctx = canvas.getContext("2d")!;

      const dims = findDimensions(img.naturalWidth, img.naturalHeight);
      canvas.width = dims.width;
      canvas.height = dims.height;

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(img, dims.offsetX, dims.offsetY);
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
