<script lang="ts">
  import dateFormat from 'dateformat'

  export let ts: number
  let now = Date.now() / 1000
  let d = new Date(ts * 1000)
</script>

{#if ts > now - 60 * 60 * 24}
  {dateFormat(d, 'mmmm dS, hh:MM')}
{:else if ts < now - 60 * 60 * 24 * 180}
  {dateFormat(d, 'mmmm dS, YYYY')}
{:else}
  {dateFormat(d, 'mmmm dS')}
{/if}
