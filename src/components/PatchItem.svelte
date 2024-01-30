<script lang="ts">
  import type {Patch} from '../lib/nip34.ts'

  import {repoCache} from '../lib/nostr.ts'
  import UserLabel from './UserLabel.svelte'
  import HumanDate from './HumanDate.svelte'

  export let patch: Patch
  export let withRepo = false

  $: repo =
    !withRepo || !patch.repo
      ? undefined
      : {
          owner: patch.repo.owner,
          name: repoCache.get(patch.repo.guid)?.name || patch.repo.id,
          naddr: patch.repo.naddr
        }
</script>

<div class="mb-4">
  <div class="flex justify-between items-center text-sm">
    <UserLabel pubkey={patch.event.pubkey} />
    {#if repo}
      <div class="ml-2">
        to <UserLabel pubkey={repo.owner} />'s
        <a
          class="text-sky-700 hover:underline cursor-pointer"
          href={'/' + repo.naddr}
        >
          {repo.name}
        </a>
      </div>
    {/if}
    <div class="ml-2">
      <a
        class="hover:underline cursor-pointer text-zinc-700"
        href={'/' + patch.nevent}
      >
        <HumanDate ts={patch.event.created_at} />
      </a>
    </div>
  </div>
  <div class="text-zinc-700">
    {patch.preamble.match(/Subject: (.*)/)?.[1] || '<empty>'}
  </div>
</div>
