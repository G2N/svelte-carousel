<script>
  import { swipe } from "svelte-gestures";
  export let count;
  export let itemsPerPage = 1;
  export let page = 0;
  // wether next and previous actions go back to the end or start of the carousel
  export let circular = false;
  // do we show bullets below indicating which page we're on
  export let bullets = false;
  // do we show arrow for next and previous actions
  export let arrows = false;

  // the floater is the actual div containing the items, it's bigger than the carousel
  $: floaterSize = (count / itemsPerPage) * 100;
  $: pagesCount = Math.ceil(count / itemsPerPage);
  $: translateX = getTranslateX(page);
  $: isFirstPage = page - 1 <= -1;
  $: isLastPage = page + 1 >= pagesCount;

  /**
   * Computes the translation percentage needed to display given page
   * @param page
   */
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

  // Variables used to pan during the user's input
  // the panX is added to the translateX offset via CSS calc
  // ie: transform: translate3d(50% - 100px)
  // this could be done directly in the getTranslateX function
  // but would result in a less readable code
  // the panThreshold is the fraction of the carousel's width
  // needed to change page ie: 2 = half the width, 3 = a third
  //
  // if panX is set (aka the user is panning the view)
  // the transition is turned off via a CSS class

  export let panThreshold = 2;
  let panX = 0;
  let panStart = 0;

  /**
   * Left for reference: simple onSwipeHandler
   * This page change is added to the one in the onSwipeUp handler
   * causing the carousel to move two slides at a time
   * @param event EventDetail
   */
  /*
  function onSwipe(event) {
    const { direction } = event.detail;
    if (direction === "left") {
      next();
    } else if (direction === "right") {
      previous();
    }
  }
  */

  /**
   * When the user start swiping (pressing down)
   * Sets the panStart point for future calculations
   * and resets the panX juist in case this was not done earlier
   * @param event EventDetail
   */
  function onSwipeDown(event) {
    const { x } = event.detail.event;
    panStart = x;
    panX = 0;
  }

  /**
   * When the user is actually moving his thumb
   * sets the panX to the difference between event's x and panStart
   * We should probably check the y axis too so that a diagonal move
   * does not trigger
   * @param event
   */
  function onSwipeMove(event) {
    const { x } = event.detail.event;
    panX = panStart - x;
  }

  /**
   * When the user releases his press
   * if they moved more than the threshold, we change page
   * then we resets panStart and panX
   * @param event
   */
  function onSwipeUp(event) {
    if (Math.abs(panX) >= event.target.clientWidth / panThreshold) {
      panX > 0 ? next() : previous();
    }
    panStart = 0;
    panX = 0;
  }
</script>

<div
  class="wrapper"
  use:swipe={{ timeframe: 300, minSwipeDistance: 100, touchAction: "pan-y" }}
  on:swipedown={onSwipeDown}
  on:swipemove={onSwipeMove}
  on:swipeup={onSwipeUp}
>
  <div
    class="floater"
    class:notransition={panX !== 0}
    style="width: {floaterSize}%; transform: translate3d(calc({translateX}% - {panX}px), 0, 0); grid-template-columns: repeat({count}, 1fr;"
  >
    <slot />
  </div>
</div>
{#if arrows}
  <div class="controls arrows">
    {#if pagesCount > 1 && (circular || !isFirstPage)}
      <button on:click={previous}>&larr;</button>
    {/if}
    {#if pagesCount > 1 && (circular || !isLastPage)}
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
  .floater.notransition {
    transition: none;
  }
  .controls {
    text-align: center;
    margin-top: 10px;
  }
</style>
