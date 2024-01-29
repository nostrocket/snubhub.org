<script lang="ts">
  import * as nip19 from 'nostr-tools/nip19'
  import {
    parsePatch,
    parseRepo,
    type Patch,
    type Repo
  } from '../../lib/nip34.ts'

  import {afterNavigate} from '$app/navigation'
  import {page} from '$app/stores'
  import {showToast} from '$lib/utils.ts'
  import {
    getWriteRelays,
    pool,
    repoCache,
    patchCache,
    repositoryRelays
  } from '$lib/nostr.ts'

  import RepositoryPage from '../../pages/RepositoryPage.svelte'
  import PatchPage from '../../pages/PatchPage.svelte'

  let code: string
  let patch: Patch | undefined
  let repo: Repo | undefined
  let loading: boolean = true
  let cancel: (() => void) | undefined

  afterNavigate(async () => {
    if (code === $page.params.code) return

    loading = true
    cancel?.()
    patch = undefined
    repo = undefined

    code = $page.params.code
    let {data, type} = nip19.decode(code)
    switch (type) {
      case 'naddr': {
        let {relays, identifier, pubkey, kind} = data as nip19.AddressPointer
        if (kind !== 30617) {
          showToast({type: 'error', text: 'unsupported event kind ' + kind})
          return
        }
        repo = repoCache.get(pubkey + '/' + identifier)
        if (repo) return

        let sc = pool.subscribeMany(
          [
            ...(relays || []),
            ...(await getWriteRelays(pubkey)),
            ...repositoryRelays
          ],
          [
            {
              kinds: [30617],
              authors: [pubkey],
              '#d': [identifier]
            }
          ],
          {
            onevent(evt) {
              if (!repo || evt.created_at > repo.event.created_at) {
                repo = parseRepo(evt)
                repoCache.set(repo.guid, repo)
              }
            },
            oneose() {
              loading = false
            }
          }
        )
        cancel = () => sc.close()
        break
      }
      case 'nevent': {
        let {relays, id, author} = data as nip19.EventPointer
        patch = patchCache.get(id)
        if (patch) return

        let sc = pool.subscribeMany(
          [...(relays || []), ...(await getWriteRelays(author))],
          [
            {
              kinds: [1617],
              ids: [id]
            }
          ],
          {
            onevent(evt) {
              let parsed = parsePatch(evt)
              if (parsed) {
                patch = parsed
                patchCache.set(evt.id, patch)
              }
            },
            oneose() {
              loading = false
            }
          }
        )
        cancel = () => sc.close()
        break
      }
    }
  })
</script>

{#if patch}
  <PatchPage {patch} />
{:else if repo}
  <RepositoryPage {repo} />
{:else if loading}
  <div class="p-8">loading</div>
{:else}
  <div class="p-8">not found</div>
{/if}
