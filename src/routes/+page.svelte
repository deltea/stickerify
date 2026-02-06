<script lang="ts">
    const validFileTypes = [
    "image/png",
    "image/jpeg",
    "image/jpg",
    "image/webp",
  ];

  let canvas: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D | null;
  let fileInput: HTMLInputElement;
  let input = $state<HTMLImageElement | null>(null);
  let imgSrc = $state("");

  function onUpload(file: File) {
    const img = new Image();
    input = img;
    imgSrc = URL.createObjectURL(file);
    img.src = imgSrc;
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
      {#if input && imgSrc}
        <canvas
          class="size-full aspect-square bg-bg shadow-lg outline-10 outline-surface rounded-2xl"
        ></canvas>
      {:else}
        <button
          onclick={() => fileInput.click()}
          class="size-full aspect-square flex items-center justify-center cursor-pointer duration-100 bg-bg shadow-lg outline-10 outline-surface rounded-2xl"
        >
          <span class="opacity-50">drop + select an image</span>
        </button>
      {/if}

      <canvas
        bind:this={canvas}
        class="w-full pixelated absolute active:cursor-grabbing {input ? "cursor-grab" : "pointer-events-none hidden"}"
      ></canvas>
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
