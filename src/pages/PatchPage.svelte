<script lang="ts">
  import type {Patch} from '../lib/nip34.ts'

  import {repoCache} from '../lib/nostr.ts'
  import Header from '../components/Header.svelte'
  import UserLabel from '../components/UserLabel.svelte'

  export let patch: Patch

  $: repoName = repoCache.get(patch.repo!.guid)?.name || patch.repo!.id
  $: repoOwner = patch.repo!.owner
  $: repoLink = '/' + patch.repo!.naddr
</script>

<header class="pb-3 bg-white">
  <Header />
</header>
<main class="p-8 max-w-8xl">
  <div class="flex justify-start py-2 px-8 rounded">
    <UserLabel withLink pubkey={repoOwner} /> &nbsp; / &nbsp;
    <a class="hover:underline cursor-pointer text-sky-700" href={repoLink}
      >{repoName}</a
    >
  </div>
  <section class="font-mono bg-zinc-50 border px-2 py-1">
    <div>{patch.patch.hash}</div>
    <div>{patch.patch.message}</div>
    <div>{patch.patch.authorName} &lt;{patch.patch.authorEmail}&gt;</div>
    <div>{patch.patch.date}</div>
  </section>
  <div class="grid grid-flow-col auto-cols-auto mt-4">
    <section>
      {#each patch.patch.files as file, i (file.afterName)}
        <div class="font-mono">
          {#if file.added}
            <span class="text-blue-500" title="added"> + </span>
          {:else if file.deleted}
            <span class="text-red-500" title="removed"> - </span>
          {:else if file.beforeName !== file.afterName}
            <span class="text-indigo-500" title="renamed"> ⤭ </span>
            <span class="mr-2 line-through">{file.beforeName}</span>
          {:else}
            <span class="text-amber-500" title="edited"> ⭞ </span>
          {/if}
          <a href={`#f${i}`}>{file.afterName}</a>
        </div>
      {/each}
    </section>
    <section class="ml-4">
      {#each patch.patch.files as file, i (i)}
        <div id={`f${i}`} class="font-mono mb-4">
          <div class="text-lg block py-1 px-2 border bg-zinc-50">
            {file.afterName}
          </div>
          <div class="border border-t-0">
            {#each file.modifiedLines as line, i (i)}
              <div
                id={`l${line.added ? 'a' : 'r'}${line.lineNumber}`}
                class="grid px-2"
                class:bg-green-100={line.added}
                class:bg-red-100={!line.added}
                style:grid-template-columns="3rem 3rem 2rem 1fr"
              >
                <a
                  class="justify-self-end"
                  href={`#l${line.added ? 'a' : 'r'}${line.lineNumber}`}
                >
                  {#if !line.added}{line.lineNumber}{/if}
                </a>
                <a
                  class="justify-self-end"
                  href={`#l${line.added ? 'a' : 'r'}${line.lineNumber}`}
                >
                  {#if line.added}{line.lineNumber}{/if}
                </a>
                <a
                  class="justify-self-center"
                  href={`#l${line.added ? 'a' : 'r'}${line.lineNumber}`}
                >
                  {#if line.added}+{:else}-{/if}
                </a>
                <div class="whitespace-pre-wrap break-all">{line.line}</div>
              </div>
            {/each}
          </div>
        </div>
      {/each}
    </section>
  </div>
</main>
