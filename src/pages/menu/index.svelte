<script>
  import { url } from '@roxi/routify/runtime'
  import { metatags } from '@roxi/routify'
  import { fly } from 'svelte/transition'
  import Title from '../../components/Title.svelte'
  import MealCard from '../../components/MealCard.svelte'
  import menu, { categories as c } from '../../content/pages/menu'
  import svitsConfig from '../../../svits.config'

  const categories = Object.keys(c).map(k => ({
    slug: k,
    name: c[k].title,
  }))

  import { getCollection } from '../../collections'

  const PAGESIZE = 4
  const meals = {}
  categories.map(c => {
    const coll = getCollection('menu', { field: 'date', order: 'desc', isDate: true })
      .filter(m => m.type === c.slug)
    meals[c.slug] = {
      total: coll.elements.length,
      elements: coll.paginate(PAGESIZE, 1).elements
    }
  })

  metatags.title = 'Our menu | ' + svitsConfig.name
</script>

<div class="content">
  <div in:fly={{y: -10, duration: 500, delay: 100}}>
    <Title first="Our menu" second="Taste our meals"/>
    <p
      class="mt-12 mb-6 text-lg text-center text-gray-500 sm:text-xl"
      >{menu.description}</p
    >
    {#each categories as { slug, name, image }, i}
      <div
        class="flex flex-col my-12 text-gray-700 category dark:text-gray-500"
        in:fly={{x: -20, duration: 500, delay: 100, delay: 300 + i * 100}}
      >
        <a
          href={$url('/menu/:category', { category: slug })}
          class="category-link"
          title="View more meals from {name}"
        >
          <span class="title">{name}</span>
          <div class="flex items-center no-underline more">
            <span class="text-sm sm:text-base">View more</span>
            <span class="text-xs i jam:chevron-right sm:text-sm"></span>
          </div>
        </a>
        <span class="w-full m-auto bg-green-500" style="height: 2px"></span>
        <div class="py-4 meals">
          {#if meals[slug].total}
            <div class="flex flex-wrap pt-4 -m-4">
              {#each meals[slug].elements as m, ii}
                <div
                  class="w-full p-4 lg:w-1/3 sm:w-1/2 xxl:w-1/4 {ii == PAGESIZE - 1 ? 'hidden xxl:block' : ''}"
                  in:fly={{y: -20, duration: 800, delay: (i + 1) * 300 + (300 + ii * 100)}}
                >
                  <MealCard m={m} />
                </div>
              {/each}
              {#if meals[slug].total > PAGESIZE}
                <a
                  title="View more meals from {name}"
                  href={$url('/menu/:category', { category: slug })}
                  class="flex items-center py-2 m-auto text-xl text-center text-gray-400 sm:text-2xl font-title"
                  ><span class="hover:underline">View more</span> <span class="i jam:chevron-down"></span></a
                >
              {/if}
            </div>
          {:else}
            <p class="py-2 m-auto text-xl text-center text-gray-400 sm:text-2xl font-title">Empty</p>
          {/if}
        </div>
      </div>
    {/each}
  </div>
</div>

<style>
  .category-link {
    display: flex;
    align-items: center;
    justify-content: space-between;
    @apply font-title;
    @apply text-2xl;
  }

  .category-link:hover .title {
    text-decoration: underline;
  }

  @screen sm {
    .category-link {
      @apply text-4xl;
    }

    .category .more {
      opacity: 0;
      @apply duration-200;
      transform: translateX(-10%);
    }

    .category:hover .more {
      opacity: 1;
      transform: translateX(0);
    }
  }
</style>
