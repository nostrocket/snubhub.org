<script lang="ts">
  import type {Patch} from '../lib/nip34.ts'

  import {repoCache} from '../lib/nostr.ts'
  import Header from '../components/Header.svelte'
  import UserLabel from '../components/UserLabel.svelte'

  export let patch: Patch

  $: repoName = repoCache.get(patch.repo!.guid)?.name || patch.repo!.id
  $: repoOwner = patch.repo!.owner
  $: repoLink = '/' + patch.repo!.naddr

  console.log(patch)
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
  <section class="text-lg mt-4 px-2 py-1 flex items-center">
    <span class="mr-2">patch from</span>
    <UserLabel withLink withPicture pubkey={patch.event.pubkey} />
  </section>
  <section class="font-mono bg-zinc-50 border px-2 py-1">
    <div class="whitespace-pre-wrap">{patch.preamble}</div>
  </section>
  {#if patch.comment !== ''}
    <section class="px-2 py-1 border border-t-0">
      <div class="whitespace-pre-wrap">{patch.comment}</div>
    </section>
  {/if}
  <div class="grid grid-flow-col auto-cols-auto mt-4">
    <section>
      {#each patch.files as file, i (i)}
        <div class="font-mono">
          {#if file.new}
            <span class="text-blue-500" title="added"> + </span>
            <a href={`#f${i}`}>{file.to}</a>
          {:else if file.deleted}
            <span class="text-red-500" title="removed"> - </span>
            <a href={`#f${i}`}>{file.from}</a>
          {:else if file.from !== file.to}
            <span class="text-indigo-500" title="renamed"> ⤭ </span>
            <span class="mr-2 line-through">{file.from}</span>
            <a href={`#f${i}`}>{file.to}</a>
          {:else}
            <span class="text-amber-500" title="modified"> ⭞ </span>
            <a href={`#f${i}`}>{file.to}</a>
          {/if}
        </div>
      {/each}
    </section>
    <section class="ml-4">
      {#each patch.files as file, i (i)}
        <div id={`f${i}`} class="font-mono mb-4">
          <div class="text-lg block py-1 px-2 border bg-zinc-50">
            {file.to}
          </div>
          <div class="border border-t-0">
            {#each file.chunks as chunk, i (i)}
              <div>
                {#each chunk.changes as line, i (i)}
                  <div
                    id={`l${line.type[0]}${line.ln}`}
                    class="grid px-2"
                    class:bg-green-100={line.type === 'add'}
                    class:bg-red-100={line.type === 'del'}
                    style:grid-template-columns="3rem 3rem 2rem 1fr"
                  >
                    <a
                      class="justify-self-end"
                      href={`#l${line.type[0]}${
                        line.type === 'normal'
                          ? line.ln1
                          : line.type === 'del'
                          ? line.ln
                          : ''
                      }`}
                    >
                      {line.type === 'normal'
                        ? line.ln1
                        : line.type === 'del'
                        ? line.ln
                        : ''}
                    </a>
                    <a
                      class="justify-self-end"
                      href={`#l${line.type[0]}${
                        line.type === 'normal'
                          ? line.ln2
                          : line.type === 'add'
                          ? line.ln
                          : ''
                      }`}
                    >
                      {line.type === 'normal'
                        ? line.ln2
                        : line.type === 'add'
                        ? line.ln
                        : ''}
                    </a>
                    <span class="justify-self-center">
                      {#if line.type === 'add'}+{:else if line.type === 'del'}-{/if}
                    </span>
                    <div class="whitespace-pre-wrap break-all">
                      {line.content.substring(1)}
                    </div>
                  </div>
                {/each}
              </div>
              {#if i !== file.chunks.length - 1}
                <div class="bg-zinc-200 h-5">&nbsp;</div>
              {/if}
            {/each}
          </div>
        </div>
      {/each}
    </section>
  </div>
</main>
