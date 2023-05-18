<script lang="ts">
    import Navbar from '../../components/navbar.svelte';
    import Converter from '../../components/converter.svelte';
    import Sheet from '../../components/sheet.svelte';

    let file: File = null
  </script>
  
  <main class="m-2 flex flex-row w-screen h-screen">
    <Navbar></Navbar>
    <Converter bind:selected_file={file}></Converter>
    <Sheet></Sheet>
  </main>