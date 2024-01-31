<script lang="ts">
  import '../app.postcss'
  import Toast from '../components/Toast.svelte'
  // export const prerender = true;
</script>

<svelte:head>
  <title>patch34</title>
</svelte:head>

<slot />

<Toast />
