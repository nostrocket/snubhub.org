<script>
  import UserLabel from './UserLabel.svelte'
  import {account, signer} from '../lib/nostr.ts'
</script>

<div class="flex justify-between">
  <div>
    <a href="/" class="text-stone-600 text-lg hover:underline">patch34</a>
    <span class="text-xs"
      >: <a
        href="https://github.com/nostr-protocol/nips/blob/git/34.md"
        class="text-rose-500 hover:underline"
        target="_blank">NIP-34</a
      >-based patch browser.
    </span>
  </div>
  <div class="w-1/6">
    {#if $account}
      <div class="flex items-center">
        <UserLabel pubkey={$account.pubkey} />
        <button
          class="ml-2 hover:underline text-stone-300 cursor-pointer text-sm"
          on:click={() => signer.signOut()}>logout</button
        >
      </div>
    {:else}
      <button
        class="h-full w-full px-2 py-1 text-sm text-white rounded transition-colors bg-rose-500 hover:bg-rose-700"
        on:click={() => signer.getPublicKey()}
      >
        login with extension
      </button>
    {/if}
  </div>
</div>
