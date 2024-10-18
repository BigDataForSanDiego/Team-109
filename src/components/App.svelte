<script>
  import Gemini from './Gemini.svelte';

  let data = {};
  let error = null;
  let loading = true;
</script>

<main>
  <Gemini {data} {error} {loading}/>
</main>

<style>
</style>
