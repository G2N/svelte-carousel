<script>
  import { swipe } from "svelte-gestures";
  export let count;
  export let itemsPerPage = 1;
  export let page = 0;
  export let circular = false;
  export let bullets = false;
  export let arrows = false;

  $: floaterSize = (count / itemsPerPage) * 100;
  $: pagesCount = Math.ceil(count / itemsPerPage);
  $: translateX = getTranslateX(page);
  $: isFirstPage = page - 1 <= -1;
  $: isLastPage = page + 1 >= pagesCount;

  function getTranslateX(page) {
    const pageOffset = ((page * itemsPerPage) / count) * 100;
    const missingItems =
      count % itemsPerPage === 0 ? 0 : itemsPerPage - (count % itemsPerPage);
    const missingOffset = (missingItems * 100) / count;

    let offset =
      page + 1 >= pagesCount &&
      missingItems > 0 &&
      missingItems !== itemsPerPage
        ? // we don't want to go too far on the last page
          pageOffset - missingOffset
        : pageOffset;
    return 0 - offset;
  }

  function next() {
    if (isLastPage) {
      if (circular) {
        page = 0;
      }
      return;
    }
    page++;
  }

  function previous() {
    if (isFirstPage) {
      if (circular) {
        page = pagesCount - 1;
      }
      return;
    }
    page--;
  }

  function swipeHandler(event) {
    const { direction } = event.detail;
    if (direction === "left") {
      next();
    } else if (direction === "right") {
      previous();
    }
  }
</script>

<div
  class="wrapper"
  use:swipe={{ timeframe: 300, minSwipeDistance: 100, touchAction: "pan-y" }}
  on:swipe={swipeHandler}
>
  <div
    class="floater"
    style="width: {floaterSize}%; transform: translate3d({translateX}%, 0, 0); grid-template-columns: repeat({count}, 1fr;"
  >
    <slot />
  </div>
</div>
{#if arrows}
  <div class="controls arrows">
    {#if circular || !isFirstPage}
      <button on:click={previous}>&larr;</button>
    {/if}
    {#if circular || !isLastPage}
      <button on:click={next}>&rarr;</button>
    {/if}
  </div>
{/if}
{#if bullets}
  <div class="controls bullets">
    {#each new Array(pagesCount) as e, idx}
      <input type="radio" bind:group={page} value={idx} />
    {/each}
  </div>
{/if}

<style>
  *,
  *:before,
  *:after {
    box-sizing: border-box;
  }
  .wrapper {
    overflow: hidden;
  }
  .floater {
    display: grid;
    gap: 10px;
    transition: transform 0.25s ease-in;
  }
  .controls {
    text-align: center;
    margin-top: 10px;
  }
</style>
