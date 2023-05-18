<script lang='ts'>
    import { onMount } from 'svelte';
    import { copied_image } from '../stores/stores';

    let files: FileList;
    export let selected_file: File;
    let rows=8
    let cols=4
    let offsetx=0
    let offsety=0
    let buttons = [];
    let imgWidth = 0;
    let imgHeight = 0;
    let hi
    let test1
    let test2

    function createButtons() {
    for (let i = 0; i < rows; i++) {
      for (let j = 0; j < cols; j++) {
        let button = {
          top: `${((100 / rows) * i) + offsety}%`,
          left: `${((100 / cols) * j) + offsetx}%`,
          width: `${100 / cols}%`,
          height: `${100 / rows}%`,
          label: i * cols + j
        };
        buttons.push(button);
      }
    }
  }

  function updateButtons() {
    buttons=[]
    createButtons()
    buttons=buttons
  }

  function copy_image(imageNumber: number) {
    let img = new Image();
    img.src = URL.createObjectURL(selected_file)

    // Set the crossOrigin attribute if the image is from a different origin
    // This allows you to access the pixel data for cropping
    img.crossOrigin = "Anonymous";
    
    // Create a new canvas element for the cropped image
    const croppedCanvas = document.createElement('canvas');
      const croppedContext = croppedCanvas.getContext('2d');

      // Set the dimensions of the new canvas to match the crop size
      croppedCanvas.width = img.width / cols;
      croppedCanvas.height = img.height / rows;

      test1=(img.width / cols) * (imageNumber % cols)
      test2= Math.floor((imageNumber) / cols)

      const offsetxpx = img.width * (offsetx)/100
      const offsetxpy = img.height * (offsety)/100

      // Draw the cropped portion of the image onto the canvas
      croppedContext.drawImage(img, (img.width / cols) * (imageNumber % cols) + offsetxpx, img.height / rows * Math.floor((imageNumber) / cols) + offsetxpy, img.width / cols, img.height / rows, 0, 0, img.width / cols, img.height / rows);

      // Create a new image element for the cropped image
      const croppedImageElement = new Image();

      // Set the src of the cropped image element to the canvas data URL
      croppedImageElement.src = croppedCanvas.toDataURL();

      copied_image.set(croppedCanvas.toDataURL())

      hi = croppedCanvas.toDataURL()

    // Set the onload event handler to handle the image loading
    img.onload = () => {
      // Create a new canvas element for the cropped image
      const croppedCanvas = document.createElement('canvas');
      const croppedContext = croppedCanvas.getContext('2d');

      // Set the dimensions of the new canvas to match the crop size
      croppedCanvas.width = img.width / cols;
      croppedCanvas.height = img.height / rows;

      test1=(img.width / cols) * (imageNumber % cols)
      test2= Math.floor((imageNumber) / cols)

      const offsetxpx = img.width * (offsetx)/100
      const offsetxpy = img.height * (offsety)/100

      // Draw the cropped portion of the image onto the canvas
      croppedContext.drawImage(img, (img.width / cols) * (imageNumber % cols) + offsetxpx, img.height / rows * Math.floor((imageNumber) / cols) + offsetxpy, img.width / cols, img.height / rows, 0, 0, img.width / cols, img.height / rows);

      // Create a new image element for the cropped image
      const croppedImageElement = new Image();

      // Set the src of the cropped image element to the canvas data URL
      croppedImageElement.src = croppedCanvas.toDataURL();

      copied_image.set(croppedCanvas.toDataURL())

      hi = croppedCanvas.toDataURL()
    }
  }

  onMount(() => {
    createButtons();
  });
</script>

<main class="flex flex-col items-center justify-start w-fit h-screen">
  <input
    class="mb-2" 
    accept="image/png, image/jpeg"
    multiple
    bind:files
    type="file" 
  />
  <div class="border-2 flex flex-row">
    <div class="m-2 flex flex-col items-center justify-center">
      <label for="rows" class="text-center">Number of Rows</label>
      <input name="rows" type="number" class="text-center" bind:value={rows} on:change={updateButtons}/>
      <label for="offsety" class="text-center">Offset-Y</label>
      <input name="offsety" type="number" class="text-center" bind:value={offsety} on:change={updateButtons}/>
    </div>
    <div class="m-2 ml-0 flex flex-col items-center justify-center">
      <label for="cols" class="text-center">Number of Columns</label>
      <input name="cols" type="number" class="text-center" bind:value={cols} on:change={updateButtons}/>
      <label for="offsetx" class="text-center">Offset-X</label>
      <input name="offsetx" type="number" class="text-center" bind:value={offsetx} on:change={updateButtons}/>
    </div>
  </div>
  
  {#if files}
    <select class= "m-2 ml-0" bind:value={selected_file}>
    {#each Array.from(files) as file}
        <option value={file}>{file.name}</option>
    {/each}
    </select>
  {/if}
  <div>
    {#if selected_file}
      <div class="absolute flex relative">
        <img src={URL.createObjectURL(selected_file)} alt="Loading..." class="object-cover"/>
        <div style="margin-left: {offsetx}">
            {#each buttons as { top, left, width, height, label }}
              <button class="absolute border border-gray-500 bg-transparent focus:outline-none p-4 rounded text-none" style={`top: ${top}; left: ${left}; width: ${width}; height: ${height};`} on:click={e => copy_image(label)}></button>
            {/each}
        </div>
      </div>
    {/if}
  </div>
</main>
   