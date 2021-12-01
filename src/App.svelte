<script>
  import Carousel from "./Carousel.svelte";
  import images from "./images.js";

  import products from "./products.js";

  let inputItemsPerPage = 1;
  let circular = false;
  let isViewPortDependant = false;
  let startPage = 0;
  let arrows = true;
  let bullets = true;
  let width;

  $: dynamicItemsPerPage = width > 1024 ? 6 : 4;
  $: itemsPerPage = isViewPortDependant
    ? dynamicItemsPerPage
    : inputItemsPerPage;
</script>

<svelte:window bind:innerWidth={width} />

<main>
  <div class="form">
    <label>
      Nb d'items par page:
      <input
        bind:value={inputItemsPerPage}
        type="number"
        min="1"
        max={images.length}
      />
    </label>
    <label>
      <input type="checkbox" bind:checked={isViewPortDependant} />
      Nb d'items en fonction du viewport: ({width}px => {dynamicItemsPerPage})
    </label>
    <label>
      Page:
      <input bind:value={startPage} type="number" min="0" max={images.length} />
    </label>
    <label>
      <input type="checkbox" bind:checked={circular} />
      Défilement circulaire ?
    </label>
    <label>
      <input type="checkbox" bind:checked={arrows} />
      Afficher les flèches
    </label>
    <label>
      <input type="checkbox" bind:checked={bullets} />
      Afficher les boutons
    </label>
  </div>

  <div class="images-carousel">
    <Carousel
      count={images.length}
      {itemsPerPage}
      {circular}
      page={startPage}
      {arrows}
      {bullets}
    >
      {#each images as slide, idx}
        <figure class="slide">
          <span class="idx">{idx}</span>
          <img src={slide.url} width="1280" height="1024" alt={slide.title} />
          <figcaption>{slide.title}</figcaption>
        </figure>
      {/each}
    </Carousel>
  </div>

  <div class="products-carousel">
    <Carousel count={products.length} itemsPerPage={4} {arrows} {bullets}>
      {#each products as slide, idx}
        <div class="product slide">
          <span class="idx">{idx}</span>
          <img
            class="product-image"
            src={slide.url_image}
            width="500"
            height="500"
            alt={slide.label}
          />
          <div class="product-label">{slide.label}</div>
          <div class="product-price">{slide.price_final} €</div>
        </div>
      {/each}
    </Carousel>
  </div>
</main>

<style>
  *,
  *:before,
  *:after {
    box-sizing: border-box;
  }
  main {
    font-family: sans-serif;
  }
  img {
    max-width: 100%;
    height: auto;
  }
  figure {
    padding: 0;
    margin: 0;
    text-align: center;
  }
  label {
    display: block;
  }

  .slide {
    /* padding: 10px; */
    position: relative;
    user-select: none;
  }
  .slide .idx {
    position: absolute;
    color: #fff;
    font-weight: bold;
    font-size: 2rem;
    top: 20px;
    left: 20px;
  }

  .product {
    text-align: center;
    padding: 5px;
    background: #ccc;
    border-radius: 3px;
  }
  .product-image {
    max-width: 100px;
  }
  .product-label,
  .product-price {
    font-weight: bold;
    padding: 0.5em;
  }
  .product-price {
    color: #e00;
    font-size: 1.125rem;
  }
</style>
