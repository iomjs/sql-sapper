<style>

</style>

<svelte:head>
  <title>Sql Prettifier</title>
</svelte:head>

<h2 class="text-center">Sql Prettifier</h2>
<br />

{#if process.browser}
  <SqlPrettifier />
{/if}

<script>
  import SqlPrettifier from '~/components/SqlPrettifier.svelte';
</script>
