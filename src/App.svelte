<script>
  import './sw/sw'
  import { isOffline, updateAvailable } from './sw/store'
  import { count, loaded } from './stores/snipcart'

  import { Router } from '@roxi/routify/runtime'
  import { routes } from '../.routify/routes'

  import OfflineBanner from './components/OfflineBanner.svelte'
  import BannerUpdate from './components/BannerUpdate.svelte'

  import svitsConfig from '../svits.config.json'

  import { preferences } from './stores/preferences'

  if (svitsConfig.routifyRuntimeConfig?.useHash) {
    ! window.location.hash && window.location.replace(`${window.location.origin}/#${window.location.pathname}`)
  }
  $: document.documentElement.classList.toggle('dark', $preferences.darkMode)

  function handleNetworkChange() {
    $isOffline = !navigator.onLine
  }

  import { onMount } from 'svelte'

  onMount(() => {
    window.addEventListener('online', handleNetworkChange)
    window.addEventListener('offline', handleNetworkChange)
    document.addEventListener('snipcart.ready', () => {
      window.Snipcart.events.on('snipcart.initialized', (snipcartState) => {
        $loaded = true
      })
      window.Snipcart.store.subscribe(() => {
        $count = window.Snipcart.store.getState().cart.items.count
      })
    })
  })

  $: if ($isOffline != undefined) {
    handleNetworkChange()
  }
</script>

<svelte:head>
  <link rel="stylesheet" href="https://cdn.snipcart.com/themes/v3.0.24/default/snipcart.css" />
</svelte:head>

<OfflineBanner/>
<BannerUpdate/>
<Router {routes} config={{ ...svitsConfig?.routifyRuntimeConfig }} />
