<script lang="ts">
    import pica from 'pica';
    import { onMount } from 'svelte';
    import { copied_image } from '../stores/stores';
  
    let overlayX = 0; // X-coordinate of the overlay
    let overlayY = 0; // Y-coordinate of the overlay

    let X = 0;
    let Y = 0;
    let selected_slot = 1

    let baseImageLoaded = false;
    let overlayImageLoaded = false;
    $: $copied_image, run();

    let baseImage;
    let overlayImage;
    let canvas;
    let buttons = [];

    onMount(() => {
        createButtons()
        baseImage = new Image();
        baseImage.onload = () => {
        baseImageLoaded = true;
        drawOverlay();
        buttons = buttons
    };

    overlayImage = new Image(settings.spriteSizeX * settings.spritePerAnim * settings.spritesheetPerRow, settings.spriteSizeY * settings.animPerSprite * settings.spritesheetPerSupersheet);
    overlayImage.onload = () => {
      overlayImageLoaded = true;
      drawOverlay();
    };

    canvas = document.getElementById('canvas');
  });

  function drawOverlay() {
    if (baseImageLoaded && overlayImageLoaded) {
      const context = canvas.getContext('2d');
      context.clearRect(0, 0, canvas.width, canvas.height);
      context.drawImage(baseImage, 0, 0, canvas.width, canvas.height);
      context.drawImage(overlayImage, overlayX, overlayY);
    }
  }

  function drawImg() {
    if (baseImageLoaded && overlayImageLoaded) {
      const context = canvas.getContext('2d');
      context.clearRect(X, Y, settings.spriteSizeX, settings.spriteSizeY);
      context.drawImage(X, Y, settings.spriteSizeX, settings.spriteSizeY);
      X = X + settings.spriteSizeX
      if(X > settings.spriteSizeX * settings.spritePerAnim * settings.spritesheetPerRow) {
        X = 0;
        Y = Y + settings.spriteSizeY
      }
    }
  }

  function saveImage() {
    const link = document.createElement('a');
    link.href = canvas.toDataURL();
    link.download = 'edited_image.png';
    link.click();
  }

  function createButtons() {
    buttons = []
    const cols = settings.spritePerAnim * settings.spritesheetPerRow
    const rows = settings.animPerSprite * settings.spritesheetPerSupersheet
    const canvas = document.getElementById('canvas') as HTMLCanvasElement;

    const x = canvas.width / cols
    const y = canvas.height / rows
    for (let i = 0; i < rows; i++) {
      for (let j = 0; j < cols; j++) {
        let button = {
          top: `${(settings.spriteSizeY * i)}px`,
          left: `${(settings.spriteSizeX * j)}px`,
          width: `${x}px`,
          height: `${y}px`,
          id: i * cols + j + 1,
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

  function button_click(button: number) {
    selected_slot = button
    overlayX = (button-1) % (settings.spritePerAnim * settings.spritesheetPerRow)
    overlayY = Math.floor((button-1) / (settings.spritePerAnim * settings.spritesheetPerRow))
  }

  function run() {
    let src = $copied_image
    const img = new Image()
    img.src = src
    const canvas = document.getElementById('canvas') as HTMLCanvasElement;
    if(canvas) {
      const context = canvas.getContext('2d');
      img.onload = () => {
        let imageWidth = img.width;
        let imageHeight = img.height;

        if (imageWidth > settings.spriteSizeX || imageHeight > settings.spriteSizeY) {
          // Calculate the aspect ratio of the image
          const aspectRatio = imageWidth / imageHeight;

          // Calculate the new dimensions while maintaining the aspect ratio
          if (imageWidth > imageHeight) {
            imageWidth = settings.spriteSizeX;
            imageHeight = Math.floor(imageWidth / aspectRatio);
          } else {
            imageHeight = settings.spriteSizeY;
            imageWidth = Math.floor(imageHeight * aspectRatio);
          }
        }

        const x = (overlayX * settings.spriteSizeX) + ((settings.spriteSizeX - imageWidth) / 2);
        const y = (overlayY * settings.spriteSizeY) + ((settings.spriteSizeY - imageHeight) / 2);

        context.clearRect(overlayX * settings.spriteSizeX, overlayY * settings.spriteSizeY, settings.spriteSizeX, settings.spriteSizeY)
        context.drawImage(img, x, y, imageWidth, imageHeight);
        overlayX = overlayX+1
        selected_slot = selected_slot + 1
        if(overlayX % settings.spritePerAnim == 0){
          overlayX = overlayX - settings.spritePerAnim
          overlayY = overlayY + 1
          if(overlayY % settings.animPerSprite == 0) {
            overlayX = overlayX + settings.spritePerAnim
            overlayY = overlayY - settings.animPerSprite
            if(overlayX == settings.spritePerAnim * settings.spritesheetPerRow) {
              overlayX = 0
              overlayY = overlayY + settings.animPerSprite
              if(overlayY == settings.animPerSprite * settings.spritesheetPerSupersheet) {
                overlayX = 0
                overlayY = 0
              }
            }
          }
          selected_slot = (overlayX + (settings.spritePerAnim * settings.spritesheetPerRow) * overlayY) + 1
        }
     }
    }
  }

  function change_overlay() { 
    selected_slot = (overlayX + (settings.spritePerAnim * settings.spritesheetPerRow) * overlayY) + 1
  }

  onMount(() => {
    createButtons();
    buttons=buttons
  });

let settings = {
    spriteSizeX: 48,
    spriteSizeY: 48,
    spritePerAnim: 3,
    animPerSprite: 4,
    spritesheetPerRow: 4,
    spritesheetPerSupersheet: 2,
}

</script>

<main class="top-0 right-0">

    <div>
      <div>
        <label for="overlayX">X-coordinate:</label>
        <input type="number" id="overlayX" bind:value="{overlayX}" on:change="{change_overlay}" />
      </div>
      
      <div>
        <label for="overlayY">Y-coordinate:</label>
        <input type="number" id="overlayY" bind:value="{overlayY}" on:change="{change_overlay}" />
      </div>
      <div class="relative" style="width:{settings.spriteSizeX * settings.spritePerAnim * settings.spritesheetPerRow}px; height:{settings.spriteSizeY * settings.animPerSprite * settings.spritesheetPerSupersheet}px;">
        <canvas class="border-2 absolute" id="canvas" width="{settings.spriteSizeX * settings.spritePerAnim * settings.spritesheetPerRow}" height="{settings.spriteSizeY * settings.animPerSprite * settings.spritesheetPerSupersheet}" style="z-index: 0;"></canvas>
        <div style="z-index: 1;">
          {#each buttons as { top, left, width, height, id}}
            {#if selected_slot != id}
              <button class="border border-gray-500 p-4 rounded bg-transparent text-none hover:border-gray-500 mr-0" style={`top: ${top}; left: ${left}; width: ${width}; height: ${height};`} on:click={e=>button_click(id)}></button>
            {:else}
              <button class="p-4 rounded border-white border-2 bg-transparent text-none hover:border-white mr-0" style={`top: ${top}; left: ${left}; width: ${width}; height: ${height};`} on:click={e=>button_click(id)}></button>
            {/if}


          {/each}
        </div>
      </div>
      <button on:click="{saveImage}">Save Image</button>
    </div>

    <div class="border-2">
        <div class="m-2 flex flex-row items-center space-x-2">
            <h1 class='text-2xl text-white font-bold my-4'>Settings</h1>
            <button>Save</button>
            <button>Upload</button>
        </div>
        <div class="flex flex-row m-2 space-x-2">
            <p>Sprite Size X</p>
            <input name="spriteSizeX" bind:value={settings.spriteSizeX} type="number" on:change={updateButtons}/>
        </div>
        <div class="flex flex-row m-2 space-x-2">
            <p>Sprite Size </p>
            <input name="spriteSizeY" bind:value={settings.spriteSizeY} type="number" on:change={updateButtons}/>
        </div>
        <div class="flex flex-row m-2 space-x-2">
            <p>Sprite Per Animation</p>
            <input name="spritePerAnim" bind:value={settings.spritePerAnim} type="number" on:change={updateButtons}/>
        </div>
        <div class="flex flex-row m-2 space-x-2">
            <p>Animation Per Sprite</p>
            <input name="animPerSprite" bind:value={settings.animPerSprite} type="number" on:change={updateButtons}/>
        </div>
        <div class="flex flex-row m-2 space-x-2">
            <p>Spritesheet Per Row</p>
            <input name="spritesheetPerRow" bind:value={settings.spritesheetPerRow} type="number" on:change={updateButtons}/>
        </div>
        <div class="flex flex-row m-2 space-x-2">
            <p>Spritesheet Per Supersheet</p>
            <input name="spritesheetPerSupersheet" bind:value={settings.spritesheetPerSupersheet} type="number" on:change={updateButtons}/>
        </div>
    </div>
</main>